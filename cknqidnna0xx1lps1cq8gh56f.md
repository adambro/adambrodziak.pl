## Ad-hoc documentation

The promise: With just little bit more effort you can create an ad-hoc documentation that is searchable and useful.

Writing documentation is tedious process, that's why in Agile we don't write documentation, right? Wrong! The truth is as software developers we're typing quite a lot of docs already, be it code comments, git messages, pull request comments, chats, mails, Jira comments, etc. By doing those in a more thoughtful way we can make it useful for our future selves and for our colleagues.

## Work on the message

Essentially it comes down to switching to [low-context (vs high-context)](https://en.wikipedia.org/wiki/High-context_and_low-context_cultures) communication style. Practical tips below.

### Provide more details

When you're in the middle of doing something the context is fresh in your mind, so it makes sense to use shortcuts like:
> Restarted and it's working.

That's completely valid message. But what it is about, do you know? Will you know in 6 months from now? How about this example:
> Deleted the agent pod, so it got restarted and it's responding to HTTP requests again.

Now that makes sense even to you, dear reader, because it includes some context:
- What exactly has been done: Deleted the pod.
- Specific subject is mentioned: _agent_ instead of _it_ pronoun.
- The effect is described: Restarted, responding.
- How it was verified: _Responding to HTTP_ instead of _working_.

Little effort, great effect.

### Summarize often

Chats are usually in-the-moment, high-context conversations like:

``` 
Al: kubectl not working
Bob: is it in /usr/bin?
Al: yeah
Bob: what is the error?
Al: `permission denied: kubectl`
Bob: is it executable?
Al: dunno
Bob: try `chmod +x /usr/bin/kubectl`
Al: another error
Bob: use the `sudo` Luke!
Al: worked <3
```

This conversation has the information to fix the problem, but is will you be able to find it in 6 months? Even if you find it how much time you're going to spend on tring to read that all and get the gist? Maybe write a quick summary:
> After downloading `kubectl` put it in `/usr/bin` dir and make it executable using `chmod +x /usr/bin/kubectl`. Both require root privileges (use `sudo`). Otherwise you'd see `permission denied` error.

Some effort and you're becoming the owner of the solution.

### Rephrase what other people said

In addition to summary you can also re-phrase some of the points. It servers two purposes:
- You make sure both sides understand the same.
- You add additional keywords to match future search terms.

This effort will show you as a good communicator.

## Make it searchable

Information is useful only if it can be found. This is what made Google a giant. Similar story with Slack, which is an acronym from Searchable Log of All Communication and Knowledge.

### Funnel comments into Slack

Slack has pretty good search capabilities and a lot of integrations with services. We've got Jira comment being syndicated to Slack automatically. The same can be done with other source of docs like: code review comments, git messages and other temporal texts.

I guess the notes, mails and documents can be copy&pasted to Slack as well. There might be good to have a reference for original document, but having a snapshot copy in Slack might be useful too.

### Code comments and git messages

GitHub has got pretty good search across many projects. GitLab can do search only within single project, unfortunately. BitBucket has limited cross-project search capabilities. Any of that can be syndicated to Slack.

[Sourcegraph](https://about.sourcegraph.com/) has semantic code search, because it actually understands the code being indexed. It has some powers of IDE for search. The limitation is it can't search code and git at the same time.

### Use task numbers and tags in the message

Quite obvious, but sometimes forgotten, is to put task number (i.e. Jira identifier) in the git commit, code comment, Slack message, etc. The same way regular tags would work, especially if we have some shared vocabulary of such tags, i.e. #versionBump indicating it's just a version increment.

## Summary

With little additional effort you can start building institutional knowledge. Such documentation is contextual and temporal, so it made sense at that time and given circumstances. It wont replace permanent documentation (i.e, specifications, reference manuals), but is light and agile addition that is almost free.

Every time when you write a comment stop for a second and think about your future self while writing. You'll thank me later.