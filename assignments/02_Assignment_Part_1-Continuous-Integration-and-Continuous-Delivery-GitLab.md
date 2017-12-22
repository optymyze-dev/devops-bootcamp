# Continuous Integration and Continuous Delivery with GitLab

## Table of Contents
* [GitLab](#gitlab)
    * [About](#about)
    * [Vagrant Setup](#vagrant-setup)
    * [Add the VM in Hosts](#add-the-vm-in-hosts)
    * [Web Setup](#web-setup)
    * [Create a Project](#create-a-project)
    * [Class Excercise](#class-excercise)

## GitLab
GitLab is a repo manager with built-in CI and CD features. In this chapter we'll be setting up our on-premise GitLab server, create a repository and build some code.

### About
"GitLab is a web-based Git repository manager with wiki and issue tracking features, using an open source license, developed by GitLab Inc." - https://en.wikipedia.org/wiki/GitLab

"GitLab is an integrated product that unifies issues, code review, CI and CD into a single UI." - https://about.gitlab.com/about/

Website: https://about.gitlab.com/

GitLab can be installed on-premise or can be used as a service (similar to GitHub) at https://gitlab.com/users/sign_in.

### Vagrant Setup
 > Prerequisites:
 > - you should already have a Vagrant Base Box created as described in [creating-virtual-machines-using-vagrant.md](../extra/creating-virtual-machines-using-vagrant.md));
 > - minimum requirements: 4GiB of free RAM, two virtual CPU cores, ~5GiB of disk space.

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

  config.vm.network :private_network, ip: "192.169.0.200", bridge: "enp0s4"

  config.vm.provider "virtualbox" do |v|
    v.memory = 4096
    v.cpus = 2
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "70"]
  end

  config.vm.synced_folder "../../shared/gitlab_jenkins_shared", "/mnt/rpms", owner: "vagrant", group: "vagrant"

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

# add gitlab-runner to vagrant group so it can write to /mnt/rpms
usermod -G vagrant gitlab-runner

# install golang
export GOLANG_VERSION="1.9.1"
export GOTGZ="go${GOLANG_VERSION}.linux-amd64.tar.gz"
export GOINSTALLPATH="/usr/local/go"
rm -rf "${GOINSTALLPATH}"
mkdir -p "${GOINSTALLPATH}"
wget -q "https://storage.googleapis.com/golang/${GOTGZ}"
tar -C "${GOINSTALLPATH}/.." -xzf "${GOTGZ}" 
rm -f "${GOTGZ}"

# install rpm build tools
yum install -y rpmdevtools
```

Run `vagrant up`.

 > Don't worry if you get this message `Warning: Remote connection disconnect.` repeatedly. It will take some time until the machine is fully started.

### Add the VM in Hosts

GNU/Linux
```bash
sudo vim /etc/hosts # add the following line
192.169.0.200 gitlab.localhost
```


Windows
```
# open as Administrator c:\Windows\system32\dirvers\etc\hosts and add the following line
192.169.0.200 gitlab.localhost
```

### Web Setup
Go to http://gitlab.localhost. This may take a while. You will be asked to change your password. That is the `root` user password. Add a root password, then connect as `root` with the password created.

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
$ vagrant ssh
Last login: Thu Dec  7 10:00:00 2017 from 10.0.2.2
[vagrant@gitlab ~]$ mutt
```

Create a password, then login with `vagrant` user to the GitLab page

#### Register a Runner

Go to GitLab Page > Admin Area > Overview > Runners. Check at the registration token. We'll need it below.

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

Refresh the Runners Page (http://gitlab.localhost/admin/runners). We should have a runner in the runners list.

### Create a Project
Go to GitLab Page > Projects > Your projects > Create a Project: `hello-world`

Add a README file from the link [README](http://gitlab.localhost/vagrant/hello-world/new/master?commit_message=Add+readme.md&file_name=README.md), on the project page.

Fill the page with this content:
```md
# Hello World

This is a demo project.
```

Commit changes.


### Class Excercise
**We'll do the next steps in class.**

Create a new file called `main.go` with this content:
```go
package main

import (
	"fmt"

	"gitlab.localhost/vagrant/hello-world/calc"
)

var Version = "0.1"

func main() {
	fmt.Printf("Hello, World! Version %s\n", Version)
	a, b := 1, 2
	fmt.Printf("%d + %d = %d\n", a, b, calc.Sum(a, b))
}
```

Create a new file called `calc/calc.go` with this content:
```go
package calc

// Sum returns the sum of two integers
func Sum(a, b int) int {
	return a + b + 1
}
```

Create a new file called `calc/calc_test.go` with this content:
```go
package calc

import (
	"testing"
)

// TestSum tests the Sum function
func TestSum(t *testing.T) {
	testCases := []struct {
		a              int
		b              int
		expectedResult int
	}{
		{
			a:              1,
			b:              2,
			expectedResult: 3,
		},
		{
			a:              -1,
			b:              -2,
			expectedResult: -3,
		},
	}

	for i, testCase := range testCases {
		actualResult := Sum(testCase.a, testCase.b)
		if testCase.expectedResult != actualResult {
			t.Errorf("TestSum - test case %d\n expected result: %v\n   actual result: %v\n", i,
				testCase.expectedResult, actualResult)
		}
	}
}
```

Create a new file called `helloworld.spec` with this content:
```spec
%define installpath /home/vagrant/hello-world
%define rpmname hello-world

%define vagrant_user vagrant
%define vagrant_group vagrant

Name:           %{rpmname}
Version:        0.1
Release:        1%{?dist}
Epoch:          1
Summary:        %{rpmname} demo application
Group:          System Environment/Test Tools
Packager:       John Doe <vagrant@localhost>
License:        MIT
Distribution:   John Doe
Vendor:         John Doe
URL:            http://localhost
BuildRoot:      %{_tmppath}/%{name}-%{version}-%{release}-root-%(%{__id_u} -n)
Prefix:         %{installpath}

%description
%{rpmname} is a demo application used.

%build
export GOPATH=$(pwd .)
export PATH="${PATH}:/usr/local/go/bin:${GOPATH}/bin"
cd src/gitlab.localhost/vagrant/hello-world
go install gitlab.localhost/vagrant/hello-world

%install
install -m 0750 -d $RPM_BUILD_ROOT%{installpath}
cp bin/hello-world $RPM_BUILD_ROOT%{installpath}

%clean
rm -rf $RPM_BUILD_ROOT

%pre
## check for user
%{_bindir}/getent group %{vagrant_group} >/dev/null || (printf "error: %s group not found.\n" %{vagrant_group}; exit 1)
%{_bindir}/getent passwd %{vagrant_user} >/dev/null || (printf "error: %s user not found.\n" %{vagrant_user}; exit 1)

%files
%defattr(0754,%{vagrant_user},%{vagrant_group},0755)
%{installpath}

%doc
%changelog
```

Create a new file called `.gitlab-ci.yml` with this content:
```yaml
stages:
  - test
  - build

helloworld_test_job:
  stage: test
  script:
    - rm -rf ./src
    - shopt -s extglob

    - export GOINSTALLPATH="/usr/local/go"
    - export GOPATH="$(pwd .)/hello-world"
    - rm -rf "${GOPATH}"
    - mkdir -p "${GOPATH}"

    - export PATH="${PATH}:${GOINSTALLPATH}/bin:${GOPATH}/bin"

    - export HELLOWORLDSRCPATH="${GOPATH}/src/gitlab.localhost/vagrant/hello-world"
    - mkdir -p "${HELLOWORLDSRCPATH}"

    - cp -R !(helloworld.spec|README.md|hello-world|src) "${HELLOWORLDSRCPATH}"
    - go get github.com/golang/lint/golint github.com/GeertJohan/fgt

    - cd "${HELLOWORLDSRCPATH}"
    - go vet $(go list ./... | grep -v /vendor/)
    - exit_code=0; while read line; do fgt golint $line || exit_code=1; done < <(find * -type f -name '*.go' -not -wholename 'vendor/*' -not -wholename 'Godeps/*'); if [ $exit_code -ne 0 ]; then exit $exit_code; fi
    - exit_code=0; while read line; do go test -race "${line}" || exit_code=1; done < <(find * -type f -name '*_test.go' -not -wholename 'vendor/*' -not -wholename 'Godeps/*' | sed -r 's#^(.*)/[^/]+$#gitlab.localhost/vagrant/hello-world/\1#g' | sort | uniq); exit $exit_code

helloworld_el7_build_job:
  only:
    - master@vagrant/hello-world
  stage: build
  script:
    - sed s/0.1/$CI_BUILD_REF_NAME/ -i helloworld.spec
    - sed "s/\"0.1\"/\"$CI_BUILD_REF_NAME\"/" -i main.go
    - rm -rf ~/rpmbuild/
    - rpmdev-setuptree
    - shopt -s extglob
    - mkdir -p ~/rpmbuild/BUILD/src/gitlab.localhost/vagrant/hello-world
    - cp -R !(helloworld.spec|README.md|go) ~/rpmbuild/BUILD/src/gitlab.localhost/vagrant/hello-world
    - rpmbuild -bb helloworld.spec
    - mv ~/rpmbuild/RPMS/x86_64/*.rpm /mnt/rpms
  tags:
    - el7
  only:
    - /^[0-9]+\.[0-9]+$/
  except:
    - branches
```

The build should run and fail.

Edit `main.go`:
```go
package main

import (
	"fmt"

	"gitlab.localhost/vagrant/hello-world/calc"
)

// Version - the version of the application
var Version = "0.1"

func main() {
	fmt.Printf("Hello, World! Version %s\n", Version)
	a, b := 1, 2
	fmt.Printf("%d + %d = %d\n", a, b, calc.Sum(a, b))
}
```

The build should run and fail again, at some tests.

Edit `calc/calc.go`:
```go
package calc

// Sum returns the sum of two integers
func Sum(a, b int) int {
	return a + b
}
```

Check the build. SUCCESS!


Create a tag `0.2`.


Check the build.


On VM, as vagrant, run the following:
```bash
[vagrant@gitlab ~]$ sudo yum install -y /mnt/rpms/hello-world-0.2-1.el7.centos.x86_64.rpm
...
[vagrant@gitlab ~]$ hello-world/hello-world 
Hello, World! Version 0.2
1 + 2 = 3	
[vagrant@gitlab ~]$ sudo yum remove -y hello-world
[vagrant@gitlab ~]$ ls
```

This last part (without the removal of the package), if automated, would simulate the continuous delivery in the most basic way possible.

 > Optional Homework: define a `deliver` stage in `.gitlab-ci.yml` for installing and running the RPM on the GitLab VM. Test that the last line of the output is `1 + 2 = 3`.
