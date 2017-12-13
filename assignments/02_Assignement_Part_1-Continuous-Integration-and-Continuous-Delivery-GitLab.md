# Continuous Integration and Continuous Delivery with GitLab

## Table of Contents
* [GitLab](#gitlab)
    * [About](#about)
    * [Vagrant Setup](#vagrant-setup)
    * [Add the VM in Hosts](#add-the-vm-in-hosts)
    * [Web Setup](#web-setup)
    * [Create a Project](#create-a-project)

## GitLab
GitLab is a repo manager with built-in CI and CD features. In this chapter we'll be setting up our on-premise GitLab server, create a repository and build some code.

### About
"GitLab is a web-based Git repository manager with wiki and issue tracking features, using an open source license, developed by GitLab Inc." - https://en.wikipedia.org/wiki/GitLab

"GitLab is an integrated product that unifies issues, code review, CI and CD into a single UI." - https://about.gitlab.com/about/

Website: https://about.gitlab.com/

GitLab can be installed on-premise or can be used as a service (similar to GitHub) at https://gitlab.com/users/sign_in.

### Vagrant Setup
 > Prerequisites: you should already have a Vagrant Base Box created as described in [creating-virtual-machines-using-vagrant.md](../extra/creating-virtual-machines-using-vagrant.md)).

Create a new VM dir: `centos-gitlab`. Check the directory structure below.

Create a new directory outside the VM dir that will be used as a shared folder between the host and VMs (e.g. `shared/gitlab_jenkins_shared`). Check the directory structure below.

Example of the directory structure (!!! make sure your VMs dir and shared dir are in the same parent dir, otherwise you must change the Vagrantfile file's synced_folder):
```bash
...
├── shared
│   └── gitlab_jenkins_shared
└── VMs
    ├── centos-gitlab
...
```

Inside `centos-gitlab`...

Create a `Vagrantfile`:
```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "centos-base-box"
  config.vm.host_name = "gitlab.localhost"
  config.vm.define "gitlab.localhost"

  config.vm.network :private_network, ip: "192.168.0.200", bridge: "enp0s4"

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "70"]
  end

  config.vm.synced_folder "../shared/gitlab_jenkins_shared", "/mnt/rpms", owner: "vagrant", group: "vagrant"

  config.vm.provision :shell, path: "bootstrap.sh"
end
```

Create `bootstrap.sh` in the same directory with the Vagrantfile:
```bash
#!/bin/bash

# gitlab requirements
yum install -y curl policycoreutils-python openssh-server
systemctl enable sshd
systemctl start sshd
firewall-cmd --permanent --add-service=http
systemctl reload firewalld

# install postfix for sending e-mails
yum install postfix
systemctl enable postfix
sed -i 's/inet_protocols = all/inet_protocols = ipv4/' /etc/postfix/main.cf
systemctl start postfix

# install an e-mail client
yum install -y mutt 

# install gitlab repo
curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ce/script.rpm.sh | bash

# install gitlab
EXTERNAL_URL="http://gitlab.localhost" yum install -y gitlab-ce

# configure gitlab
gitlab-ctl reconfigure

# install gitlab-runner repo
curl -L https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.rpm.sh | bash

# install gitlab-runner
yum install -y gitlab-runner
```

Run `vagrant up`.


### Add the VM in Hosts

GNU/Linux
```bash
sudo vim /etc/hosts # add the following line
192.168.0.200 gitlab.localhost
```


Windows
```
# open as Administrator c:\Windows\system32\dirvers\etc\hosts and add the following line
192.168.0.200 gitlab.localhost
```

### Web Setup
Go to http://gitlab.localhost. This may take a while. Add a root passowrd, then connect as `root` with the password created.

#### Create an Admin User

GitLab Welcome Page > Add people

Fill the form:

- Name: `Vagrant`

- Username: `vagrant`

- Email: `vagrant@localhost`

- Access level: `Admin`

Press **Create User**.

Sign out.


#### Login with the Admin User

Check e-mail on VM (with vagrant) for credentials
```bash
mutt
```

Create a password, then login with `vagrant` user to the GitLab page

#### Register a Runner

Go to GitLab Page > Admin Area > Overview > Runners

 > Docs: https://docs.gitlab.com/runner/register/index.html

```bash
[vagrant@gitlab ~]$ sudo gitlab-runner register
Running in system-mode.
                                                   
Please enter the gitlab-ci coordinator URL (e.g. https://gitlab.com/):
http://gitlab.localhost
Please enter the gitlab-ci token for this runner:
<copy_paste_the_token_from_runners_page>
Please enter the gitlab-ci description for this runner:
[gitlab.localhost]: First Runner
Please enter the gitlab-ci tags for this runner (comma separated):
el7 
Whether to run untagged builds [true/false]:
[false]: true
Whether to lock the Runner to current project [true/false]:
[true]: false
Registering runner... succeeded                     runner=3ZRZzRF5
Please enter the executor: docker, docker+machine, shell, ssh, virtualbox, docker-ssh+machine, kubernetes, docker-ssh, parallels:
shell
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!
```

Refresh the Runners Page (http://gitlab.localhost/admin/runners).

### Create a Project
Create a Project: `hello-world`

Add a README file from the link [README](http://gitlab.localhost/vagrant/hello-world/new/master?commit_message=Add+readme.md&file_name=README.md), on the project page.

Fill the page with this content:
```md
# Hello World

This is a demo project.
```

Commit changes.

To be continued...


