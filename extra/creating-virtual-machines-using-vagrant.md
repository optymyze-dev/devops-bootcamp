# Creating Virtual Machines Using Vagrant
In this document, you will learn how to automate the configuration of specific virtual machines based on a base image, using Vagrant.

## Table of Contents
* [Resources](#resources)
    * [Hardware](#hardware)
    * [Software](#software)
* [Download and Validate Resources](#download-and-validate-resources)
    * [VirtualBox, Guest Additions and CentOS Minimal](#virtualbox-guest-additions-and-centos-minimal-1)
    * [Vagrant](#vagrant-1)
* [Install](#install)
    * [Install VirtualBox](#install-virtualbox)
    * [Install Vagrant](#install-vagrant)
* [Vagrant Documentation](#vagrant-documentation)
* [Create a CentOS Base Box](#create-a-centos-base-box)
    * [Create the Base CentOS Virtual Machine](#create-the-base-centos-virtual-machine)
    * [Additional CentOS VM Configuration](#additional-centos-vm-configuration)
    * [Create a Base Box from CentOS VM](#create-a-base-box-from-centos-vm)
* [Create a Virtual Machine from the CentOS Base Box](#create-a-virtual-machine-from-the-centos-base-box)
* [Stop the CentOS VM Created](#stop-the-centos-vm-created)



## Resources

### Hardware
1 GiB RAM and 30 GiB disk space should be more than enough to do everything described in this document.

### Software

#### VirtualBox, Guest Additions and CentOS Minimal
In order to work with Vagrant, we will need to have already installed and configured VirtualBox, an image of VirtualBox Guest Additions and also an install image of CentOS (minimal). You can read more about them on this page: [virtualbox-and-centos-minimal.md](virtualbox-and-centos-minimal.md).

#### Vagrant
"Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past." - https://www.vagrantup.com/intro/index.html

WebSite: https://www.vagrantup.com/


## Download and Validate Resources

### VirtualBox, Guest Additions and CentOS Minimal
For downloading and validating these resources, please check the following chapters from [virtualbox-and-centos-minimal.md](virtualbox-and-centos-minimal.md):

* [Download Resources](virtualbox-and-centos-minimal.md#download-resources)

* [Validating Resources](virtualbox-and-centos-minimal.md#validating-resources)


### Vagrant
#### Download
Download Vagrant installer for your OS from the Vagrant site: https://www.vagrantup.com/downloads.html

 > Note on Packages for GNU/Linux distros
 > - Use the CentOS package for any RedHat based distro
 > - Use the Debian package for any Debian based distro


#### Validating Resources
To test that the software downloaded was not corrupted during tranfer, we can compare the hashes of the files we have with the ones found on the official sites:

- Vagrant Current release link: https://releases.hashicorp.com/vagrant/2.0.1/vagrant_2.0.1_SHA256SUMS

 > On the download page, there are also some additional resources to further check if the information is trustful: GPG signatures and keys. We won't cover them in this document. If you are curious, search for the OpenPGP standard and the GnuPG application.

##### Comparing Hashes
GNU/Linux - RedHat distro:
```bash
$ cat vagrant_2.0.1_SHA256SUMS | grep vagrant_2.0.1_x86_64.rpm | sha256sum -c
vagrant_2.0.1_x86_64.rpm: OK
```

GNU/Linux - Debian distro:
```bash
$ cat vagrant_2.0.1_SHA256SUMS | grep vagrant_2.0.1_x86_64.deb | sha256sum -c
vagrant_2.0.1_x86_64.deb: OK
```

Windows: 

- you can use one of the tools mentioned at https://wiki.centos.org/TipsAndTricks/sha256sum

- if you have Git installed, you can go to `<git_bin>` directory, run bash.exe and use the GNU/Linux commands:
    ```bash
    $ cat vagrant_2.0.1_SHA256SUMS | grep vagrant_2.0.1_x86_64.msi | sha256sum -c
    vagrant_2.0.1_x86_64.msi: OK
    ```


## Install
### Install VirtualBox
For Installing VirtualBox, please follow the steps described at [Install VirtualBox](virtualbox-and-centos-minimal.md#install-virtualbox), from [virtualbox-and-centos-minimal.md](virtualbox-and-centos-minimal.md).

### Install Vagrant
Fedora:
```bash
dnf install vagrant_2.0.1_x86_64.rpm
```

CentOS:
```
yum install vagrant_2.0.1_x86_64.rpm
```

Debian/Ubuntu:
```
dpkg -i vagrant_2.0.1_x86_64.deb
```

Windows: just run the installer :)

## Vagrant Documentation
Before continue, please skim through the documentation found on [Vagrant Docs site](https://www.vagrantup.com/docs/index.html). Have a fast look through these links:
* [Getting Started](https://www.vagrantup.com/intro/getting-started/index.html)
* [Boxes](https://www.vagrantup.com/docs/boxes.html) and [Creating a Base Box](https://www.vagrantup.com/docs/boxes/base.html)
* [Providers](https://www.vagrantup.com/docs/providers/), [VirtualBox](https://www.vagrantup.com/docs/virtualbox/) and [Creating a Base Box with VirtualBox](https://www.vagrantup.com/docs/virtualbox/boxes.html)
* [Vagrantfile](https://www.vagrantup.com/docs/vagrantfile/)

Next, we'll be using all the info we found in the Vagrant Docs to make a base image called a "base box" and use it to create a custom VM.

## Create a CentOS Base Box
Let's get the party started!

### Create the Base CentOS Virtual Machine
The creation of a CentOS Virtual Machine is covered in detail in [virtualbox-and-centos-minimal.md](virtualbox-and-centos-minimal.md):
* [Create the CentOS Virtual Machine](virtualbox-and-centos-minimal.md#create-the-centos-virtual-machine)
* [Additional CentOS VM Configuration](virtualbox-and-centos-minimal.md#additional-centos-vm-configuration)

Based on that documentation, please install the Base CentOS VM, with the following changes (**in bold**):

#### Create a Virtual Machine
- New > Name: **centos-base**, Type: Linux, Version: Red Hat (64-bit)
- Memory size: 1024 MiB
- Create a Virtual Disk Now
- VDI
- **Dynamically allocated**
- 30 GiB

#### Install CentOS

... (same steps as in the document linked above)

- Configuration

    - **Root Password - DON'T CREATE A ROOT PASSWORD! We are going to use only the vagrant user** 

    - User Creation

        - **Full name: `Vagrant`**

        - **User name: `vagrant`**

        - check "Make this user administrator" (https://unix.stackexchange.com/questions/8581/which-is-the-safest-way-to-get-root-privileges-sudo-su-or-login#8588)

        - **Password: `vagrant`**

... (same steps as in the document linked above)

### Additional CentOS VM Configuration

- **Connect with `vagrant`**

- **Fast config - we'll skip all the details of the commands. Most of them were detailed in the [virtualbox-and-centos-minimal.md](virtualbox-and-centos-minimal.md) document**

    ```bash
    $ sudo dhclient enp0s3

    $ sudo yum install vim

    $ sudo vim /etc/sudoers # Password-less Sudo - check these lines to look like this
    ## Allows people in group wheel to run all commands
    #%wheel ALL=(ALL)       ALL

    ## Same thing without a password
    %wheel  ALL=(ALL)       NOPASSWD: ALL


    $ sudo vim /etc/sysconfig/network-scripts/ifcfg-enp0s3 # change the following: disable IPV6 (we really don't need it) by changing any IPV6 yes values to no; start network device on boot
    IPV6...=no
    ONBOOT=yes


    $ sudo vim /etc/sysctl.d/disableipv6.conf # put inside the file the following content
    net.ipv6.conf.all.disable_ipv6 = 1

    ```

- **Expose the VM's 22 port externally for ease of use of SSH**: VBox > Select the VM > Settings > Network > Adapter 1 > Advanced > Port Forwarding

    - Add rule

        - Host Port: 2222

        - Guest Port: 22


- **Reboot**
    ```bash
    $ reboot
    ```

- **Connect through SSH**
    ```bash
    $ ssh vagrant@localhost -p2222
    ```

- **Install Guest additions and other apps**
    ```bash
    $ sudo yum install bzip2 gcc make perl kernel-devel kernel-devel-3.10.0-693.el7.x86_64 -y


    # In VBox, there's a CD icon representing the IDE device. Right click on it, and click "Choose disk image..."

    # Open the GuestAdditions iso

    # into the terminal we opened through SSH, continue with

    $ sudo mkdir -p /media/cdrom

    $ sudo mount /dev/cdrom /media/cdrom
    mount: /dev/sr0 is write-protected, mounting read-only

    $ sudo sh /media/cdrom/VBoxLinuxAdditions.run
    ...
    VirtualBox Guest Additions: Starting.

    $ sudo umount /media/cdrom

    $ sudo yum install epel-release -y # install EPEL repo

    $ sudo yum install htop iotop wget -y

    ```

- **"vagrant" User Key Authentication Setup**
    ```bash
    # by default, if we didn't changed directory, we should be in vagrant's home, but, to be sure...
	$ cd /home/oracle

	# create ssh directory to user home (/home/vagrant/.ssh)
	mkdir .ssh

	# download insecure key and save it to authorized_keys
	wget https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub -O .ssh/authorized_keys
	
    # Set the right permissions
	chmod 0700 .ssh
	chmod 0600 .ssh/authorized_keys
	```

- **ShutDown the VM**
    ```bash
    $ sudo shutdown now
    ```

### Create a Base Box from CentOS VM
Now that we have our VM all prepped, we are ready to create a Vagrant box.

- On your host machine (your dekstop OS), create a directory where you want your Vagrant files for this box to be saved.
    ```bash
    # GNU/Linux example:
    $ mkdir -p ~/Vagrant/CentOSBaseBox
    ```

- Open a console (if you are not in one yet) and go to that directory
    ```bash
    # GNU/Linux example:
    $ cd ~/Vagrant/CentOSBaseBox
    ```

- Create the box. This can take a while
    ```bash
    # vagrant package --base <vm_name>
    $ vagrant package --base centos-base
    ==> centos-base: Clearing any previously set forwarded ports...
    ==> centos-base: Exporting VM...
    ==> centos-base: Compressing package to: /home/<user>/Vagrant/CentOSBaseBox/package.box
    ```

- Register your Vagrant box with Vagrant
    ```bash
    # vagrant box add <registered_name_for_vagrant_box> <path_to_box>
    # run 'vagrant box add --help' to see more options
    $ vagrant box add centos-base-box package.box
    ==> box: Box file was not detected as metadata. Adding it directly...
    ==> box: Adding box 'centos-base-box' (v0) for provider: 
        box: Unpacking necessary files from: file:///home/<user>/Vagrant/CentOSBaseBox/package.box
    ==> box: Successfully added box 'centos-base-box' (v0) for 'virtualbox'!
    ```

- Check for the registered box
    ```bash
    $ vagrant box list
    centos-base-box (virtualbox, 0)
    ```

## Create a Virtual Machine from the CentOS Base Box 
If a Vagrant box can be seen as a class from a programming language, the VM created using a box is the instance of that class.

- On your host machine (your dekstop OS), create a directory where you want your Vagrant files for this VM to be saved.
    ```bash
    # GNU/Linux example:
    $ mkdir -p ~/Vagrant/VMs/CentOSVM1
    ```

- Open a console (if you are not in one yet) and go to that directory
    ```bash
    # GNU/Linux example:
    $ cd ~/Vagrant/VMs/CentOSVM1
    ```

- Create the Vagrant files for the VM
    ```bash
    $ vagrant init centos-base-box
    A `Vagrantfile` has been placed in this directory. You are now
    ready to `vagrant up` your first virtual environment! Please read
    the comments in the Vagrantfile as well as documentation on
    `vagrantup.com` for more information on using Vagrant.
    ```

- Create the VM using the generated Vagrantfile
    ```bash
    $ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Importing base box 'centos-base-box'...
    ==> default: Matching MAC address for NAT networking...
    ==> default: Setting the name of the VM: CentOSVM1_default_1512641324426_44971
    ==> default: Clearing any previously set network interfaces...
    ==> default: Preparing network interfaces based on configuration...
        default: Adapter 1: nat
    ==> default: Forwarding ports...
        default: 22 (guest) => 2222 (host) (adapter 1)
    ==> default: Booting VM...
    ==> default: Waiting for machine to boot. This may take a few minutes...
        default: SSH address: 127.0.0.1:2222
        default: SSH username: vagrant
        default: SSH auth method: private key
        default: 
        default: Vagrant insecure key detected. Vagrant will automatically replace
        default: this with a newly generated keypair for better security.
        default: 
        default: Inserting generated public key within guest...
        default: Removing insecure key from the guest if it's present...
        default: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> default: Machine booted and ready!
    ==> default: Checking for guest additions in VM...
    ==> default: Mounting shared folders...
        default: /vagrant => /home/radu/Vagrant/VMs/CentOSVM1
    ```
    As you can see, by default, Vagrant is doing port fowarding... Nice!

- Check the VirtualBox. You should see a new VM started.

- Test SSH to the VM
    ```bash
    $ ssh vagrant@localhost -p2222
    
    [vagrant@localhost ~]$ vagrant@localhost's password: 
    Last login: Thu Dec  7 10:00:00 2017 from 10.0.2.2
    [vagrant@localhost ~]$ exit
    logout
    Connection to localhost closed.
    ```

- Test SSH using Vagrant. This will use key authentication
    ```bash
    $ vagrant ssh
    Last login: Thu Dec  7 12:00:00 2017 from 10.0.2.2
    [vagrant@localhost ~]$ exit
    logout
    Connection to 127.0.0.1 closed.
    ```

## Stop the CentOS VM Created
There are a few Vagrant commands which manage the state of the VM.
```
vagrant suspend - it's equivalent with VirtualBox's Pause. It pausing the VM, saving the current state of the VM
vagrant resume  - resumes a suspended VM
vagrant halt    - stops the machine
vagrant up      - start the machine
vagrant destroy - stops and deletes all traces of the vagrant machine
```

 > Note that `vagrant up` works a little different after you created the VM. First time, it provisions the machine based on the configuration found in Vagrantfile. Running `vagrant up` after the VM was halted will start the VM **without** provisioning. Only if the VM is deleted/destroyed, and `vagrant up` will go through the provisioning process again. To re-provision an existing VM, you can use `vagrant provision`.

Now, stop your VM
```bash
$ vagrant halt
==> default: Attempting graceful shutdown of VM...
```

In VirtualBox, you should see now the VM created using Vagrant is "Powered Off".
