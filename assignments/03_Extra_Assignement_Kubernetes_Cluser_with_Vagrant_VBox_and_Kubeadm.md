# Assignment 3 - Kubernetes Cluster with Vagrant, VBox and Kubeadm


## Table of Contents
* [Prerequisites](#prerequisites)
* [Docs](#docs)
* [Excercise](#exercise)


## Prerequisites
- VirtualBox
- ~4 GiB RAM free
- ~10 GiB Disk Space


## Docs
- https://www.vagrantup.com/docs/index.html
- https://kubernetes.io/docs/home/
    - https://kubernetes.io/docs/setup/independent/install-kubeadm/
    - https://kubernetes.io/docs/getting-started-guides/fedora/flannel_multi_node_cluster/
    - https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/
    - https://kubernetes.io/docs/concepts/configuration/assign-pod-node/


## Exercise
> This excercise is not so much about the Kubernetes commands we use to start a deployment, a service and test everything, but more about the configuration required to make the cluster work on multiple machines. A nice picture of the whole networking behind Kubernetes cluster can be found at https://raw.githubusercontent.com/coreos/flannel/master/packet-01.png. You can check the nework interfaces on the master with `ip a`, after the VMs are up.


Create a directory structure similar to this:
```
└── Vagrant
    ├── shared
    └── VMs
        └── centos-k8s
```

In `centos-k8s`, create the following files...

`Vagrantfile`
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

# !!! IMPORTANT NOTE !!!
# !!! Requires vbguest plugin !!!
# vagrant plugin install vagrant-vbguest

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.synced_folder "../../shared", "/mnt/shared", owner: "vagrant", group: "vagrant"
  
  config.vm.provision :shell, path: "k8s_bootstrap_shared.sh"

  config.vm.define "centos-k8s-master.localhost" do |master|
    master.vm.host_name = "centos-k8s-master.localhost"
    master.vm.network :private_network, ip: "192.170.0.201"

    master.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "70"]
    end

    master.vm.provision :shell, path: "k8s_bootstrap_master.sh"
  end

  config.vm.define "centos-k8s-node1.localhost" do |node1|
    node1.vm.host_name = "centos-k8s-node1.localhost"
    node1.vm.network :private_network, ip: "192.170.0.221"
    
    node1.vm.provider "virtualbox" do |v|
      v.memory = 2048
      v.cpus = 2
      v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
    end

    node1.vm.provision :shell, path: "k8s_bootstrap_node1.sh"
  end
end
```

`k8s_bootstrap_shared.sh`
```
#!/bin/bash

# Disable Swap. You MUST disable swap in order for the kubelet to work properly
swapoff -a
sed -r 's/^(.*swap.*)$/# \1/g' -i /etc/fstab

# Vagrant hack: from https://github.com/Yolean/kubeadm-vagrant
sed -i 's/127.*centos-k8s/#\0/' /etc/hosts
printf "192.170.0.201  centos-k8s-master.localhost centos-k8s-master\n" >> /etc/hosts
printf "192.170.0.221  centos-k8s-node1.localhost centos-k8s-node1\n" >> /etc/hosts

# Install Docker
yum install -y docker
systemctl enable docker && systemctl start docker

# Install Kubernetes (kubelet, kubeadm, kubectl)
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF
setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet

# Some users on RHEL/CentOS 7 have reported issues with traffic being routed incorrectly due to iptables being bypassed. You should ensure net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sysctl --system

# Disable the firewall on both the master and node, as Docker does not play well with other firewall rule managers. Please note that iptables.service does not exist on the default Fedora Server install
systemctl mask firewalld.service
systemctl stop firewalld.service

systemctl disable iptables.service || exit 0
systemctl stop iptables.service || exit 0
```


`k8s_bootstrap_master.sh`
```
#!/bin/bash

token_path="/mnt/shared/k8s_token"
ca_cert_hash_path="/mnt/shared/k8s_ca_cert_hash"

# Generate token to be shared between master and nodes
kubeadm token generate > "${token_path}"

# Init Kubeadm 
kubeadm init --token $(cat "${token_path}") --pod-network-cidr=10.244.0.0/16 --apiserver-advertise-address=192.170.0.201

# Generate discovery token ca cert hash
openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //' > "${ca_cert_hash_path}"

# Enable using the cluster as root
mkdir -p $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# Enable using the cluster as vagrant
su vagrant -c 'mkdir -p $HOME/.kube'
su vagrant -c 'sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config'
su vagrant -c 'sudo chown $(id -u):$(id -g) $HOME/.kube/config'

# Flannel
# For flannel to work correctly, --pod-network-cidr=10.244.0.0/16 has to be passed to kubeadm init
curl -o /vagrant/kube-flannel.yml https://raw.githubusercontent.com/coreos/flannel/v0.9.1/Documentation/kube-flannel.yml
# ... use the private interface, https://kubernetes.io/docs/setup/independent/troubleshooting-kubeadm/ - Default NIC When using flannel as the pod network in Vagrant
sed 's#"/opt/bin/flanneld",#"/opt/bin/flanneld", "--iface=eth1",#' -i /vagrant/kube-flannel.yml
kubectl apply -f /vagrant/kube-flannel.yml

# Enable pods scheduling on Master
kubectl taint nodes --all node-role.kubernetes.io/master-
```


`k8s_bootstrap_node1.sh`
```
#!/bin/bash

token_path="/mnt/shared/k8s_token"
ca_cert_hash_path="/mnt/shared/k8s_ca_cert_hash"

# Join Kubernetes Cluster
kubeadm join --token $(cat "${token_path}") 192.170.0.201:6443 --discovery-token-ca-cert-hash sha256:$(cat "${ca_cert_hash_path}")


# Cleanup
rm -rf "${token_path}"
rm -rf "${ca_cert_hash_path}"
```

Finally, you should end up with this:
```
└── Vagrant
    ├── shared
    └── VMs
        └── centos-k8s
            ├── k8s_bootstrap_master.sh
            ├── k8s_bootstrap_node1.sh
            ├── k8s_bootstrap_shared.sh
            └── Vagrantfile

```

Now that we have everything in place, we can start the cluster and test it all works well.
```
# install vagrant virtual box guest addtitions plugin
vagrant plugin install vagrant-vbguest

# start the VMs
vagrant up

# SSH into the master
vagrant ssh centos-k8s-master.localhost

# check the nodes
kubectl get nodes
kubectl describe nodes

# create a test-webserver-deployment definition
cat <<EOF > test-webserver-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: test-webserver-deployment
  labels:
    app: test-webserver
spec:
  replicas: 4
  selector:
    matchLabels:
      app: test-webserver
  template:
    metadata:
      labels:
        app: test-webserver
    spec:
      containers:
      - name: test-webserver
        image: kubernetes/test-webserver:latest
        ports:
        - containerPort: 80
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: beta.kubernetes.io/os
                operator: In
                values:
                - linux
EOF


# start the test-webserver-deployment
kubectl apply --filename test-webserver-deployment.yaml

# check the pods, they should be distributed evenly on both nodes
kubectl get pods -o wide

# create a test-webserver-service definition
cat <<EOF > test-webserver-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: test-webserver-service
  labels:
    name: test-webserver-service
spec:
  type: NodePort
  ports:
    # the port that this service should serve on
    - port: 80
  selector:
    app: test-webserver
EOF

# start the test-webserver-service
kubectl apply --filename test-webserver-service.yaml

# test that the service is rediredirecting the request to all the pods
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname
curl 192.170.0.201:$(kubectl get service test-webserver-service | tail -1 | sed -n -r 's#.* [0-9]+:([0-9]+)/TCP.*#\1#p')/etc/hostname

# cleanup
kubectl delete service test-webserver-service
kubectl delete deployment test-webserver-deployment

# exit the SSH connection
exit

# destroy the VMs
vagrant destroy -f
```
