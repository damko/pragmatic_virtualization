---
title: Create your dev vm
isChild: true
anchor: create_your_development_vm
---

## Create your development vm {#create_your_development_vm_title}

As already said, the code will be on the workstation filesystem but it will be run on the VM. This means that:

1. the VM needs access to the workstation filesystem
1. the VM needs to be configured with services like Apache to run the code

Both the step will be accomplished by Vagrant with the help of Ansible.

Specifically, Vagrant will take care of:

* cloning the Vagrant template vm
* configuring the network environment for the new vm
* making available the local code folder to the new vm
* launch Ansible

Ansible, instead, will take care to install on the vanilla vm all the services and software required for the development environment (Apache, Mysql, Xdebug, etc).

### Creation, setup and configuration of a Vagrant vm via Ansible - 3rd step

Now have a look at the extra devvm directory:

    cd $fspdm1_extra/fspdm1.vm

This is its content:

    $ tree
    .
    ├── orchestration
    │   ├── ansible
    │   │   ├── playbook.yml
    │   │   ├── tasks
    │   │   │   ├── apache2.yml
    │   │   │   ├── hello_world.yml
    │   │   │   ├── init.yml
    │   │   │   ├── php5.yml
    │   │   │   └── xdebug.yml
    │   │   ├── templates
    │   │   │   ├── helloworld.dev
    │   │   │   └── standard_project.macro
    │   │   └── vars.yml
    │   └── vagrant
    │       └── Vagrantfile
    └── projects


This skeleton is arbitrary: the _orchestration_ contains whatever is needed for the creation, the configuration and the maintenance of the vm itself while the _projects_ folder contains the source code hosted by that vm and will be shared

Personally, I have a _development_ directory in my home (~) which contains a bunch of subdirs similar to the _devvm_ and each of them contains some projects. I feel comfortable with this schema, however, you can change it as you like.

The _Vagrantfile_ is already configured and it contains all the needed instructions for vagrant to build the new vm.

The _ansible_ directory contains all the scripts and the instructions for Ansible. Ansible will be executed each time the vm is started so the scripts take care of installing the missing software and updating the software over the time.

At the beginning of the _Vagrantfile_ there are some variables declarations which should be well set for any situation:

    # Vagrant template box name
    VG_BOX_NAME = "debian_wheezy_770_64bit"

    # Local virtual machine for development
    VB_DEV_VM_NAME = "dev.fspdm1.vm"
    VB_DEV_VM_IP = "192.168.51.10"

If in your network there is already a machine using the IP address 192.168.51.10 then you must change the "VB_DEV_VM_IP" variable setting an IP which does not belong to your workstation network class.


#### Changes on your workstation

**Hostname**

The hostname for the development vm will be `dev.fspdm1.vm` and it's convenient to add it to the workstation hosts list.

    # add host
    sudo echo "192.168.51.10    dev.fspdm1.vm" >> /etc/hosts


**NFS**

The NFS service is the only service that really needs to be installed on the workstation. It will share a specific folder with the development vm.

From: http://docs.vagrantup.com/v2/synced-folders/
> Synced folders enable Vagrant to sync a folder on the host machine to the guest machine, allowing you to continue working on your project's files on your host machine, but use the resources in the guest machine to compile or run your project.

This line in the Vagrantfile

    # devel.vm.synced_folder  ".", "/vagrant", disabled: true
    devel.vm.synced_folder "../../projects", "/vagrant", type: "nfs"

tells Vagrant which is the directory that should be shared with the vm right each time the vm is booted. Vagrant will automatically take care of the configuration of the NFS shares (which are set in the /etc/exports file).

#### ansible-galaxy

TODO explain https://galaxy.ansible.com/list#/roles/1119

    ansible-galaxy install kosssi.composer

#### vagrant up

Everything is ready to launch the creation of the vm with the command:

    cd orchestration/vagrant
    vagrant up

See what's happening:

1. Vagrant clones the "template" box to a Virtualbox vm which will be saved in the Virtualbox default directory (in my case ~/vms/virtualbox/)
1. Vagrant arranges some changes to the vm, like the number of cores, the amount of ram, the network card MAC address, the hostname accordingly to what set in the Vagrantfile
1. Vagrant starts the vm
1. Vagrant mounts the directory `projects` on the vm mount point `/vagrant` (as specified in the Vagrantfile)
1. Being the first `vagrant up`, once the vm is started, Vagrant runs Ansible
1. Ansible installs the additional software and configurations files

Step 1 & 2

    Bringing machine 'dev.fspdm1.vm' up with 'virtualbox' provider

    ==> dev.fspdm1.vm: Importing base box 'debian_wheezy_770_64bit'...
    ==> dev.fspdm1.vm: Matching MAC address for NAT networking...
    ==> dev.fspdm1.vm: Setting the name of the VM: dev.fspdm1.vm
    ==> dev.fspdm1.vm: Clearing any previously set network interfaces...
    ==> dev.fspdm1.vm: Preparing network interfaces based on configuration...
        dev.fspdm1.vm: Adapter 1: nat
        dev.fspdm1.vm: Adapter 2: hostonly
    ==> dev.fspdm1.vm: Forwarding ports...
        dev.fspdm1.vm: 22 => 2222 (adapter 1)
    ==> dev.fspdm1.vm: Running 'pre-boot' VM customizations...

Step 3 & 4

    ==> dev.fspdm1.vm: Booting VM...
    ==> dev.fspdm1.vm: Waiting for machine to boot. This may take a few minutes...
        dev.fspdm1.vm: SSH address: 127.0.0.1:2222
        dev.fspdm1.vm: SSH username: vagrant
        dev.fspdm1.vm: SSH auth method: private key
        dev.fspdm1.vm: Warning: Connection timeout. Retrying...
    ==> dev.fspdm1.vm: Machine booted and ready!
    GuestAdditions 4.3.10 running --- OK.
    ==> dev.fspdm1.vm: Checking for guest additions in VM...
    ==> dev.fspdm1.vm: Setting hostname...
    ==> dev.fspdm1.vm: Configuring and enabling network interfaces...
    ==> dev.fspdm1.vm: Exporting NFS shared folders...
    ==> dev.fspdm1.vm: Preparing to edit /etc/exports. Administrator privileges will be required...

Here the execution will stop: insert your Linux user's password

    nfsd running
    ==> dev.fspdm1.vm: Mounting NFS shared folders...

Step 5 & 6

    ==> dev.fspdm1.vm: Running provisioner: ansible...

    PLAY [all] ********************************************************************

    GATHERING FACTS ***************************************************************
    ok: [dev.fspdm1.vm]

    TASK: [playbook.yml | Run apt-get update if it was run last time more than 12 hours ago] ***
    ok: [dev.fspdm1.vm]

    TASK: [init.yml | Install Sys Packages] ***************************************
    changed: [dev.fspdm1.vm] => (item=curl,vim,python-pycurl,python-apt,aptitude,multitail)

    TASK: [init.yml | run Security Upgrade] ***************************************
    changed: [dev.fspdm1.vm]

    TASK: [init.yml | Ensure NTP is installed] ************************************
    ok: [dev.fspdm1.vm]

    TASK: [init.yml | Ensure NTP is running] **************************************
    changed: [dev.fspdm1.vm]

    TASK: [apache2.yml | Install Apache Packages] *********************************
    changed: [dev.fspdm1.vm] => (item=apache2,libapache2-mod-php5,libapache2-mod-macro)

    TASK: [apache2.yml | Enable Apache rewrite module] ****************************
    ok: [dev.fspdm1.vm]

    TASK: [apache2.yml | Copy Apache Macro file] **********************************
    changed: [dev.fspdm1.vm]

    TASK: [apache2.yml | Enable Apache macro module] ******************************
    ok: [dev.fspdm1.vm]

    TASK: [php5.yml | Check if php5-fpm is already installed] *********************
    failed: [dev.fspdm1.vm] => {"changed": true, "cmd": ["dpkg", "-s", "php5-fpm"], "delta": "0:00:00.010250", "end": "2014-04-28 22:11:57.861917", "failed": true, "failed_when_result": true, "item": "", "rc": 1, "start": "2014-04-28 22:11:57.851667", "stdout_lines": []}
    stderr: dpkg-query: package 'php5-fpm' is not installed and no information is available
    Use dpkg --info (= dpkg-deb --info) to examine archive files,
    and dpkg --contents (= dpkg-deb --contents) to list their contents.
    ...ignoring

    TASK: [php5.yml | Add php5 repository] ****************************************
    changed: [dev.fspdm1.vm]

    TASK: [php5.yml | Add php5 repository key] ************************************
    changed: [dev.fspdm1.vm]

    TASK: [php5.yml | Update apt] *************************************************
    ok: [dev.fspdm1.vm]

    TASK: [php5.yml | Install PHP Packages] ***************************************
    changed: [dev.fspdm1.vm] => (item=php5-fpm,php5-curl,php5-cli)

    TASK: [xdebug.yml | Install xdebug] *******************************************
    changed: [dev.fspdm1.vm] => (item=php5-xdebug)

    TASK: [hello_world.yml | Create symlink /var/www/"helloworld.dev" pointing to /vagrant/"helloworld"/] ***
    changed: [dev.fspdm1.vm]

    TASK: [hello_world.yml | Disable Apache default virtualhost file."] ***********
    changed: [dev.fspdm1.vm]

    TASK: [hello_world.yml | Remove Apache default virtualhost file."] ************
    changed: [dev.fspdm1.vm]

    TASK: [hello_world.yml | Remove Apache default ssl virtualhost file."] ********
    changed: [dev.fspdm1.vm]

    TASK: [hello_world.yml | Copy Apache vhost for "helloworld.dev"] **************
    changed: [dev.fspdm1.vm]

    TASK: [hello_world.yml | Enable Apache vhost "helloworld.dev"] ****************
    changed: [dev.fspdm1.vm]

    TASK: [hello_world.yml | Add host to /etc/hosts] ******************************
    changed: [dev.fspdm1.vm]

    NOTIFIED: [restart apache2] ***************************************************
    changed: [dev.fspdm1.vm]

    PLAY RECAP ********************************************************************
    dev.fspdm1.vm          : ok=24   changed=18   unreachable=0    failed=0


The process, all together, will take about 15 minutes to finish.

#### vagrant ssh

When finished you'll be able to log into the new virtual machine via ssh using the command:

    vagrant ssh

You can also log in with:

    ssh vagrant@dev.fspdm1.vm

because dev.fspdm1.vm has been already added to the /etc/hosts file. At the prompt for password type "vagrant".


#### vagrant halt or reload

If you want to stop or reboot the development vm from your workstation console, be sure to be in the vagrant folder and then run either:

    vagrant halt

or

    vagrant reload

To start the development vm after you stop it:

    vagrant up

#### vagrant provision

Vagrant launches Ansible automatically only at the first `vagrant up`. After that if you change any Ansible file or you want to update the system packages you need to run:

    vagrant provision