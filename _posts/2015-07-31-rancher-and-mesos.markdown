---
layout: post
title:  "Rancher and Apache Mesos"
date:   2015-07-31 13:00:06
categories: rancher-mesos
---

Mesos is a resource manager, and a scheduler, but is often not enough to scale your applications. In order to scale your applications using Mesos, you need a Mesos framework. For eg. Mesosphere, Chronos, and others. Frameworks provide capabilities like load balancing, service discovery, rolling upgrades, application composability, continuous deployment and others. 

A number of Rancher's community members who were also using Apache Mesos have felt the need for a system that combines the fault tolerance, and scheduling capabilities of Mesos with the container management service provided by Rancher. One community user, [Marcel Neuhausler](https://github.com/neuhausler) from AT&T, took the initiative to chart a broad design and envisioned a workflow for such an integration. It was his idea to combine the scheduling capability of Mesos to schedule VMs and then Manage those VMs using Rancher.  He even wrote a Mesos framework, which proved to be a good starting point for writing this framework. In this blog, I am going to describe the ideas and the software - Rancher Mesos framework, that resulted through the act of collaboration with Marcel. This framework can be used to setup large scale production jobs like Hadoop, Kafka, ElasticSearch etc. in docker containers.

In the sections below, I'll discuss the architecture of the framework, and show you how to set it up on your local environment. 

### The Rancher Mesos Architecture

Here's a diagram explaining the Rancher Mesos Framework

![](http://i.imgur.com/CR5QOWA.png)

As you can see, Rancher integrates with Mesos using standard Mesos interfaces - A Mesos framework. The various components are 

1. Mesos Master
    
    The Mesos Master is a cluster of machines that run the `mesos-master` process. It maintains, and monitors the Mesos slaves, and handles resource offering, task launching, task monitoring, fault tolerance and message passing etc.

2. Mesos Slave
    
    These are the hosts on which jobs are to be scheduled. In this case, these will be used to launch VMs that register with Rancher. We use VMs instead of containers because we provision hosts using Mesos, unlike other frameworks that schedule jobs on it. These VMs can then be orchestrated using Rancher, and jobs can be scheduled on them using containers.  
    
3. Rancher Server
    
    This is a cluster of machines that run the `rancher/server` docker container. It maintains, and monitors the Rancher host, while providing services to them such as private networking, service discovery, rolling upgrades, load balancing and rancher-compose.

4. Rancher Hosts
    
    These are hosts provisioned using Mesos' resource offers. These hosts run docker and have the `rancher/agent` container running, which is used for Rancher's private networking and for various tasks involving the hosts.

5. Rancher-Mesos Scheduler [(github)](https://github.com/wlan0/rancher-mesos-scheduler)
    
    The scheduler is a two tiered application. It is a Rancher external event handler, as well as a Mesos scheduler. The event handler is used to listen on `create host` event from Rancher. The scheduler is used to listen for `resource offers` from Mesos. When the Rancher-Mesos scheduler receives a `create host` event, it adds that event to an event queue. Once Mesos provides a suitable slave to schedule tasks on, it de-queues events , and the Rancher Host is created on that Mesos slave, if it has sufficient capacity.

6. Rancher-Mesos Executor [(github)](https://github.com/wlan0/rancher-mesos-executor)
    
    This is the process that is invoked when an available slave is provided to rancher for creating hosts. This process uses QEMU-KVM to create VMs with bridge networking. Docker is installed on these VMs and then `rancher/agent` is started to make it register with Rancher Server.

7. Rancher-Mesos Framework
    
    The Rancher-Mesos Framework is used to refer to Rancher-Mesos Scheduler and Rancher-Mesos Executor collectively.

### The Rancher Mesos Workflow

The user's point of view of working with the Rancher Mesos framework would be no different from using Rancher today. 

- The user would click on `Add Host` in the UI, which would provision a host in one of the available Mesos slaves. The slave on which it is provisioned is determined by the Mesos master. 
- The host, once provisioned will register itself with the rancher server. It will show up in the UI and the user can view stats, execute shell or start/stop containers like normal.
    
### Looking under the hood

This picture explains the sequence of events to provision a host using Rancher Mesos

![](http://i.imgur.com/N3NoCR3.png)


1. When you click on `Add Host` in the UI, Rancher server creates a `physicalhost.create` event.
2. This event is received by all the external handlers that have subscribed to this event. In this case, the Rancher-Mesos scheduler subscribes to this event.
3. On Receiving the event, the scheduler saves the event in an event queue.
4. Then the scheduler waits for a resource offer of a free host from Mesos Master. 
5. Once the scheduler receives the resource offer, it can retrieve the earliest event from the queue, and launch that task on the offered host.
6. The task starts Rancher Mesos Executor. The executor uses QEMU-KVM to start a new VM.
7. Then it install docker on the new VM.
8. The executor then instructs the new VM to registers itself as a host with rancher server.

### Setting up and running Rancher Mesos framework

In this section, I'll show you how to setup this architecture on your laptop to try it out. We'll use [VMware fusion pro](http://store.vmware.com/store/vmware/en_US/DisplayProductDetailsPage/ThemeID.2485600/productID.304322800) to virtualize the setup as it requires changing networking configuration, and its easier to work this way.

Download the iso for [Ubuntu Desktop 14.04.2](http://www.ubuntu.com/download/desktop). In VMware fusion, select `Add > Install from disk or image`. Make sure you enable nested virtualization, and have at least 2GB of Memory before booting. 

To enable nested virtualization 

        Click on settings > 
            Processors and Memory > 
                Advanced Options > 
                    Enable Hypervisor Applications

Now boot it up. 

1. The first step in setting up is network configuration.

    We need to setup bridge networking for `eth0`. 
    
    Before continuing, ensure that `bridge-utils` is installed, using 
    
    `sudo apt-get install bridge-utils`. 

    Setup your `/etc/network/interfaces` as follows :-

        
        auto lo
        iface lo inet loopback
     
        auto eth0
        iface eth0 inet manual
      
        auto br0
        iface br0 inet dhcp
            bridge_ports eth0
            bridge_stp off
            bridge_fd 0
            bridge_maxwait 0
    

    Then run `ifup -a`, which reads the config file and sets up the bridge interface. 
    
    If you run `ifconfig` now, you'll notice there is no IP address on `eth0`, and there is a `br0` interface with a configured IP address.

    From here onwards, when I refer to `$IP`, it is the IP address on `br0` on this machine

    
2. The next step is installing the necessary packages, 

    First, you'll need git

    `sudo apt-get install git`

    To install QEMU-KVM, use this command,

        sudo apt-get install -y qemu-kvm libvirt-bin ubuntu-vm-builder bridge-utils

        echo 'allow br0' > /etc/qemu/bridge.conf

    Then, install the executor (You need [golang](https://golang.org/doc/install), [mercurial](https://help.ubuntu.com/community/Mercurial) , and [Godeps](https://github.com/tools/godep))
    
        go get -d github.com/wlan0/rancher-mesos-executor
        cd $GOPATH/src/github.com/wlan0/rancher-mesos-executor && ./scripts/build
        sudo cp build/rancher-mesos-executor /bin/ 
    
    replace executor with scheduler in the previous steps to install rancher-mesos-scheduler.
    
    Then install docker
    
        wget -qO- https://get.docker.com | sh

3. Start rancher-server

        sudo docker run -p 8080:8080 -d wlan0/rancher-server

    This will start rancher-server on port 8080

4. Install Mesos master and slave

        sudo apt-key adv --keyserver keyserver.ubuntu.com --recv E56151BF
        echo "deb http://repos.mesosphere.io/ubuntu/ trusty main" > /etc/apt/sources.list.d/mesosphere.list
        sudo apt-get -y update
        sudo apt-get -y install mesos
        service zookeeper stop
        sudo apt-get -y remove --purge zookeeper
        echo manual > /etc/init/mesos-master.override

5. Start Mesos master and slave

        sudo nohup mesos-master --work_dir=$(pwd) --ip=$IP &

        sudo nohup mesos-slave --master=$IP:5050 --ip=$IP &

6. Start the rancher-mesos scheduler. 

        CATTLE_URL=http://$IP:8080/v1 CATTLE_ACCESS_KEY=service CATTLE_SECRET_KEY=servicepass MESOS_MASTER=$IP:5050 IP_CIDR=$IP/24 rancher-mesos-scheduler

7. Now from a browser, go to `$IP:8080` and create a rackspace host with dummy credentials to see the Rancher Mesos framework in action. Note: we can add host from any cloud provider, I have short circuited the authentication part in the external handler(rancher-mesos-scheduler) to ignore the cloud type and always provision mesos hosts. Wait for a few minutes for the host to connect to cattle. Once it does, you'll be able to use this host to start containers.

8. Note that everytime you provision a host, the console for the created VM will pop up on your screen. This can be disabled for production environments.

### Next Steps

1. If you have any questions, please post them on our [forums](http://forums.rancher.com)
2. If you like to reach out to me, email me at sid@rancher.com
