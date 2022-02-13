## DevOps skills for Medium, Senior and Architect levels

I've been asked to prepare an outline of DevOps skills for different levels of experience (and salary). To be honest I have no idea how those should be, as DevOps is such a broad area that it's almost impossible. Also DevOps is a little bit different, so the skills described below are biased towards what we do at my company [Future Processing](https://www.future-processing.com/). In this list I've focused on technical skills, because for "soft skills" we've got separate matrix applicable for all positions.

The levels in questions could be defined as follows (just to give you some context):
- **Medium** has some experience and is able to deliver simple, well defined tasks. Usually requires mentoring from more experienced colleagues.
- **Senior** is someone who can work on their own, being able to deliver projects rather just than tasks.
- **Architect** can design and deliver complex projects and also advise, train and mentor colleagues and clients.

As you many notice **JUnior** level is missing, on purpose. I strongly believe there's no *Junior DevOps* but rather *Junior Ops* or *Junior Dev* that gains experience. After some time they can transition to *Medium DevOps* to earn their stripes. Such process is something that I went myself and see in the wild.

Most of the skills listed for particular level do apply for the level above too. So Architect should know everything Senior does. In some cases I've used the same skill to describe expected change of attitude or understanding when evaluating for promotion.

Next to each skill there's a grade stating how important certain skill is:
- **(5)** means critical, must have on this level.
- **(3)** means important, but can live without if other skills are strong.
- **(1)** means optional, nice to have but not required.

## Development

Dictionary for the terms used:
- Programming language is any general purpose language like Python, Java or Go.
- Software development life cycle (SDLC) is all the steps needed to deliver a software product including: design, development, build, test, deploy.
- Git actually means any (distributed) version control system, but it's shorter just to write git ;)

### Medium
- (3) Knows at least basics of one programming language.
- (3) Knows what SDLC is and can explain steps.
- (1) Is aware of Agile and Waterfall approach to SDLC.
- (3) Can use the basic git commands (commit, pull, push).

### Senior
- (3) Is fluent in at least one programming language.
- (1) Can point out problems in existing software development life cycle.
- (1) Is able to judge if project is Agile or Waterfall and explain the consequences.
- (3) Knows various branching models in git and how to use them.

### Architect
- (5) Took active part in writing a complex, enterprise-grade system.
- (5) Can read and provide hotfix in more than one scripting languages.
- (3) Can propose and design optimizations in software development life cycle.
- (1) Can design SDLC in the Agile or Waterfall approach.
- (5) Knows git-flow is a lie and trunk-based development is the way to achieve proper CI setup.


## CI/CD

Dictionary:
- CI/CD stands for Continuous Integration / Continuous Delivery (or Deployment) - an approach to test, build and deliver software.
- CI system (Continuous Integration) is a software (or service) that runs test, build and other automation around SDLC. Examples are Jenkins, GitLab, GitHub Actions, AWS CodeBuild.
- Package manager is a system to manage dependencies (i.e. libraries, modules) during software development, used to build a software artifact. Examples are Maven or Gradle in Java/JVM, NPM or Yarn for JavaScript or TypeScript.

### Medium
- (5) Knows the CI/CD terms.
- (5) Can setup a simple CI pipeline (few steps) based on existing setup.
- (3) Can use at least one package manager.

### Senior
- (5) Can tell the difference between Continuous Integration and Delivery.
- (5) Can design and deliver CI pipeline (with many steps) based on requirements.
- (3) Can use many package managers and is fluent in at least one.

### Architect
- (5) Can differentiate between Continuous Delivery and Deployment and advice which one is better.
- (3) Can propose and build CI setup (many pipelines) based on developer team needs.
- (3) Can point out common pitfalls in package manager usage and optimize it.


## Observability

- Log aggregation is process of gathering logs from many nodes (typically in a cluster) into a single place for processing. Examples are ELK stack (Elasticseach, Logstash, Kibana), Grafana Loki, Splunk, AWS CloudWatch.
- Metrics visualization is a way to use graphs to display application or system metrics over time. Example tools are Grafana, Kibana.
- Alert is a notification of some issue (incident) in the system.
- Runbook is a tutorial that describes how to react on an alert to troubleshoot or mitigate an incident.
- Trace is a detailed information of what caused software issue (i.e. exception stack trace). Example of distributed tracing software is OpenTelemetry, Jaeger, Grafana Tempo.

### Medium
- (5) Knows the benefits of log aggregation and can it use it.
- (5) Can use metrics visualization to troubleshoot some problems.
- (3) Knows how to react to alert using existing runbooks and can update / create runbooks.
- (1) Knows what is trace and why it is useful.

### Senior
- (5) Can build the log aggregation system using self-hosted solution based on provided design.
- (5) Can build metrics visualization in some popular tool.
- (3) Can tell the difference between log even and metric and is able to advice dev team on that.
- (3) Can advice on actions that will prevent alert from firing on known issues.
- (1) Uses traces to discover problems with the software to help dev teams fix them.

### Architect
- (5) Can provide many designs of log aggregation and explain their pros and cons.
- (5) Can design and advice on system or application metrics, where tracking those will prevent some issues.
- (3) Can advice dev team on alerts that will help them react before issue happens.
- (1) Can setup trace aggregation system.


## Containers

Definitions:
- Container is a way to run application in a cloud-native way. Example container runtimes are Docker and containerd.
- Orchestrator is a system that manages containers in a cluster. Examples are Kubernetes, ECS (Elastic Container Service).
- VM stands for Virtual Machine, a way to isolate apps before containers came along.

### Medium
- (5) Understands containers and how do they work. Can run an app in container.
- (3) Knows there is an orchestrator and what it does.
- (3) Understands the difference between Docker container and image.
- (1) Understands te difference between container and VM.

### Senior
- (5) Knows one container runtime in-depth (i.e. attaching volumes, config options, health-check).
- (5) Can setup and manage orchestrator in production.
- (5) Understands the difference between Docker container and image.
- (1) Understands te difference between container and VM.

### Architect
- (5) Can advice dev team how containers should be build and run (i.e. 12factor app).
- (3) Can design a platform based on orchestrator that is the best suited based on requirements.
- (3) Understands that container is just a Linux process with some degree of isolation.


## Operations

Dictionary:
- Shell script is a Bash or PowerShell script. Used to automate actions.
- System signals are a way to manage processes by the kernel. Examples are SIGTERM, SIGKILL, SIGHUP.

### Medium
- (5) Can write a simple shell script.
- (5) Can check state of processes on the system.
- (3) Knows system signals and how to use them.

### Senior
- (5) Can write significantly complex scripts (i.e. deployment pipeline), using conditionals and loops.
- (3) Is able to debug process (i.e. logs, strace).
- (3) Can advise dev team how to handle system signals in the app.

### Architect
- (3) Knows when shell script is not enough and proper programming language should be used.
- (3) Can advise if distributed tracing solution would help.
- (3) Is able to propose a fix in code based on signals not being handled.

## Networks and cloud

- DNS stands for Domain Name System ahd how the name resolution works.
- RBAC stands for Role-Based Access Control. Examples are Kubernetes RBAC or IAM Roles.
- Proxy is software that passes network traffic. HTTP reverse proxy is a layer 7 proxy, it can act as a load balancer too. Layer 7 refers to ISO/OSI network model.
- IaaS, PaaS and SaaS are Infrastructure / Platform / Software as a Service.

### Medium
- (5) Knows how DNS works, for the basic name resolution.
- (5) Understands the concept of RBAC and can explain rules evaluation.
- (3) Knows the concept of proxy (i.e. HTTP reverse proxy) and load balancer.
- (3) Knows the difference between IaaS, PaaS and SaaS.

### Senior
- (5) Knows all the bits that are involved in DNS name resolution (i.e. in Linux).
- (5) Can setup RBAC rules according to specification and good practices (i.e. the least possible permissions).
- (5) Can setup proxy server using some popular tools (i.e. Nginx).
- (3) Knows the difference between layer 4 and layer 7 load balancer (or proxy).
- (3) Knows varius classes of cloud offerings (i.e. object storage, load balancer) and how to use them.

### Architect
- (3) Understands the performance implications of DNS in a large cluster.
- (5) Able do design RBAC setup (roles, policies, groups) for the whole system (i.e. cluster).
- (3) Can design a cluster using both layer 4 and layer 7 proxies or load balancers where appropriate.
- (1) Can leverage PaaS / SaaS offerings (i.e. load balancers) to achieve business goals (i.e. time-to-market, cost reduction).


## Infrastructure as Code

Dictionary:
- Infrastructure as Code (IaC) is a concept to keep setup and configuration of infrastructure as code in git, so it can be developed just as applications are.
- Provisioning is a way to reach desired state of software and configuration on the sever. Example tools are Ansible, Puppet, cloud-init. Those tools are not used in the era of container orchestration, but the concept is still relevant.
- Configuration management is about providing config for app (in many ways), but also a system to deliver and distribute the configuration.
- 12factor is a set of good practices for modern apps https://12factor.net/

### Medium
- (5) Knows IaC concept and its benefits.
- (5) Understands the server should be managed (spin up, provision) automatically, by some tool.
- (3) Can use various ways to configure app (i.e. env vars, files).

### Senior
- (5) Can prevent common pitfalls of IaC setup using tools and processes (i.e. git branches, code review).
- (5) Is able to manage server (spin up, provision) automatically, by some tool.
- (5) Understands configuration is separate from app deliverable (12factor app).
- (3) Can setup configuration delivery using existing tool (i.e. orchestrator or some open-source or SaaS).

### Architect
- (5) Can advise on GitOps drawbacks and put guardrails into place.
- (3) Understands the provisioning tools do not fit the cloud-native era. Can advise on how to migrate out of them.
- (3) Can design configuration management system for the cluster.
- (1) Can coach dev teams to adhere to config best practices (12factor app).


## Others

- DORA metrics (DevOps Research & Assessment) are 4 basic metrics of software delivery performance, proven to work using scientific approach.

### Medium
- (1) Understands DevOps role in SDLC.

### Senior
- (3) Understands how DevOps practices affect SDLC.

### Architect
- (1) Can use metrics (i.e. DORA) to lead and track SDLC optimization.


## Tell me what do you think

All of the skills and the grades are highly subjective, I realise that. Also as I mentioned they reflect the environment my company is operating, so it (DevOps, skills, grades) might be completely different for you. Even though I'd like to know your opinion and I'm open to feedback what I'm missing, what is wrong or unclear. Feel free to leave a comment or hit me on [Twitter](https://twitter.com/AdamBrodziak) :)
