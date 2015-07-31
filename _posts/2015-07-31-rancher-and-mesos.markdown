---
layout: post
title:  "Rancher and Apache Mesos"
date:   2015-07-31 13:00:06
categories: rancher-mesos
---

Ever since Benjamin Hindman created Mesos as a part of his PhD thesis work, it has risen in popularity as the *de-facto* framework for running distributed applications at scale. Twitter, Apple and other web giants use this framework to run their applications at scale. Apache mesos can be used to run applications on 10s of 1000s of nodes, but in order to run your applications on Mesos, you need a Mesos framework. The effort required for writing one could range from a few weeks to months. As an enterprise or a startup, this may not be ideal.

Mesos helps you scale your data centres really really well, but not your applications. Mesos ensures that jobs are scheduled while optimizing resource utilization. However, Mesos is not designed to help your application scale. In order to scale your applications, you need load balancing, service discovery, rolling upgrades, application composability, continuous deployment and others. Upgrading apps at scale requires complex orchestration not provided by Mesos. 

These are not flaws in Mesos, but it is by design. These needs are meant to be filled by Mesos frameworks - Marathon, Chronos, others and now Rancher. Rancher aims to fulfill this need for an application centric, docker oriented framework that will work in conjunction with Mesos, and provide you with the best of the three worlds. 

1. Resource optimization, fault tolerance of Mesos
2. Utility, composability, usability of Rancher
3. Isolation, containerization of Docker. 

   *With Mesos, you can scale your data centre. With Rancher, you can scale your applications.*

### The Rancher Mesos Architecture

Here's a diagram explaining the Rancher Mesos Framework

![](http://i.imgur.com/CR5QOWA.png)

As you can see, Rancher integrates with Mesos using standard Mesos interfaces - A Mesos framework. The various components are 

1. Mesos Master
    
    The Mesos Master is a cluster of machines that run the `mesos-master` process. It maintains, and monitors the Mesos slaves, and handles resource offering, task launching, task monitoring, fault tolerance and message passing etc.

2. Mesos Slave
    
    These are the hosts on which jobs are to be scheduled. In this case, these will be used to launch VMs that register with Rancher. We use VMs instead of containers because we provision hosts using Mesos, unlike other frameworks that schedule jobs on it. These VMs can then be orchestrated using Rancher, and jobs can be scheduled on them using containers.  
    
3. Rancher Server
    
    This is a cluster of machines that run the `rancher/server` docker container. It maintains, and monitors the Rancher host, while providing services to them such as private networking, service disovery, rolling upgrades, load balancing and rancher-compose.

4. Rancher Hosts
    
    These are hosts provisioned using Mesos' resource offers. These hosts run docker and have the `rancher/agent` container running, which is used for Rancher's private networking and for various tasks involving the hosts.

5. Rancher-Mesos Scheduler [(github)](https://github.com/wlan0/rancher-mesos-scheduler)
    
    The scheduler is a two tiered application. It is a Rancher external event handler, as well as a Mesos scheduler. The event handler is used to listen on `create host` event from Rancher. The scheduler is used to listen for `resource offers` from Mesos. When the Rancher-Mesos scheduler receives a `create host` event, it adds that event to an event queue. Once Mesos provides a suitable slave to schedule tasks on, it dequeues events , and the Rancher Host is created on that Mesos slave, if it has sufficient capacity.

6. Rancher-Mesos Executor [(github)](https://github.com/wlan0/rancher-mesos-executor)
    
    This is the process that is invoked when an available slave is provided to rancher for creating hosts. This process uses QEMU-KVM to create VMs with bridge networking. Docker is installed on these VMs and then `rancher/agent` is started to make it register with Rancher Server.

7. Rancher-Mesos Framework
    
    The Rancher-Mesos Framework is used to refer to Rancher-Mesos Scheduler and Rancher-Mesos Executor collectively.

### The Rancher Mesos Workflow

The user's point of view of working with the Rancher Mesos framework would be no different from using Rancher today. The user would click on `Add Host` in the UI. Which would provision a Rancher Host in one of the available Mesos slaves. The slave on which it is provisioned is determined by the Mesos master. The Rancher host, once provisioned will register with the rancher server. It will show up in the UI and the user can view stats, execute shell or start/stop containers like normal.
    
To leverage Rancher's feature richness, use Rancher UI or `rancher-compose` to create services, load balance services, define stacks, and environments or define relationships between applications. The simplest way to start a docker container in one the Rancher Hosts is from the UI. By clicking on the large '+' in the host UI, a container can be started.

### Looking under the hood

In order to understand how the framework works, let us take an example of adding a host and starting the nginx container on it.

 - When you click on `Add Host` in the UI, the scheduler receives a `physicalhost.create` event. 
 - The scheduler saves this event in an event queue, and waits for the mesos master to offer resources. 
 - Once an offer is received, the scheduler retrieves the earliest event from the event queue, and launches a task on the offered slave. 
 - The task launches the executor on the slave machine.

 - On the slave side, the executor is launched by the slave whenever a task is launched. 
 - The executor first downloads the necessary OS images for the new VM and then boots up the OS. 
 - The executor uses QEMU-KVM to start the VM. Here, the networking for the Slave and the VMs are setup in such a way that all the slaves and the newly provisioned VMs will be on the same subnet. This is achieved by using bridged networking while provision VMs. The benefit of doing this is that all the VMs and the Slaves are reachable from Rancher Server.  
 - The newly created VMs are then made to download docker, then start `rancher/agent`, effectively announcing themselves to the Rancher Server.

### Next Steps

1. If you have any questions, please post them on our [forums](https://groups.google.com/forum/#!forum/rancherio)
2. If you like to reach out to me, email me at sid@rancher.com
