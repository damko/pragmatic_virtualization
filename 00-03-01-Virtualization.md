---
anchor: virtualization
---

# Virtualization {#virtualization_title}

## What's virtualization

The term _Virtualization_ refers to the act of creating a virtual version of something, including but not limited to a virtual computer hardware platform, operating system, storage device, or computer network resources. [Wikipedia](https://en.wikipedia.org/wiki/Virtualization)

In this manual "_virtualization_" means "_hardware virtualization_"  which is the creation of a virtual machine that acts like a real computer with its own operating system. Software executed on virtual machines is separated from the underlying hardware resources and the whole virtual machine is stored in a few files located on the hard-drive of the host system.

Basically with virtualization you are running virtual computers inside your own computer.

## Why to use the virtualization

A common good practice during the development of a web application is to produce separated environments at least for the development and the production.

The development environment is the environment used by the developers to create and test the web application while the production environment is the server on which the final web application is hosted and published.

In this case "environment" means pretty much everything:

* the operating system in use
* the applications and services installed in the system and their version
* the libraries used in the web application and their version

Running a web application on different environments, in development and production, can lead to strange bugs popping up when you go live.

Virtualization is the answer to the problem of inhomogeneous environments.

Especially when working in a team, virtualization offers other great benefits:

* all the team members can work in the same exact environment even when the team mates are in different locations
* any change to an environment can be easily replicated in the team members virtual machines
* to share different environments or environment updates among team mates is enough to provide a few configuration files

Virtualization also grants **isolation**: each project can be run in isolation inside its own virtual machine and the developer's workstation is totally decoupled from the projects. To clarify consider this case: you have an application requiring with PHP 5.3 and another one requiring with PHP 5.6. You can set up two different virtual machines, one with PHP 5.3 and the other one with PHP 5.6.

## Virtualization tools

The most famous virtualization environments are [VirtualBox](http://www.virtualbox.org/), [VMware](http://www.vmware.com/) and [Docker](http://www.docker.com/) but this manual will focus only on Virtualbox.

Virtualbox and other two applications called Vagrant and Packer provide what's necessary to create, clone, run, maintain and destroy your virtual machines (aka. boxes or vm/vms) in an easy and very predictable way.

At the end of the Virtualization chapter you will have a Virtualbox virtual machine managed by Vagrant which will be used as *development environment* for your applications.

Your applications code will be stored on your workstation filesystem and you will edit it locally using an editor installed on your workstation.

The applications code will be run by the virtual machine and this explains why you don't need to install services like Apache or Mysql on your workstation: they will run on the vm.

## Virtualbox

Virtualbox offers "full virtualization" which is a particular kind of virtualization that allows an unmodified operating system with all of its installed software to run in a special environment, on top of your existing operating system. Your existing operating system is called _host_ while the virtual machines are called _guests_.
This approach, often called "native virtualization", is different from mere emulation which is typically quite slow.

VirtualBox is also different from the so-called "paravirtualization" solutions (such as Xen) which require the guest operating system to be modified.

### Virtualbox installation

Download the latest package that suits at best your workstation from [here](https://www.virtualbox.org/wiki/Linux_Downloads).

Ex.:

    cd /tmp
    wget -c http://download.virtualbox.org/virtualbox/4.3.18/virtualbox-4.3_4.3.18-96516~Ubuntu~raring_amd64.deb
    sudo dpkg -i virtualbox-4.3_4.3.18-96516~Ubuntu~raring_amd64.deb

## Vagrant

[Vagrant][http://vagrantup.com/] helps you building your virtual boxes on top of Virtualbox (or VMWare). Vagrant is considered a wrapper for Virtualbox and it will configure the virtual machine(s) using a single configuration file named Vagrantfile.

Vagrant creates folders for sharing your code between your host and your virtual machine so that you can create and edit your files on your host machine and then run the code inside your virtual machine.

### Vagrant installation

Download the latest package that suits at best your workstation from [here](http://www.vagrantup.com/downloads.html) then install it like this:

Ex.:

    cd /tmp
    wget -c https://dl.bintray.com/mitchellh/vagrant/vagrant_1.5.4_x86_64.deb
    sudo dpkg -i vagrant_1.5.4_x86_64.deb


## Packer

[Packer](http://packer.io) is a tool for creating identical machine images for multiple platforms from a single source configuration. Packer automates the creation of any type of machine image: out of the box Packer comes with support to build images for Amazon EC2, DigitalOcean, Google Compute Engine, QEMU, VirtualBox, VMware, and more.

Out of the box Packer comes with support to build images for Amazon EC2, DigitalOcean, Google Compute Engine, QEMU, VirtualBox, VMware, and more.

This means that with Packer you write a bunch of configuration files (json format) and then you can programmatically create the same virtual machine for different environments like Virtualbox, Vmware etc.

In this manual Packer will be used to create a Debian based virtual machine for Virtualbox which will be then used by Vagrant as a "template" virtual machine for the creation of your environments. The customization of each environment (Vagrant virtual machine) will be done using an automation tool called Ansible (see later).

### Packer installation

Differently from Vagrant and Virtualbox, Packer doesn't come with a debian installer so the easiest way to install it in the system is to run the `install_packer.sh` script  provided in the manual extra.

Before running the script edit it and change the variables listed at the beginning of the file accordingly to your needs.