---
layout: post
title:  "Rancher adds support for Projects"
date:   2015-02-16 13:00:06
categories: rancher-stuff
---

This is a follow up post to my earlier post on [GitHub OAuth on Rancher](http://sidharthamani.github.io/rancher-stuff/2015/02/02/rancher-github_integration.html). If you haven't read that, I would recommend you to read it as it gives you an introduction to Rancher Auth using GitHub. 

Projects is an extension of GitHub OAuth on Rancher. GitHub provides abstractions such as users, teams and organizations which naturally reflect a real life organizational structure. We leverage these structures at Rancher to support Rancher Projects.

## Projects demo

This demo will show you how to create projects on rancher for various levels of access control. 

<iframe width="420" height="315" src="https://www.youtube.com/embed/e9YbYxoXIUg" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

## The project use case

The main use case for this feature is that companies can now provide selective access to their resources. For example, if a company has a development team and a ops team, and let us say the ops team manages the production resources, and the development team uses the development resources. Production resources are critical and are not to be tampered with. The development resources are meant to act as a production emulating environment. The developers can push alpha, and beta releases into this and test their code on production like systems. This environment should be accessible by developers who are testing their code, and should not be given to anyone else, as their code is being tested on it. There is clear case here to separate the access to these two types of resources. 

Now, with the latest release of rancher, which supports projects, one can address the use case described above. It can be done in three ways

1. Team projects
2. Org projects
3. User projects 

### Team projects 

Team projects allows one to allocate resources and provide access to a team of people. In the above described usecase team projects would work really well. You could create one team prokect for the dev team and one for the prod team. Then both teams would be able to access their own resources.

### Org projects

Organization level projects allocate resources and provides access to all memebers of the organization. For example, if you wanted to create a resource called demo, that everyone in your organization could orchestrate, this type of project would be the ideal choice.

### User projects

User projects allow resources to be orchestrated by a single user. It is meant to be used when a single user is to be the sole orchestrator of the resources. One of the caveats of this type of proejct is that, users can create "user-level" projects only for themselves. 

