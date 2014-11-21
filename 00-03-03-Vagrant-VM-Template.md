---
title: Vagrant VM Template
isChild: true
anchor: vagrant_vm_template
---

TODO I should say something about the creation of a Vagrant box using http://www.vagrantbox.es/

## Vagrant VM Template {#vagrant_vm_template_title}

Now that the virtualization tools are installed you can start creating your development box.

Before continuing checkout the manual extra at the _virtualization_ tag

    cd $fspdm1_extra
    git checkout virtualization


You'll be following these steps:

1. creation of a box using Packer
1. import of the Packer box into Vagrant and Virtualbox
1. creation, setup and configuration of a Vagrant vm via Ansible


### Creation of a box using Packer - 1st step

You are now going to use Packer to create a sort of "fresh" Debian virtual machine compatible with Virtualbox and Vagrant.

Accordingly to the instructions contained in the ta-debian-770-wheezy.json file (you should have a look at it) Packer will download a net-install iso from the Debian servers and will use that iso to launch the installation process and create the virtual machine image.

Basically Packer will do what you do when you install Debian from a net-install cd and will adjust it for the selected environment (in our case Virtualbox and Vagrant).

#### Get the Packer scripts

    cd $fspdm1_extra
    git clone git@github.com:damko/packer-templates.git
    cd packer-templates

If you have a look at the json file you'll see that the Packer box is configured to have 1 core, 512MB ram and 10Gb hdd but it's easy to add a core or some ram later on.

Before creating the vm let's check if the json file is valid

    cd $fspdm1_extra/packer_templates
    packer validate ta-debian-7-wheezy-virtualbox.json

It should output "Template validated successfully."

Finally this is the command to create the fresh vm:

    packer build ta-debian-7-wheezy-virtualbox.json

The command will take a while because of the download and the installation but in the end it will show this message:

    ==> Builds finished. The artifacts of successful builds are:
    --> virtualbox-iso: 'virtualbox' provider box: debian-770-wheezy.box

_debian-770-wheezy.box_ is a file about 450MB big which lives inside the packer_template directory and contains the fresh virtual machine.

*Notes*:

1. The debian iso file name contains the version number and, as soon as a new release will be out and the 770 will be removed from the debian servers, the debian-770-wheezy.json file will be outdated and you'll get the "ISO download failed" error after running the build command.
To fix the issue go on http://cdimage.debian.org/debian-cd/current/amd64/iso-cd/, check which is the latest net-inst version and copy its checksum from the MD5SUMS file. Then edit the .json file and update these variables:

    * "iso_url": update the link to the iso file
    * "iso_md5": insert the new MD5 checksum
    * "vm_name": update the version

1. During the build process it is possible that the output will pause for quite a long time ( 5 minutes or so ) at the line _Waiting for SSH to become available..._. Don't quit the process and be patient.



### Import of the Packer box into Vagrant and Virtualbox - 2nd step

Now that you have the debian-770-wheezy.box file, the next step is to tell Vagrant to import the Packer box and to make it available in Vagrant too.

    vagrant box add debian_wheezy_770_64bit ./debian-770-wheezy.box

This will convert the Packer box in a Vagrant box and will make it available in your system. Imagine this Vagrant box as a "template" for a fresh Debian Wheezy 64bit based virtual machine that can be used to create as many Vagrant boxes as you want.

*Note:*

Where is the "vagrant template vm" located on the workstation disk?

    ls -lh ~/.vagrant.d/boxes

Output:

> drwxr-xr-x 3 damko damko 4.0K 2014-11-17 20:27:33 debian_wheezy_770_64bit

    du -sh ~/.vagrant.d/boxes/*

Output:

> 442M    /home/damko/.vagrant.d/boxes/debian_wheezy_770_64bit

Now that the original Packer box has been added to the Vagrant boxes the file debian-770-wheezy.box can be deleted.

### Recap

A little recap of what done till now:

* Packager created a virtual machine starting from the debian iso cd.
* Vagrant imported the Packager box into its list of available virtual machine templates
