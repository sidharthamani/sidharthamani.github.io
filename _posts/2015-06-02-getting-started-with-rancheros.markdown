---
layout: post
title:  "Getting started with RancherOS"
date:   2015-06-02 13:00:06
categories: rancheros
---

Right after I finished integrating Github OAuth with Rancher, I started working on creating RancherOS - a minimalistic operating system designed to run docker.

The idea for RancherOS was thought of by [@ibuildthecloud](http://github.com/ibuildthecloud) with the aim of providing the best environment to run docker. Turns out, all docker requires to function is the kernel. RancherOS embraces this by running Docker as PID1 and everything running inside of it is a container. At 20MB, the OS is really easy to distribute, orchestrate and spin up in your data centre. You can learn more about RancherOS [here.](http://rancher.com/rancher-os/)

In this blog, I will talk about spinning up RancherOS on virtualbox using docker machine. This is the easiest way to get RancherOS up and running.

### Requirements

1. VirtualBox in your path. Download and install from [VirtualBox Downloads page](https://www.virtualbox.org/wiki/Downloads)
2. Docker Machine. The machine version should be atleast [v0.3.0-rc1](https://github.com/docker/machine/releases/tag/v0.3.0-rc1). Download it from [the docker machine releases page](https://github.com/docker/machine/releases)

### Running RancherOS

1. Fetch the latest RancherOS ISO URL for machine from [the RancherOS releases page](https://github.com/rancherio/os/releases)

    At the time of writing this article, the latest release is [v0.3.1](https://github.com/rancherio/os/releases/tag/v0.3.1)

    You might notice that there are two .iso files, rancheros.iso and machine-rancheros.iso.

    machine-rancheros.iso is the file you want. This ISO has been built with special configuration for setup with docker-machine.

    Copy the link to machine-rancheros.iso.

2. Go to your console, and use this command


        docker-machine create -d virtualbox --virtualbox-boot2docker-url $RANCHEROS-ISO-URL $MACHINE-NAME


    Note that, For the v0.3.1 release of RancherOS, the value for 
    
    ```RANCHEROS-ISO-URL``` is ```https://github.com/rancherio/os/releases/tag/v0.3.1```
    ```MACHINE-NAME``` is the name that you would like to call your machine. 

That's it. You have a RancherOS host running on virtualbox now. You can verify that you have a VirtualBox VM running on yor host using this command

``` VBoxManage list runningvms | grep $MACHINE-NAME```

It should print out the newly crated machine. If not, something went wrong with the provisioning step.

### Logging into RancherOS

Logging into RancherOS follows the standard docker-machine way. Use this command to login into your newly provisioned RancherOS VM.

```docker-machine ssh $MACHINE-NAME```

This will log you into the RancherOS VM. You'll then be able to explore the OS, run commands, spin up containers etc.

Once you've finished exploring, exit by pressing `Ctrl+D`

### Spinning up containers on RancherOS

You can point the docker client on your host to the docker daemon running inside of the VM. That way, you can run your docker commands like you had installed docker on your host. 

To point your docker client to the daemon inside the VM, use this command

```eval $(docker-machine env $MACHINE-NAME)```

You can run any docker commmand like you would normally, and it will execute your command in the RancherOS VM. 

### Congrats! Now let's run some applications remotely using docker

```docker run -p 80:80 -p 443:443 -d nginx```

This will startup nginx on your VM. In order to access it, you need the ip address of the VM.

```docker-machine ip $MACHINE-NAME```

If you copy the IP address printed from the above command and paste it in your browser, you should see a Welcome Page for nginx!

### Next Steps

The above information should help you get started with RancherOS. 

1. If you have any questions, please post them on our [google groups page](https://groups.google.com/forum/#!forum/rancherio)
2. If you want to hack on RancherOS, then visit our [github page](https://github.com/rancherio/os)
3. The Documentation for RancherOS is available in the [docs page](http://rancherio.github.io/os/)
