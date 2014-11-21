---
title: Destroy your dev vm
isChild: true
anchor: destroy_your_development_vm
---

## Destroy your development vm {#destroy_your_development_vm_title}

You don't know anything until you can redo it yourself, right? So let's see how to destroy what done till now.

### How to identify the correct vm

Careful here because it's a bit tricky: to destroy a box it's important to know the box identification name so that you can say which one you want to delete.

Currently we have the Vagrant/Virtualbox "development" vm and the Vagrant "template" box.

The two vms have different names and they are written in the Vagrantfile

    # Vagrant template box name
    VG_BOX_NAME = "debian_wheezy_770_64bit"

    # Local virtual machine for development
    VB_DEV_VM_NAME = "dev.fspdm1.vm"


How to ask Virtualbox and Vagrant to list the current vm hosted in the system?

To list the Virtualbox vms:

    vboxmanage list vms

Output:

> "dev.fspdm1.vm" {472a10a0-6958-4e62-9219-54d50bee4928}

In the output you'll see all the Virtualbox vms made by hand and those made by Vagrant.


To list the Vagrant boxes (or "templates"):

    vagrant box list

Output:

> debian_wheezy_770_64bit (virtualbox, 0)

In the output you'll basically see something similar to what you see if you run

    ls -lh ~/.vagrant.d/boxes/


### Destroy the development vm

So, the current target is to destroy the Vagrant "development" vm and this can be done with Vagrant but you can only destroy those Virtualbox vms specified in the Vagrantfile. You can not destroy any Virtualbox vm you like.

    cd $fspdm1/orchestration/vagrant/
    vagrant destroy dev.fspdm1.vm

Output:

>   dev.fspdm1.vm: Are you sure you want to destroy the 'dev.fspdm1.vm' VM? [y/N]
    ==> dev.fspdm1.vm: Forcing shutdown of VM...
    ==> dev.fspdm1.vm: Destroying VM and associated drives...
    ==> dev.fspdm1.vm: Pruning invalid NFS exports. Administrator privileges will be required...
    [sudo] password for damko:
    ==> dev.fspdm1.vm: Running cleanup tasks for 'ansible' provisioner...

Run again:

    vboxmanage list vms

Now you should not see dev.fspdm1.vm in the list.

If you try to destroy a vm that is not listed in the Vagrantfile, like this

    vagrant destroy precise64_default_1396556373306_80830

you get

>   The machine with the name 'precise64_default_1396556373306_80830' was not found configured for this Vagrant environment.

### Destroy the Vagrant "template" box

If the target is to remove the "template" box then this is the command

    vagrant box remove debian_wheezy_770_64bit

and it can be executed from any directory because it doesn't need the Vagrantfile.

Output:

    Removing box 'debian_wheezy_770_64bit' (v0) with provider 'virtualbox'...

Run again:

    vagrant box list

Now you should not see debian_wheezy_770_64bit in the list.