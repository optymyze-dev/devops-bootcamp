# VirtualBox and CentOS Minimal
This document describes the steps required for installing a simple virtual machine on your desktop. The purpose of this virtual machine (VM) is to have a GNU/Linux machine to test different command line tools.

## Table of Contents
* [Resources](#resources)
    * [Hardware](#hardware)
    * [Software](#software)
* [Download Resources](#download-resources)
    * [VirtualBox](#virtualbox)
    * [VirtualBox Guest Addititions](#virtualbox-guest-addititions)
    * [CentOS - Minimal](#centos-minimal)
* [Validating Resources](#validating-resources)
    * [Comparing Hashes](#comparing-hashes)
* [Install](#install)
    * [Install VirtualBox](#install-virtualbox)
    * [Creating the CentOS Virtual Machine](#creating-the-centos-virtual-machine)
    * [Additional CentOS VM Configuration](#additional-centos-vm-configuration)
* [Get Familiar with GNU/Linux Tools](#get-familiar-with-gnu-linux-tools)


## Resources

### Hardware

1 GiB RAM and 10 GiB disk space should suffice.

### Software

#### VirtualBox
"Oracle VM VirtualBox (formerly Sun VirtualBox, Sun xVM VirtualBox and Innotek VirtualBox) is a free and open-source hypervisor for x86 computers currently being developed by Oracle Corporation. Developed initially by Innotek GmbH, it was acquired by Sun Microsystems in 2008 which was in turn acquired by Oracle in 2010.

VirtualBox may be installed on a number of host operating systems, including: Linux, macOS, Windows, Solaris, and OpenSolaris. There are also ports to FreeBSD[5] and Genode.[6]

It supports the creation and management of guest virtual machines running versions and derivations of Windows, Linux, BSD, OS/2, Solaris, Haiku, OSx86 and others,[7] and limited virtualization of macOS guests on Apple hardware.[8][9]

For some guest operating systems, a "Guest Additions" package of device drivers and system applications is available[10][11] which typically improves performance, especially of graphics.[12]" - https://en.wikipedia.org/wiki/VirtualBox

Website: https://www.virtualbox.org/

Manual: https://www.virtualbox.org/manual/

### VirtualBox Guest Additions
"Guest Additions are designed to be installed inside a virtual machine after the guest operating system has been installed. They consist of device drivers and system applications that optimize the guest operating system for better performance and usability." - https://www.virtualbox.org/manual/ch04.html

### CentOS (minimal)
"CentOS Linux is a community-supported distribution derived from sources freely provided to the public by Red Hat for Red Hat Enterprise Linux (RHEL). As such, CentOS Linux aims to be functionally compatible with RHEL. The CentOS Project mainly changes packages to remove upstream vendor branding and artwork. CentOS Linux is no-cost and free to redistribute. Each CentOS version is maintained for up to 10 years (by means of security updates -- the duration of the support interval by Red Hat has varied over time with respect to Sources released). A new CentOS version is released approximately every 2 years and each CentOS version is periodically updated (roughly every 6 months) to support newer hardware. This results in a secure, low-maintenance, reliable, predictable and reproducible Linux environment." - https://wiki.centos.org/

Website: https://www.centos.org/

## Download Resources

### VirtualBox
Download the installer from VirtualBox site: https://www.virtualbox.org/wiki/Downloads

 > Note on Packages for GNU/Linux distros
 > - there's no RPM for Fedora 27 yet, you can use the Fedora 26 package
 > - CentOS works with RedHat EL packages
 > - there's no deb for Ubuntu 17.10 Artful Aardvark, but it should work with the package for Ubuntu 17.04 Zesty Zapus 

You can see that, no matter the OS, all the files are located at a certain release repo address, the latest (at the moment) being: http://download.virtualbox.org/virtualbox/5.2.0/

### VirtualBox Guest Addititions
GuestAdditions ISO can be found at the repo address release mentioned above: http://download.virtualbox.org/virtualbox/5.2.0/.

Current release link: http://download.virtualbox.org/virtualbox/5.2.0/VBoxGuestAdditions_5.2.0.iso.

### CentOS - Minimal
From https://www.centos.org/download/, pick the Minimal ISO.

Current link: http://isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-Minimal-1708.iso

## Validating Resources
To test that the software downloaded was not corrupted during tranfer, we can compare the hashes of the files we have with the ones found on the official sites:

- VirtualBox & Guest Additions latest release hashes: http://download.virtualbox.org/virtualbox/5.2.0/SHA256SUMS

- CentOS:

    - Validating files: https://wiki.centos.org/TipsAndTricks/sha256sum

    - Latest release hashes: https://wiki.centos.org/Manuals/ReleaseNotes/CentOS7#head-9e717f2e95c7cb447ef66c0a0ae1315027b67684

### Comparing Hashes
GNU/Linux - Fedora:
```bash
$ cat SHA256SUMS | grep -E 'VBoxGuestAdditions_5.2.0.iso|VirtualBox-5.2-5.2.0_118431_fedora26-1.x86_64.rpm' | sha256sum -c
VBoxGuestAdditions_5.2.0.iso: OK
VirtualBox-5.2-5.2.0_118431_fedora26-1.x86_64.rpm: OK

$ sha256sum CentOS-7-x86_64-Minimal-1708.iso # compare the output with the hash from the CentOS site
```

GNU/Linux - CentOS:
```bash
$ cat SHA256SUMS | grep -E 'VBoxGuestAdditions_5.2.0.iso|VirtualBox-5.2-5.2.0_118431_el7-1.x86_64.rpm' | sha256sum -c
VBoxGuestAdditions_5.2.0.iso: OK
VirtualBox-5.2-5.2.0_118431_el7-1.x86_64.rpm: OK

$ sha256sum CentOS-7-x86_64-Minimal-1708.iso # compare the output with the hash from the CentOS site
```

GNU/Linux - Debian:
```bash
$ cat SHA256SUMS | grep -E 'VBoxGuestAdditions_5.2.0.iso|virtualbox-5.2_5.2.0-118431~Debian~stretch_amd64.deb' | sha256sum -c
VBoxGuestAdditions_5.2.0.iso: OK
virtualbox-5.2_5.2.0-118431~Debian~stretch_amd64.deb: OK

$ sha256sum CentOS-7-x86_64-Minimal-1708.iso # compare the output with the hash from the CentOS site
```

GNU/Linux - Ubuntu:
```bash
$ cat SHA256SUMS | grep -E 'VBoxGuestAdditions_5.2.0.iso|virtualbox-5.2_5.2.0-118431~Ubuntu~zesty_amd64.deb' | sha256sum -c
VBoxGuestAdditions_5.2.0.iso: OK
virtualbox-5.2_5.2.0-118431~Ubuntu~zesty_amd64.deb: OK

$ sha256sum CentOS-7-x86_64-Minimal-1708.iso # compare the output with the hash from the CentOS site
```

Windows: 

- you can use one of the tools mentioned at https://wiki.centos.org/TipsAndTricks/sha256sum

- if you have Git installed, you can go to `<git_bin>` directory, run bash.exe and use the GNU/Linux commands:
    ```bash
    $ cat SHA256SUMS | grep -E 'VBoxGuestAdditions_5.2.0.iso|VirtualBox-5.2.0-118431-Win.exe' | sha256sum -c
    VBoxGuestAdditions_5.2.0.iso: OK
    VirtualBox-5.2.0-118431-Win.exe: OK

    $ sha256sum CentOS-7-x86_64-Minimal-1708.iso # compare the output with the hash from the CentOS site
    ```

## Install
### Install VirtualBox
#### Prerequisites
For Linux distros, we need some extra components installed: https://www.virtualbox.org/manual/ch02.html#externalkernelmodules

Fedora
```bash
dnf install kernel-devel
```

CentOS
```bash
yum install kernel-devel
```

Debian/Ubuntu:
```bash
apt-get install linux-headers # pick generic
```

#### Installation
Fedora:
```bash
dnf install VirtualBox-5.2-5.2.0_118431_fedora26-1.x86_64.rpm
```

CentOS:
```
yum install VirtualBox-5.2-5.2.0_118431_el7-1.x86_64.rpm
```

Debian:
```
dpkg -i virtualbox-5.2_5.2.0-118431~Debian~stretch_amd64.deb
```

Ubuntu:
```
dpkg -i virtualbox-5.2_5.2.0-118431~Ubuntu~zesty_amd64.deb
```

Windows: just run the installer :)

### Creating the CentOS Virtual Machine
#### Start Virtual Box

#### Create a Virtual Machine
- New > Name: centos-clean, Type: Linux, Version: Red Hat (64-bit)
- Memory size: 1024 MiB
- Create a Virtual Disk Now
- VDI
- Fixed Size
- 10 GiB

#### Install CentOS
- Click on the VM

- Click Start

- On GNU/Linux, if you get this message, execute the command shown below as root or with sudo:
    ```
    Kernel driver not installed (rc=-1908)
    
    The VirtualBox Linux kernel driver (vboxdrv) is either not loaded or there is a permission problem with /dev/vboxdrv. Please reinstall the kernel module by executing
    
    '/sbin/vboxconfig'
    
    as root.
    ```

- Select start-up disk / Choose IDE > Choose Disk Image: CentOS-7-x86_64-Minimal-1708.iso

- Test this media & install CentOS 7

- Welcome - English, English (US) > Continue

- Installation Summary

    - Installation Destination > Automatic Configure Partitioning > Done

    - Localization > Europe, Bucharest > Done

    - Begin Installation

- Configuration

    - Root Password: (example - not that strong) `Str0ngP@ssw0rd`

    - User Creation

       - Full name: `Dev User`

       - User name: `dev_user`

       - check "Make this user administrator" (https://unix.stackexchange.com/questions/8581/which-is-the-safest-way-to-get-root-privileges-sudo-su-or-login#8588)

       - Password: (example) `beccainesoarepizza` (https://xkcd.com/936/)

- Complete > Reboot


### Additional CentOS VM Configuration

- Connect with `dev_user`

- Let's impersonate root, for the moment...
    ```bash
    $ sudo su -
    ```

- Check network interfaces.. We don't have any IP addresses for the main network device (enp0s3)
    ```bash
    $ ip a
    ```

- Get a dynamic IP address for enp0s3
    ```bash
    $ dhclient enp0s3
    ```

- Verify that we got an IP adddress for enp0s3
    ```bash
    $ ip a
    ```

- This IP address will be lost at reboot. Configure the network device to start on boot
    ```bash
    $ vim /etc/sysconfig/network-scripts/ifcfg-enp0s3
    -bash: vim: command not found
    ```

- By default vim is not installed (there is vi, but vim has highlighting - even though not for this type of file). Install vim
    ```bash
    $ yum install vim # just answer y where asked
    ```

- Configure the network device to start on boot (there are a lot of web pages with info on vim; this is just one: https://www.fprintf.net/vimCheatSheet.html)
    ```bash
    $ vim /etc/sysconfig/network-scripts/ifcfg-enp0s3 # change the following: disable IPV6 (we really don't need it) by changing any IPV6 yes values to no; start network device on boot
    IPV6...=no
    ONBOOT=yes
    
    
    $ vim /etc/sysctl.d/disableipv6.conf # put inside the file the following content
    net.ipv6.conf.all.disable_ipv6 = 1
    
    
    $ reboot
    ```


- Working in the VBox window is hard. Open a terminal and try to remote connect to the VM using the SSH protocol

    > For Windows, you can install Putty to connect via SSH: http://www.putty.org/

    ```bash
    # from your desktop:
    $ ssh dev_user@localhost -p2222
    ssh: connect to host localhost port 2222: Connection refused
    ```

    > The default SSH protocol listening port is 22, but not to create any confusion with the SSH server you might have started on your host (on GNU/Linux), we'll use another port  visible to your desktop (2222)

- Expose the VM's 22 port externally: VBox > settings > Network > Adapter 1 > Advanced > Port Forwarding

    - Add rule

       - Host Port: 2222

       - Guest Port: 22

- SSH again:
    ```bash
    $ ssh dev_user@localhost -p2222
    dev_user@localhost's password: 
    ```

- Impersonate root
    ```bash
    $ sudo su -
    ```

- For ease of use, make sudo for sudoers run without asking for password. In the sudoers file, you should find the following lines; comment the first configuration line using # and uncomment the last line, so everything looks like below
    ```bash
    $ vim /etc/sudoers
    ## Allows people in group wheel to run all commands
    #%wheel ALL=(ALL)       ALL

    ## Same thing without a password
    %wheel  ALL=(ALL)       NOPASSWD: ALL
    ```

- Install Guest Addtitions

    - In VBox, there's a CD icon representing the IDE device. Right click on it, and click "Choose disk image..."

    - Open the GuestAdditions iso

    - into the terminal we opened through SSH, run the following

    ```bash
    $ mkdir -p /media/cdrom # create a cdrom directory, recursivelly
    $ exit # stop using root, and make use of sudo
    $ sudo mount /dev/cdrom /media/cdrom # mount the device into the newly created directory
    mount: /dev/sr0 is write-protected, mounting read-only
    $ sudo sh /media/cdrom/VBoxLinuxAdditions.run
    Verifying archive integrity... All good.
    Uncompressing VirtualBox 5.2.0 Guest Additions for Linux........
    VirtualBox Guest Additions installer
    Copying additional installer modules ...
    ./install.sh: line 371: bzip2: command not found
    tar: This does not look like a tar archive
    tar: Exiting with failure status due to previous errors
    ./install.sh: line 384: bzip2: command not found
    tar: This does not look like a tar archive
    tar: Exiting with failure status due to previous errors
    ```

    - Install bzip 2

    ```bash
    $ sudo yum install bzip2
    ```

    - Try installing the GuestAdditions again
    ```bash
    $ sudo sh /media/cdrom/VBoxLinuxAdditions.run
    ...
    (some false positive probably from the first failure)
    Do you wish to continue? [yes or no] yes
    ...
    Copying additional installer modules ...
    Installing additional modules ...
    VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel modules.
    This system is currently not set up to build kernel modules.
    Please install the gcc make perl packages from your distribution.
    Please install the Linux kernel "header" files matching the current kernel
    for adding new hardware support to the system.
    The distribution packages containing the headers are probably:
        kernel-devel kernel-devel-3.10.0-693.el7.x86_64
    VirtualBox Guest Additions: Starting.
    VirtualBox Guest Additions: Building the VirtualBox Guest Additions kernel modules.
    This system is currently not set up to build kernel modules.
    Please install the gcc make perl packages from your distribution.
    Please install the Linux kernel "header" files matching the current kernel
    for adding new hardware support to the system.
    The distribution packages containing the headers are probably:
        kernel-devel kernel-devel-3.10.0-693.el7.x86_64
    $ sudo yum install gcc make perl
    ...
    $ sudo yum install kernel-devel kernel-devel-3.10.0-693.el7.x86_64
    ...
    $ sudo sh /media/cdrom/VBoxLinuxAdditions.run
    ...
    VirtualBox Guest Additions: Starting.
    ```

- Check and install commands

    ```bash
    $ ls /
    ...
    $ ls -la
    total 16
    drwx------. 2 dev_user dev_user  83 Nov 21 14:23 .
    drwxr-xr-x. 3 root     root      22 Nov 21 12:03 ..
    -rw-------. 1 dev_user dev_user  20 Nov 21 14:23 .bash_history
    -rw-r--r--. 1 dev_user dev_user  18 Aug  3 00:11 .bash_logout
    -rw-r--r--. 1 dev_user dev_user 193 Aug  3 00:11 .bash_profile
    -rw-r--r--. 1 dev_user dev_user 231 Aug  3 00:11 .bashrc
    $ top
    ...
    $ htop
    -bash: htop: command not found
    $ iotop
    -bash: iotop: command not found
    $ wget
    -bash: wget: command not found
    $ sudo yum install iotop wget
    ...
    $ sudo yum install htop
    ...
    No package htop available.
    Error: Nothing to do
    
    # some tools are not part of the CentoOS repo, but they should be available in RedHat Enterprise Linux repo (Extra Packages for Enterprise Linux - EPEL)
    
    $ sudo yum install epel-release # install EPEL repo

    $ sudo yum repolist # check our repo list

    $ sudo yum install htop

    $ htop

    $ sudo iotop

    $ lsblk

    $ df
    ```

## Get Familiar with GNU/Linux Tools
Now that the VM is installed and configured, the next step is to experiment with different commands that we might use in our day to day work.

This is just small list of commands. Feel free to experiment with other tools you find interesting.
```bash
ls
cd
mkdir
touch
cat
grep
sed
vim
less
tail
head
htop
iotop
curl
wget
```

