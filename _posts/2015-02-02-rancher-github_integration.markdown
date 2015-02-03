---
layout: post
title:  "Rancher now supports GitHub OAuth"
date:   2015-02-02 15:37:06
categories: rancher-stuff
---
Rancher is an exciting new product, developed by [@Rancher_Labs](https://twitter.com/Rancher_labs), that allows dockerized applications to run on any cloud service provider. As a part of its first major release, Rancher will have support for GitHub OAuth. In this blogpost I'll show you how to setup GitHub OAuth on Rancher for your organization.

* Rancher-Auth 2-minute setup.
* How do we do authentication?
* What's planned for the future?

Rancher Auth 2-minute Setup
----------------------------

Here's a short video explaining the setup of Github OAuth on Rancher.

<iframe width="420" height="315" src="https://www.youtube.com/embed/7vbq83UNxUk" frameborder="0" allowfullscreen="allowfullscreen"> </iframe>

How do we do authentication?
-----------------------------
Github is free and easy to use. A wide spectrum of organizations, from large corporations to small startups display their open source might using GitHub. In order to make it easy for our clients to use our product, we built our authentication feature based on GitHub OAuth. GitHub OAuth provides capabilities like :-

1. GitHub organizational structure reflects the access control structure that organizations wish for.
   
   * GitHub organizations consist of teams, and teams consist of repositories. Rancher allows one to create access controls based on these structures. 
   
       + For example, If you wanted the resources of one of your projects to be controlled by a limited set of people (say the members of a single team within your organization), it is easy to setup a rancher project just for that team. The team members would then be able to add/delete/edit the resources that belong to them. 
  
   * Additionally, GitHub allows one to configure auth based on users and organizations. Rancher leverages the flexibility of these structures as well. 

       + For example, If you wanted the resources to be constrained to just one user, you could create a Rancher project and set the scope to user. 
       
       + Similarly, you could set the scope to "organization" level and all the members of your organization would be able to access the resources of the project. 

2. The setup, maintanance and usage of GitHub auth is simple. 

   * Since Rancher doesn't maintain passwords or complex mappings, the implementation is safe, secure, simple and robust.

What's planned for the future?
------------------------------

GitHub OAuth doesn't provide fine grained access controls such as providing read only access to a subset of people in the organization or write access to another subset of people in the organization. Such complex access control can be provided with LDAP. LDAP can be expected in the near future versions of Rancher.
