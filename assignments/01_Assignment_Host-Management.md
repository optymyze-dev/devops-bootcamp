# Assignment 1 - Host Management using Puppet

## Creating your setup.

* Create a vagrant file using vagrant init and your image (you can use the image created using this tutorial: https://github.com/optymyze-dev/devops-bootcamp/blob/master/extra/creating-vagrant-base-box.md).
  * Initialize the image using an existing box 
	```sh
	vagrant init centos-clean
	```
  * Run vagrant up in a directory. This directory will be your work space so chose it wisely.
  ```sh
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
  * Run vagrant ssh to check if everything is working fine.
  ```sh
  [user@localhost ~]$ vagrant ssh
  ```
  It should log you in the vm at this point. 

## Working with puppet
In this section we will introduce puppet and will configure an apache webserver using puppet to serve a static website.
Out of the box, Vagrant supports provisioning servers with both Puppet Apply and Puppet Agent. Since this is a Masterless Puppet workflow, we're going to be using Puppet Apply.

Vagrant's documentation does an excellent job at describing how to use the Puppet Apply provisioner(https://www.vagrantup.com/docs/provisioning/puppet_apply.html). The only gotcha I found is that Puppet must be installed on the Vagrant-based virtual machine before Puppet Apply can be used. Maybe I just missed that bit in the documentation. 

Puppet is a very complex topic and we won't cover all the details. We're just going to give you some concepts so that you can get started. More details about puppet are available at their[website](https://puppet.com/docs/puppet/5.3/bgtm.html) allong with a more indepth documentation. Probably the best way to get started is using a book (https://www.amazon.com/Puppet-4-10-Beginners-Guide-newbie/dp/1787124002 is a great refrence to get started) or [getting started guides](https://puppet.com/docs/puppet/5.3/quick_start_essential_config.html).

Before we get started we should introduce a couple of concepts:

### Puppet
Puppet is an automated administrative engine for your Linux, Unix, and Windows systems, performs administrative tasks (such as adding users, installing packages, and updating server configurations) based on a centralized specification. Puppet uses its own configuration language, which was designed to be accessible to sysadmins. The Puppet language does not require much formal programming experience and its syntax was inspired by the Nagios configuration file format.
	* Resources, classes, and nodes
The core of the Puppet language is declaring resources. Every other part of the language exists to add flexibility and convenience to the way resources are declared.

Groups of resources can be organized into classes, which are larger units of configuration. While a resource might describe a single file or package, a class can describe everything needed to configure an entire service or application (including any number of packages, config files, service daemons, and maintenance tasks). Smaller classes can then be combined into larger classes which describe entire custom system roles, such as “database server” or “web application worker.”
An example class is:
```puppet
# A class with no parameters
class base::linux {
  file { '/etc/passwd':
    owner => 'root',
    group => 'root',
    mode  => '0644',
  }
  file { '/etc/shadow':
    owner => 'root',
    group => 'root',
    mode  => '0440',
  }
}
```
The general form of a class definition is:

    The class keyword
    The name of the class
    An optional parameter list, which consists of:
        An opening parenthesis
        A comma-separated list of parameters (e.g. String $myparam = "default value"). Each parameter consists of:
            An optional data type, which will restrict the allowed values for the parameter (defaults to Any)
            A variable name to represent the parameter, including the $ prefix
            An optional equals (=) sign and default value (which must match the data type, if one was specified)
        An optional trailing comma after the last parameter
        A closing parenthesis
    Optionally, the inherits keyword followed by a single class name
    An opening curly brace
    A block of arbitrary Puppet code, which generally contains at least one resource declaration

Resources are the fundamental unit for modeling system configurations. Each resource describes some aspect of a system, like a specific service or package. The default list of resouces that puppet can manage is available [here](https://puppet.com/docs/puppet/5.3/type.html).

A resource declaration is an expression that describes the desired state for a resource and tells Puppet to add it to the catalog. When Puppet applies that catalog to a target system, it manages every resource it contains, ensuring that the actual state matches the desired state.


Nodes that serve different roles will generally get different sets of classes. The task of configuring which classes will be applied to a given node is called node classification. Nodes can be classified in the Puppet language using node definitions; they can also be classified using node-specific data from outside your manifests, such as that from an [ENC](https://puppet.com/docs/puppet/5.3/nodes_external.html) or Hiera.

### Puppet modules
Modules are self-contained bundles of code and data. These reusable, shareable units of Puppet code are a basic building block for Puppet.
Nearly all Puppet manifests belong in modules. The sole exception is the main site.pp manifest, which contains site-wide and node-specific code.
Every Puppet user should expect to write at least some of their own modules. You can also download modules that other users have built from the Puppet Forge.
Documentation on modules are available [here](https://puppet.com/docs/puppet/5.3/modules_fundamentals.html).

### Puppet master
Puppet master is a Ruby application that compiles configurations for any number of Puppet agent nodes, using Puppet code and various other data sources. Puppet uses a agent/master architecutre. When set up as an agent/master architecture, a Puppet master server controls the configuration information, and each managed agent node requests its own configuration catalog from the master.
In this architecture, managed nodes run the Puppet agent application, usually as a background service. One or more servers run the Puppet master application, Puppet Server.
Periodically, each Puppet agent sends facts to the Puppet master, and requests a catalog. The master compiles and returns that node’s catalog, using several sources of information it has access to.
Once it receives a catalog, Puppet agent applies it to the node by checking each resource the catalog describes. If it finds any resources that are not in their desired state, it makes the changes necessary to correct them. Or, in no-op mode, it reports on what changes would have been done.
After applying the catalog, the agent sends a report to the Puppet master.
To get an overview of how this works you can read the official documentation available [here](https://puppet.com/docs/puppet/5.3/architecture.html).

### Hiera
Hiera is Puppet’s built-in key/value data lookup system. By default, it uses simple YAML or JSON files, although you can extend it to work with almost any data source. Almost every successful Puppet user relies on it heavily, and you should too.
*Hiera is the config file for your Puppet code*

Puppet’s primary strength is in reusable code. But code that serves many needs has to be configurable — site-specific information should usually be in configuration data, rather than in the code itself.

Hiera is the most flexible way to get configuration data into Puppet. Puppet automatically searches Hiera for class parameters, so you can use Hiera to configure any module.
Hiera helps you avoid repetition

Hiera’s hierarchical lookups are built for a “defaults, with overrides” pattern. This lets you specify common data once, then override it in situations where the default won’t work. And since Hiera uses Puppet’s facts to specify data sources, you can structure your overrides in whatever way makes sense for your infrastructure.

More details of Hiera are available [here](https://puppet.com/docs/puppet/4.10/hiera_intro.html).

# Using your first puppet module

In this section we will use the vagrant support to run puppet code to configure an http server with a static webpage. We'll use this aproach for the sake off making it easy to get started. In order to do this we will be using the "Vagrantfile" to configure vagrant to run the puppet code. In case you're not sure what a Vagrantfile is [here](https://www.vagrantup.com/docs/vagrantfile/) is the official documentation from Vagrant. You can think of it as vangrat's configuration file which tells the software what to do when launching the VM. 

## Updating the Vagrant File

When we run the command ~vagrant init~, Vagrant automatically placed a "Vagrantfile" in the directory where this was run. This file already contains a scheleton with the major section so we will be edding it to configure the puppet integration.

The created Vagrantfile looks like this:
```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.
  config.vm.box = "centos-clean"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   apt-get update
  #   apt-get install -y apache2
  # SHELL
end
```

The first thing we will do is remove the comments and add the following configuration block:
```ruby
 config.vm.define "my.example.com"
 # we use the shell provisioner to make sure we have an uptodate system and that puppet is installed
  config.vm.provision "shell", :inline => <<-SHELL
    yum update
    yum install -y puppet
  SHELL
 # we define a puppet provisioner which will load from the current directory the modules directory (which also contains our modules) and use the hiera.yaml file to decide what to apply.
  config.vm.provision "puppet" do |puppet|
    puppet.hiera_config_path = "hiera.yaml"
    puppet.module_path = "modules"
```
The resulting Vagrant file will look like this:

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos-clean"
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.define "my.example.com"
  # we use the shell provisioner to make sure we have an uptodate system and that puppet is installed
  config.vm.provision "shell", :inline => <<-SHELL
    yum update
    yum install -y puppet
  SHELL
  # we define a puppet provisioner which will load from the current directory the modules directory (which also contains our modules) and use the hiera.yaml file to decide what to apply.
  config.vm.provision "puppet" do |puppet|
    puppet.hiera_config_path = "hiera.yaml"
    puppet.module_path = "modules"
  end 

end
```
## Configuring Hiera
The Vagrantfile defines a Hiera configuration called hiera.yaml located in the same directory as the Vagrantfile.
Here's the contents of hiera.yaml:
```yaml
---
:backends:
  - yaml

:hierarchy:
  - "common"

:yaml:
  :datadir: '/vagrant/hiera'
```
This tells hiera to look for a file called common.yaml in the /vagrant/hiera directory on the newly provisioned virtual machine. On the Vagrant-side, everything is in the same directory as the Vagrantfile.

To create the file run the following commands in the directory where the Vagranfile exists:

```sh
#create the hiera directory
 [user@localhost ~]$ mkdir hiera
#create the common.yaml file
 [user@localhost ~]$ touch hiera/common.yaml
```

Let's add some parameters in the common.yaml file that will be passed to the apache module:

```yaml
---
apache::serveradmin: 'joe@example.com'
my_message: 'Hello, World!'
```
Once this content is added to the common.yaml file, save it. The first line sets the serveradmin setting for Apache and the second line sets an arbitrary message for later use.

## Using the Apache Module

The Vagrant Puppet Apply provisioner supports hosting modules. In the Vagrantfile, the module path is configured as ./modules.

To install and configure Apache via Puppet, we're going to use the puppetlabs/apache module. This module requires the puppetlabs/stdlib and puppetlabs/concat modules, so in total, three modules will be used:
```sh
 [user@localhost ~]$ mkdir modules
 [user@localhost ~]$ cd modules
 [user@localhost ~]$ git clone https://github.com/puppetlabs/puppetlabs-apache apache
Cloning into 'apache'...
remote: Counting objects: 15650, done.
remote: Compressing objects: 100% (3/3), done.
remote: Total 15650 (delta 1), reused 0 (delta 0), pack-reused 15647
Receiving objects: 100% (15650/15650), 5.21 MiB | 4.23 MiB/s, done.
Resolving deltas: 100% (10750/10750), done.
 [user@localhost ~]$ git clone https://github.com/puppetlabs/puppetlabs-stdlib stdlib
Cloning into 'stdlib'...
remote: Counting objects: 11083, done.
remote: Compressing objects: 100% (41/41), done.
remote: Total 11083 (delta 16), reused 35 (delta 10), pack-reused 11031
Receiving objects: 100% (11083/11083), 2.44 MiB | 2.07 MiB/s, done.
Resolving deltas: 100% (5683/5683), done.
 [user@localhost ~]$ git clone https://github.com/puppetlabs/puppetlabs-concat concat
Cloning into 'concat'...
remote: Counting objects: 3102, done.
remote: Compressing objects: 100% (12/12), done.
remote: Total 3102 (delta 4), reused 9 (delta 2), pack-reused 3088
Receiving objects: 100% (3102/3102), 690.58 KiB | 823.00 KiB/s, done.
Resolving deltas: 100% (1527/1527), done.
```
At this point we have cloned from github the apache, sdtlib and concat modules and placed them in our modules directory.

The final component is an actual Puppet manifest. By default, Vagrant looks for environments/test/manifests/default.pp which is exactly what I'll use. First create the directory structure:
```sh
[user@localhost ~]$ mkdir -p environments/test/manifests
```
Inside the manifest directory we will create the default.pp file and add the following content in it.

```puppet
include apache
notify { 'my_message':
  message => lookup('my_message'),
}

file { '/var/www/html/index.html':
      content => '<html><h1>Test Page</html>',
      ensure => present,
      owner => 'apache',
      group => 'apache',
 }

```

The first line installs and configures apache using the default settings from the puppetlabs/apache module except for the serveradmin setting which was defined in Hiera as joe@example.com.
The second line prints a message stored in Hiera just as another way to show that data is successfully being pulled from Hiera.

We also have a file resouce defined, and puppet will create a file under "/var/www/html/index.html" with the content "Test Page" and grant ownership to puppet.

With everything in place, the directory structure should look like this:

```sh
[user@localhost ~]$ tree -L 2
Vagrantfile
hiera/
	common.yaml
hiera.yaml
environments/
	test/
		manifests/
			default.pp
modules/
	apache/
	concat/
	stdlib/
package.box
```
