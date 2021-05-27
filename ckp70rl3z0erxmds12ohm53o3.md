## Terraform is terrible

Here is my experience from running and upgrading a small Terraform project. As you might have guessed it was not great, but I'll try to focus on facts rather than opinions (even though some might sneak in). It will be mainly about the CLI client and it's versioning schema, but also some complaints about state management. I'm big proponent of CI/CD and Infrastructure as Code and I will try to explain how Terraform does not fit the picture.

The project is small, but manages 8 clusters. Contrary to typical case it's a SaaS: Atlas service that offers managed MongoDB on AWS in our case. Every project is using some of 7 modules that represent Atlas resources with necessary AWS bindings (i.e. secrets). When we started 0.12 was the newest version, so upgrade to 0.13 is part of the story.

Since the issues are about Terraform client mostly, the IaaS or SaaS used is not that relevant. However Atlas plugin had to change how internal configuration structure, which only added insult to injury.

## State

Let's start with state, which is common pain point while working with Terraform. To some extend I understand the decision of using state, but it is inherently difficult to manage.

I dare to say that remote state has all the disadvantages of cache, but not many advantages. Sure, for a team working on a project remote state is a must. Actually I'd like a solution that would enforce using remote state, but there's none - we have to rely on state config will be copied over from existing project.

### State config on AWS using S3 and DynamoDB

State configuration is another weak point. Typically S3 backend is used to store state, that's fine. But if you want to make it safe from many people overwriting each other changes by running `terraform apply` at the same time you need additional configuration for locks or mutex. You should definitely use that!

%[https://twitter.com/AdamBrodziak/status/1387764929286004745]

In AWS realm DynamoDB is needed for locks. My guess is S3 does not have atomic operation to obtain a lock, so key-value database is needed. What I don't understand is: why I need both? __Why not store state in DynamoDB directly?__ The state contents is just JSON, right? If you happen to work on Terraform let me know, please. For me that's a big overlook.

Anyway, below is sample state config using S3 and DynamoDB - both are needed to make it safe. Feel free to copy :)

```hcl
terraform {
  backend "s3" {
    bucket          = "terraform-state-prod"
    key             = "mongo-atlas/resources/DEVELOPMENT-resources.state"
    region          = "eu-west-1"
    dynamodb_table  = "terraform-state-prod"
  }
}
```

Remember to use remote state (with locks!) if you're working on a team or using Terraform on CI/CD pipeline. This is lesser of two evils (everyone having its own state!).

### Discrepancies of state contents

Using locks avoid one way state can be corrupted: running more than one `terraform apply` at the same time. Other issues arise from HCL code, state and system discrepancies. Let's look at those.

First issue is the difference between Terraform state and the actual state of the system. Sometimes it's called _configuration drift_. One case is when someone adds something in the admin console, then it's invisible for Terraform. We had that for IP access list, but since all the entries are independent there was no conflict. The only drawback was that we no longer have single source of truth, so better avoid such practice and either manage things manually or via Terraform code.

Bigger problem is when someone _changes_ something on the server manually, but the change is not reflected in Terraform. In such case it will be visible during `terraform plan` as change. You have to make a decision if to override it via `terraform apply` or reconcile to HCL code. In case of the latter there will be playing a detective investigation on who and why made a config change. My recommendation: avoid those cases at all cost and manage everything via Terraform!

### Default values are stored in state

Another interesting case is when state has items that are not in the Terraform code. I guess that happens, because Terraform stores in state the _evaluated_ state from the system (for us: Atlas service), with the created resource IDs and values for default parameters. We were quite surprised that plan says `bi_connector` will be removed. What puzzled me even more: `bi_connector` was not in the code!

What happened was: `bi_connector` is optional, so default values were stored in state. Since Atlas plugin was upgraded [they've removed `bi_connector` attribute](https://github.com/mongodb/terraform-provider-mongodbatlas/pull/423) and replaced it with `bi_connector_config` to adhere to new HCL parser. That's an example how syntax breaking changes affect you in a surprising way, but more on that later.

### Other cases of state corruption

Terraform stores a client version in the state too. It has got interesting implications: if you change the state with newer client (running `terraform apply`) it will enforce others (i.e. CI worker) to use the same version too. We've mitigated that by wrapping Terraform CLI in script that manages the version, so everyone will use exactly the same.

On top of all that state can get into messed up state when `terraform apply` process terminates (i.e. by hitting `Ctrl+C`). To be honest I haven't experienced that myself, just heard that on Terraform training. However I can guess why: there's no transactional support in applying changes by Terraform. I understand why, distributed transactions are hard, but still I don't like it.

## Client

My biggest surprise was how bad the Terraform client is. Don't get me wrong: I love that it's a standalone Go binary, so it was easy to manage specific version. The Makefile script downloads Terraform binary, then runs `terraform init` and `terraform plan` for specific cluster. On the other hand I don't want to create such wrappers, just use whatever version is compatible. That is not possible due to versioning policy and instability of the tool, but more on that later.

### Managing many workspaces

As I mentioned we have 8 clusters, each having resources configuration in its own directory. I needed to use the downloaded binary and run it in the cluster dir, easy. After looking at help there's a `terraform apply [options] DIR` param, so tried to use that. Unfortunately I've got an error about missing param values, like it would not read the `.tfvars` files from the same directory.
Apparently the `DIR` at the end does not work as you'd expect, instead Hashicorp [added `-chdir` global param](https://github.com/hashicorp/terraform/commit/efe78b2910c5b1f2292f0a16d990b2ad92352feb) to handle such use case. Param `-chdir` landed in 0.14 version, but I was upgrading to 0.13, so clever workaround of `cd cluster/DEVELOPMENT && ../../terraform apply` was necessary.

Since Makefile is excellent at managing dependency graph I thought: lets run `terraform init` only when necessary. That was quite easy, unless init fails for some reason and leaves local workspace partially initialized. Then only way to manage that in Makefile was to remove the partially initialized workspace and start over. That is how the overly familiar Makefile `cleanup` target was born, which nukes `.terraform` dir for every workspace and removes the binary too. I wish I better understood how to local Terraform workspace is created, because I sense there's a room for improvement :)

### Running apply in CI/CD pipeline

Terrafrom client has been made for interactive use from the start. That's why it will force you to type `yes` in to confirm `terraform apply` changes and provide a prompt for missing parameter values (instead of just exiting with an error). To overcome that you have to resort to CLI params like `-auto-approve` or `-var 'foo=bar'` for example. Those are necessary for any CI/CD pipeline, but such UX design looks like an afterthought. The UX with `-auto-approve` on CI is crumbled in another way: `terraform apply -auto-approve` does not present changes that are going to be made. Why!? How to verify what kind of changes were made by looking at pipeline console then?

The solution is to run `terraform plan` just before apply, so we'll have a record of changes to the infra. There's another problem with that: there might be changes in the infra system state between `plan` and `apply` actions. Let's imaging a pull request workflow, where `plan` is run for a branch to verify if code diff has the desired effect on the infra and `apply` is run after merge to main branch. There could be hours, even days, between change plan and the actual application of them.

One solution to that problem would be to run `plan` command just before `apply` again on the main branch (after merge). Ideally there should be an option to prevent `apply` in case of change plan taking an undesired direction.
Second solution could be using `plan -out` parameter, which saves change plan to file, so `apply` can pick up the exact change plan that was generated before. To be honest I haven't tried that yet, but I keep wondering why `-out` is not the default setting and why apply does not require change plan as input. Such design decisions keep me puzzled.

## Versioning

Out project has started around August 2020, so Terraform 0.12 was the most recent version. I was happy about that, because 0.12 introduced syntax changes to how params and should be quoted. In reality many of the existing solutions used the old syntax, which still worked in 0.12 version. As a result our project was a mix of new and old syntax, which was not a problem until the upgrade to 0.13 started throwing deprecation warnings. Of course I've learned about `terraform 0.12upgrade` command soon, but it kept throwing syntax errors on projects that had mix of old and new syntax. Our option was either to downgrade everything to 0.11 (which sounds silly) or upgrade syntax manually. I went with the latter, which was quite a lot of getting back and forth, because deprecation warnings do not show you all the occurrences of problem, but just the first one and "there are 55 more" message. Not useful.

Of course `terraform 0.12upgrade` is only available in 0.12 version, so even if I wanted 0.13 first had to download older one to try to upgrade code. That was the reason that led me to creating Makefile wrapper for Terraform binary.
Why bother with 0.13 upgrade after all? Well, we needed [`for_each` syntax feature](https://www.terraform.io/docs/language/meta-arguments/for_each.html) which was only in 0.13 version for modules. A side note: is it only me or adding new syntax for some cases in `0.12.6` version feels wired? The `for_each` feature was required to manage many Atlas user roles in a dynamic attribute.

The biggest problem is that even patch versions (x.y.Z) can introduce new features, so it might not be enough to have any 0.13.x version, but rather bind to the specific `0.13.7` for all clients. In addition client version is stored in state, so using newer by accident can enforce an upgrade for everyone.
Managing Terraform versions is really cumbersome, to the extend that tools like [terraform-switcher](https://github.com/warrensbox/terraform-switcher) exist.  I did not wanted yet another interactive CLI tool for our CI server, that's why I've went with Makefile that can be used on CI and locally. In my opinion Makefile is vastly misunderstood (and hence underused tool), but that's a story for another time.

Since the start of the project (August 2021) Hashicorp released 3 new major versions. The newest one is `0.15.4` and we've started on `0.12.5` back then. It means that over the last 6 months 3 breaking change releases have been made, one every second month! That is rapid change rate for something that should be stable and boring like infrastructure. The other surprising fact is that initial Terraform release was in 2014, so the project is almost 7 years old.

## Parting thoughts

So far it was mostly about facts and my experiences around using and upgrading Terraform. Now time for a little opinion and thoughts about the project. My experience with managing Terraform is just last few months, previously I've mostly used Terraform setup by someone else.

Based on the rate of breaking changes in the last 6 months I'm worried about the stability of the product. In my opinion it should have a big BETA badge to warn about that, even despite being 7 years in development. Sure the `0.x.y` versioning scheme might indicate that it's not ready for prime time and API breaking changes will happen. I understand that, even SemVer allows breaking changes for minor version bump (0.x.0) if it's not `1.0.0` yet. For me that looks like a lazy policy on the Hashicorp side that after years of development they still have a policy open to breaking changes. I thought even Facebook dropped "move fast and break things" attitude by now...

Even though development seems to be dynamic I see that legacy seems to creep in already. Just look at the `DIR` command line parameter that is going to be replaced by `-chdir` which is clearly stated in the commit message. Even for the Terraform documentation it is going to be quite a lot of work. What about all the solutions and workarounds existing the the wild, i.e. on StackOverlow or blog posts? This is going to have similar impact as using old syntax (0.11 and older) in new project, because someone found a solution somewhere. Without clearly communicated versioning policy it will never get in order.

The biggest surprise to me is that Terraform has ADOPT status on [TechRadar April 2019](https://www.thoughtworks.com/radar/tools/terraform) from ThoughtWorks. Maybe the timing plays a role here, since it was way before the major change in syntax  in 0.12 version shipped mid-2020? I wonder if Terraform is still state of the art, or there are better Infrastructure as Code solutions recommended by ThoughtWorks or others?