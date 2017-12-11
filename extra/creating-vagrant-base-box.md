# Creating a vagrant base box

**If you've just started learning about Vagrant, please refer to the new document [creating-virtual-machines-using-vagrant.md](../extra/creating-virtual-machines-using-vagrant.md)**

## Using Vagrant
Vagrant is a tool for building and managing virtual machine environments in a single workflow. With an easy-to-use workflow and focus on automation, Vagrant lowers development environment setup time, increases production parity, and makes the "works on my machine" excuse a relic of the past.

If you are already familiar with the basics of Vagrant, the [documentation](https://www.vagrantup.com/docs/index.html "documentation") provides a better reference build for all available features and internals.

In this guide we will create a vagrant base box using CentOS 7. You can also download directly from either [Vagrant Cloud](https://app.vagrantup.com/centos/boxes/7) or from the CentOS team directly a pre-build [image](https://seven.centos.org/2017/10/updated-centos-vagrant-images-available-v1710-01/).

To build the image we will be following official vagrant guides available here:
* [Box Format](https://www.vagrantup.com/docs/boxes/format.html)
* [Creating a Base Box](https://www.vagrantup.com/docs/virtualbox/boxes.html)
* [Vagrant Base Images](https://www.vagrantup.com/docs/boxes/base.html) 

## Creating a VM with CentOS 7 installed.

This step builds upon the the previous guide on how to install a VM. 

Let's create a new virtual machine which will be the base of our Vagrant Box. We will use the same procedure as described [here](https://github.com/optymyze-dev/devops-bootcamp/blob/master/extra/virtualbox-and-centos-minimal.md "here") with some differences however:

* Instead of the default 10GB Disk, we should use a 30 GB Disk which is "dynamically allocated" using the VDI format.
  * Dynamically allocated means that the disk will grow as needed until it reaches 30 GB. Initially it will take less space and a fixed sized disk but will grow with usage. This also includes a performance impact as growing the disk will be slower and allocating it all at once.
* Instead of creating the dev_user, create a user named "vagrant" with the password "vagrant".

## Configuring the VM to be packaged for Vagrant

Once the OS is installed and we can login as the root user, or the vagrant user, the following steps should be followed in order to prepare the vm for Vagrant.

* Ensure that you have a working IP address in your virtual box by verify that we got an IP adddress for enp0s3. Ensure the network is configured to start at boot time. For details see the [documentation](https://github.com/optymyze-dev/devops-bootcamp/blob/master/extra/virtualbox-and-centos-minimal.md "documentation") 
* To connect using ssh to the Virtual Box image, first we need to configure port forwarding so we can connect to it using a ssh client. To do this perform the following steps:
  * Click the Settings option in Virtual Box, once the VM is selected.
  * Navigate to the network section and expand the advanced section. Note there that your network configuration should be set to NAT.
  * Click the Port Forwarding Button to open the Port Forwarding rules section.
  * Press the button in the Right with the "+" sign to add a new rule.
  * Under the "Host Port" section enter port 2222, while under the "Guest Port" section enter port 22. This will forward all traffic from port 2222 on our machine, inside the VM on port 22 (which is the default ssh server port).
  * Open your terminal emulator software (on windows please make sure you have putty installed https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html) and connect to 127.0.0.1 on port 2222.
* Install Virtual Box Guest Additions as per the documentation available [here](https://github.com/optymyze-dev/devops-bootcamp/blob/master/extra/virtualbox-and-centos-minimal.md). 
* Configure the vagrant user:
  * Download the insecure key from https://github.com/mitchellh/vagrant/tree/master/keys
	``` sh
	# as the vagrant user run:
	cd /home/vagrant
	# install wget
	sudo yum install wget
	# create ssh directory to user home (/home/vagrant/.ssh)
	mkdir .ssh
	# download insecure key and save it to authorized_keys
	wget https://raw.githubusercontent.com/hashicorp/vagrant/master/keys/vagrant.pub	-O ~/.ssh/authorized_keys
	# Set permissions on ssh
	chmod 0700 ~/.ssh
	chmod 0600 ~/.ssh/authorized_keys
	```
  * Password-less Sudo
	*This is important!*. Many aspects of Vagrant expect the default SSH user to have passwordless sudo configured. This lets Vagrant configure networks, mount synced folders, install software, and more.
	To begin, some minimal installations of operating systems do not even include sudo by default. Verify that you install sudo in some way.
	After installing sudo, configure it (usually using visudo) to allow passwordless sudo for the "vagrant" user. This can be done with the following line at the end of the configuration file (/etc/sudoers):
	```sh
	vagrant ALL=(ALL) NOPASSWD: ALL
	```
* Install third party software
  * Install puppet since this will be used in the future. To install puppet run the following commands:
	* Add the puppet repository to the machine
	```sh
	# install repository
	sudo rpm -Uvh https://yum.puppet.com/puppet5/puppet5-release-el-7.noarch.rpm
	# install puppet agent
	sudo yum install puppet-agent
	# Add puppet binaries to the path variable in the vagrant user home
	# make sure we're in the vagrant user home
	[vagrant@localhost ~]$ pwd
	/home/vagrant
	# update the path variable to use the puppet binaries
	echo 'export PATH=/opt/puppetlabs/bin:$PATH'>> .bashrc
	```
* Package the box
  * Run the box package command with the virtual machine name
  ```sh
  # Create base box. This will save it to the current directory. 
	[user@localhost ~]$ vagrant package --base centos-clean
	==> centos-clean: Attempting graceful shutdown of VM...
	==> centos-clean: Clearing any previously set forwarded ports...
	==> centos-clean: Exporting VM...
	==> centos-clean: Compressing package to: /Users/<user>/Documents/_workspace/misc/puppet/package.box 
  # save the box
	[user@localhost ~]$ vagrant box add centos-clean /Users/<user>/Documents/_workspace/misc/puppet/package.box
	==> box: Box file was not detected as metadata. Adding it directly...
	==> box: Adding box 'centos-clean' (v0) for provider:
		box: Unpacking necessary files from: file:///Users/<user>/Documents/_workspace/misc/puppet/package.box
	==> box: Successfully added box 'centos-clean' (v0) for 'virtualbox'! 
  ```
  * Testing the box
  ```sh
  [user@localhost ~]$ vagrant init centos-clean
	A `Vagrantfile` has been placed in this directory. You are now
	ready to `vagrant up` your first virtual environment! Please read
	the comments in the Vagrantfile as well as documentation on
	`vagrantup.com` for more information on using Vagrant.
  [user@localhost ~]$	vagrant up
	Bringing machine 'default' up with 'virtualbox' provider...
	==> default: Importing base box 'centos-clean'...
	==> default: Matching MAC address for NAT networking...
	==> default: Setting the name of the VM: puppet_default_1511873672602_57381
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
		default: /vagrant => /Users/<user>/Documents/_workspace/misc/puppet
  ```
