# Scheduling
Scheduling is a hot topic in distributed systems which deal with efficiently managing services at datecenter level scales. This was pioneered by Google with their Borg system (developed in 2002).

Google's Borg system is a cluster manager that runs hundreds of thousands of jobs, from many thousands of different applications, across a number of clusters each with up to tens of thousands of machines. It achieves high utilization by combining admission control, efficient task-packing, over-commitment, and machine sharing with process-level performance isolation. It supports high-availability applications with runtime features that minimize fault-recovery time, and scheduling policies that reduce the probability of correlated failures.

Inspired from Borg other schedulers were developed most notably Apache Messos and the YARN scheduler for the Hadoop ecosystem. Both of these are open source while Borg is proprietary and only available inside Google. 

## What is a datacenter scheduler

Normal Scheduling: 
![Tetris](https://nickshell1983.files.wordpress.com/2012/03/tetrisgbscore.jpg "Tetris1")

Optimized Scheduling: 
![Tetris_opt]( http://en.gamersrd.com/wp-content/uploads/2016/09/Tetris-Gameboy-GamersRD.png "Tetris2")

A datacenter scheduler's goal is to automate scheduling of work on available resources in way that is optimal and automatic. It provides three main benefits: it (1) hides the details of resource management and failure handling so its users can focus on application development instead; (2) operates with very high reliability and availability, and supports applications that do the same; and (3) lets us run workloads across tens  of  thousands  of  machines  effectively. 

The resources that are scheduled need however to meet certain criteria. Idly they are  statically  linked  to  reduce dependencies on their runtime environment, and structured as packages of binaries and data files, whose installation is orchestrated by the scheduler.

### Why

Studies  indicate  that  datacenter  servers  operate,  most  of  the time,  at  between  10%  and  50%  of  their  maximal  utilization, and that idle/under-utilized servers consume about 50% of their peak power. Increasing utilization by a few percentage points  can  save  millions  of  dollars at scale. So let's do some calculations:

1 server in AWS (m5.2xlarge) with 8vCPU and 32GB of RAM is $0.384 per hour. This will cost us $3363.84 per year.
10 servers like this will cost us $33,638.4 per year.
100 servers like this will cost us $336,384 per year.

If you're running at 50% utilization on average you're loosing $168,192 a year. The goal is to improve utilization thus decreasing the cost(e.g number of servers you pay for).

## Kubernetes

### Overview
   Kubernetes is a platform for automating deployment, scaling, and operations. All of these actions we will group under a single word "scheduling" which encompases all of them. As such we will refer to "Kubernetes" as the "scheduler". The actual objects which are beeing scheduled are "containers".
   Kubernetes takes a lot of ideas from "Borg" and adds some new things to the mix as well. When looking at how this changes the picture of deployment we can draw these conclusions:
* The ~Old Way~ to deploy applications was to install the applications on a host using the operating system package manager. This had the disadvantage of entangling the applications’ executables, configuration, libraries, and lifecycles with each other and with the host OS. One could build immutable virtual-machine images in order to achieve predictable rollouts and rollbacks, but VMs are heavyweight and non-portable (for example our internal environment requires VMDK images for VMWare while our cloud deployment relies on AMI that run on AWS. If we factor in that at some point we might move from AWS the situation becomes even more dire).
* The ~New Way~ is to deploy containers based on operating-system-level virtualization rather than hardware virtualization. These containers are isolated from each other and from the host: they have their own filesystems, they can’t see each others’ processes, and their computational resource usage can be bounded. They are easier to build than VMs, and because they are decoupled from the underlying infrastructure and from the host filesystem, they are portable across clouds and OS distributions.
    Because containers are small and fast, one application can be packed in each container image. This one-to-one application-to-image relationship unlocks the full benefits of containers. With containers, immutable container images can be created at build/release time rather than deployment time, since each application doesn’t need to be composed with the rest of the application stack, nor married to the production infrastructure environment. Generating container images at build/release time enables a consistent environment to be carried from development into production. Similarly, containers are vastly more transparent than VMs, which facilitates monitoring and management. This is especially true when the containers’ process lifecycles are managed by the infrastructure rather than hidden by a process supervisor inside the container. Finally, with a single application per container, managing the containers becomes tantamount to managing deployment of the application. 
    At a minimum, Kubernetes can schedule and run application containers on clusters of physical or virtual machines. However, Kubernetes also allows developers to 'cut the cord' to physical and virtual machines, moving from a host-centric infrastructure to a container-centric infrastructure, which provides the full advantages and benefits inherent to containers. Kubernetes provides the infrastructure to build a truly container-centric development environment.
  
### Containers

For a datacenter scheduler to be successful it needs to operate uniformly over a set of applications. As these application can differ a lot depending on languages, dependencies, runtimes and many more factor ideal we would package them in something "static" that abstracts all this complexity from the scheduler. Enter containers.

#### Namespaces

	Linux has the notion of namespaces which you can think of views over resouces. The purpose of each namespace is to wrap a particular global system resource in an abstraction that makes it appear to the processes within the namespace that they have their own isolated instance of the global resource. By default all processes start in the root namespace, however you can create your own namespaces so that the process can have a different view of the world. The linux kernel supports 6 namespaces right now: 
* Mount namespaces isolate the set of filesystem mount points seen by a group of processes. Thus, processes in different mount namespaces can have different views of the filesystem hierarchy. With the addition of mount namespaces, the mount() and umount() system calls ceased operating on a global set of mount points visible to all processes on the system and instead performed operations that affected just the mount namespace associated with the calling process. One use of mount namespaces is to create environments that are similar to chroot jails. However, by contrast with the use of the chroot() system call, mount namespaces are a more secure and flexible tool for this task. Other more sophisticated uses of mount namespaces are also possible. For example, separate mount namespaces can be set up in a master-slave relationship, so that the mount events are automatically propagated from one namespace to another; this allows, for example, an optical disk device that is mounted in one namespace to automatically appear in other namespaces.
* UTS namespaces isolate two system identifiers—nodename and domainname—returned by the uname() system call; the names are set using the sethostname() and setdomainname() system calls. In the context of containers, the UTS namespaces feature allows each container to have its own hostname and NIS domain name. This can be useful for initialization and configuration scripts that tailor their actions based on these names. The term "UTS" derives from the name of the structure passed to the uname() system call: struct utsname. The name of that structure in turn derives from "UNIX Time-sharing System".
* IPC namespaces isolate certain interprocess communication (IPC) resources, namely, System V IPC objects. The common characteristic of these IPC mechanisms is that IPC objects are identified by mechanisms other than filesystem pathnames. Each IPC namespace has its own set of System V IPC identifiers and its own POSIX message queue filesystem.
* PID namespaces isolate the process ID number space. In other words, processes in different PID namespaces can have the same PID. One of the main benefits of PID namespaces is that containers can be migrated between hosts while keeping the same process IDs for the processes inside the container. PID namespaces also allow each container to have its own init (PID 1), the "ancestor of all processes" that manages various system initialization tasks and reaps orphaned child processes when they terminate. From the point of view of a particular PID namespace instance, a process has two PIDs: the PID inside the namespace, and the PID outside the namespace on the host system. PID namespaces can be nested: a process will have one PID for each of the layers of the hierarchy starting from the PID namespace in which it resides through to the root PID namespace. A process can see (e.g., view via /proc/PID and send signals with kill()) only processes contained in its own PID namespace and the namespaces nested below that PID namespace.
* Network namespaces provide isolation of the system resources associated with networking. Thus, each network namespace has its own network devices, IP addresses, IP routing tables, /proc/net directory, port numbers, and so on. Network namespaces make containers useful from a networking perspective: each container can have its own (virtual) network device and its own applications that bind to the per-namespace port number space; suitable routing rules in the host system can direct network packets to the network device associated with a specific container. Thus, for example, it is possible to have multiple containerized web servers on the same host system, with each server bound to port 80 in its (per-container) network namespace.
* User namespaces isolate the user and group ID number spaces. In other words, a process's user and group IDs can be different inside and outside a user namespace. The most interesting case here is that a process can have a normal unprivileged user ID outside a user namespace while at the same time having a user ID of 0 inside the namespace. This means that the process has full root privileges for operations inside the user namespace, but is unprivileged for operations outside the namespace.

#### CGroups

Cgroups allow you to allocate resources — such as CPU time, system memory, network bandwidth, or combinations of these resources — among user-defined groups of tasks (processes) running on a system. You can monitor the cgroups you configure, deny cgroups access to certain resources, and even reconfigure your cgroups dynamically on a running system. 
**How Control Groups Are Organized**
    Cgroups are organized hierarchically, like processes, and child cgroups inherit some of the attributes of their parents. However, there are differences between the two models.
    **The Linux Process Model**
    All processes on a Linux system are child processes of a common parent: the init process, which is executed by the kernel at boot time and starts other processes (which may in turn start child processes of their own). Because all processes descend from a single parent, the Linux process model is a single hierarchy, or tree.
Additionally, every Linux process except init inherits the environment (such as the PATH variable) and certain other attributes (such as open file descriptors) of its parent process.
    **The Cgroup Model**
    Cgroups are similar to processes in that:

-   they are hierarchical, and
-   child cgroups inherit certain attributes from their parent cgroup.

    The fundamental difference is that many different hierarchies of cgroups can exist simultaneously on a system. If the Linux process model is a single tree of processes, then the cgroup model is one or more separate, unconnected trees of tasks (i.e. processes). Multiple separate hierarchies of cgroups are necessary because each hierarchy is attached to one or more subsystems
The following cgroups are available on a RHEL7 server: 

-   blkio — this subsystem sets limits on input/output access to and from block devices such as physical drives (disk, solid state, or USB).
-   cpu — this subsystem uses the scheduler to provide cgroup tasks access to the CPU.
-   cpuacct — this subsystem generates automatic reports on CPU resources used by tasks in a cgroup.
-   cpuset — this subsystem assigns individual CPUs (on a multicore system) and memory nodes to tasks in a cgroup.
-   devices — this subsystem allows or denies access to devices by tasks in a cgroup.
-   freezer — this subsystem suspends or resumes tasks in a cgroup.
-   memory — this subsystem sets limits on memory use by tasks in a cgroup and generates automatic reports on memory resources used by those tasks.
-   net\_cls — this subsystem tags network packets with a class identifier (classid) that allows the Linux traffic controller (tc) to identify packets originating from a particular cgroup task.
-   net\_prio — this subsystem provides a way to dynamically set the priority of network traffic per network interface.
-   ns — the namespace subsystem.

By default all OS components start in the **root cgroup** which is unrestricted (doesn't have any cgroup limitations defined on it). The `systemd-cgtop` tool shows each group (you will see them a slices) and the resouces each group uses: 

    Path                                                                                                         Tasks   %CPU   Memory  Input/s Output/s
    
    /                                                                                                              347   19.6     7.0G        -        -
    /system.slice                                                                                                   82   13.3     2.6G        -        -
    /system.slice/kubelet.service                                                                                    2   11.5    53.6M        -        -
    /kubepods.slice                                                                                                  -    4.2   741.2M        -        -
    /kubepods.slice/kubepods-burstable.slice                                                                         -    3.5   346.6M        -        -
    /kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod1a4337e331349a4034ee871801eb6c7f.slice            -    2.2    63.1M        -        -
    /user.slice                                                                                                      8    2.0   601.5M        -        -
    /kubepods.slice/kubepods-burstable.slice/kubepods-burstable-pod1794e279e23483f245a1bc8ae4eb2f9b.slice            -    1.1   177.4M        -        -
    /system.slice/kafka.service                                                                                      1    0.8   830.1M        -        -
    /kubepods.slice/kubepods-besteffort.slice                                                                        -    0.6   394.5M        -        -
    /system.slice/docker.service                                                                                    21    0.5   150.4M        -        -
    /kubepods.slice/kubepods-besteffort.slice/kubepods-besteffort-podba6d8f264fe61854f46e7eacbd626a92.slice          -    0.5   300.5M        -        -
    /kubepods.slice/kubepods-burstable.slice/kubepods-burstable-poda0f6609c_917d_11e7_9439_02ecb0828e42.slice        -    0.2    72.3M        -        -
    /kubepods.slice/kubepods-besteffor...ice/kubepods-besteffort-poda0ef9a35_917d_11e7_9439_02ecb0828e42.slice       -    0.1    34.7M        -        -
    /system.slice/ocm.service                                                                                        2    0.1   286.5M        -        -
    /system.slice/zookeeper.service                                                                                  1    0.1   165.4M        -        -
    /system.slice/goferd.service                                                                                     1    0.1    27.2M        -        -

-   the *system.slice* refers to the route group.
-   *kubepods.slice* refers to the kubernets groups. Inside them there will be individual groups for each pod and container in the pod.

To identify the container/pod you want to check the groups status information for you will first need to get the docker container ID from kubernetes using `kubectl -n ns-org-1 describe pod <POD_NAME>`. This will also tell you the node on which the pod is currently scheduled so you can login there to get more info. Stats for the entire pod are available usualy under kubepods slice on the kubernetes node. To get the actual UID of the slice run the command `systemd-cgls` which provides a list of all cgroups running on the machine.

    root@k00003-b0ie ~]# systemd-cgls
    ├─1 /usr/lib/systemd/systemd --switched-root --system --deserialize 22
    ├─kubepods.slice
    │ ├─kubepods-podad7ab520_9ed8_11e7_979f_02ecb0828e42.slice
    │ │ ├─docker-4d66e74bd8ff3f538102dab400028f7162120b075ab29633df5762797c56c12e.scope
    │ │ │ ├─ 3647 sleep 3m
    │ │ │ ├─17650 /bin/sh -c /helpers/init.sh
    │ │ │ ├─17667 /bin/sh /helpers/init.sh
    │ │ │ ├─17675 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent -qq --use-v1-config --suppress-repeated-stacktrace
    │ │ │ └─17697 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent -qq --use-v1-config --suppress-repeated-stacktrace
    │ │ ├─docker-a60d3c296f60af17f289a855a8412b6a98cb43cd625a1b39d33a8cfc7e3de207.scope
    │ │ │ ├─ 3889 sleep 1m
    │ │ │ ├─17390 /bin/sh /tools/init.sh
    │ │ │ ├─17474 /bin/sh /opt/optymyze/loginserver/run-loginserver.sh -Xms384m -Xmx384m -XX:+HeapDumpOnOutOfMemoryError -javaagent:/tools/jmx_prometh
    │ │ │ └─17481 java -cp lib/* -Xms384m -Xmx384m -XX:+HeapDumpOnOutOfMemoryError -javaagent:/tools/jmx_prometheus_javaagent-0.9.jar=28695:/tools/prome
    │ │ └─docker-4a81002b70461359ad94d88cae7ed669402a6de6d164a479d3f5c32b8ad11150.scope
    │ │   └─16182 /pause
    │ ├─kubepods-podacd4d7e2_9ed8_11e7_979f_02ecb0828e42.slice
    │ │ ├─docker-998e9560a6ce1a0f4fe468d8448187060960583881414c4f370df6799b296d41.scope
    │ │ │ ├─ 3783 sh
    │ │ │ ├─ 3890 sleep 1m
    │ │ │ ├─ 3974 su - optymyze
    │ │ │ ├─ 3975 -sh
    │ │ │ ├─ 4323 java -cp ../lib/webworkerdispatcher/*:../lib/webdispatcher/*:../lib/workerdispatcher/*:../lib/dispatcher/* -Xms512m -Xmx512m -javaagen
    │ │ │ ├─ 8048 sh
    │ │ │ ├─ 9385 sudo su - optymyze
    │ │ │ ├─ 9386 su - optymyze
    │ │ │ ├─ 9387 -sh
    │ │ │ ├─30466 /bin/sh /tools/init.sh
    │ │ │ └─30520 /opt/farmount/farmount
    │ │ ├─docker-23f0d3a5a778145d9bd8df80de93872fa73f9d0c4be95c3f89c5285c3f089216.scope
    │ │ │ ├─ 3646 sleep 3m
    │ │ │ ├─17547 /bin/sh -c /helpers/init.sh
    │ │ │ ├─17595 /bin/sh /helpers/init.sh
    │ │ │ ├─17604 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent -qq --use-v1-config --suppress-repeated-stacktrace
    │ │ │ └─17693 /opt/td-agent/embedded/bin/ruby /usr/sbin/td-agent -qq --use-v1-config --suppress-repeated-stacktrace
    │ │ └─docker-12cfe4aab82d5e7f21cb54f326998a7f7f60361a696d0c25623b7eca83cbba01.scope
    │ │   └─15814 /pause
    │ ├─kubepods-besteffort.slice
    │ │ ├─kubepods-besteffort-podc2d0786e_9182_11e7_979f_02ecb0828e42.slice
    │ │ │ ├─docker-c055f821b65a0f2941c71605f03eae6157b373fb2010065009ca2ce1dd8f4f43.scope
    │ │ │ │ └─26174 /usr/local/bin/kube-proxy --kubeconfig=/var/lib/kube-proxy/kubeconfig.conf --cluster-cidr=10.255.128.0/17
    │ │ │ └─docker-86cc1bb4fd61e76c59613d94d5a57978192eed9797f20f5c1cc6142ba49b3213.scope
    │ │ │   └─25686 /pause
    │ │ └─kubepods-besteffort-podc2d06ff1_9182_11e7_979f_02ecb0828e42.slice
    │ │   ├─docker-6ee4f6b85436536910acafe27745af2c59af5f5596d66c6afd41fb13fe603425.scope
    │ │   │ ├─ 6689 /bin/sh -c set -e -x; cp -f /etc/kube-flannel/cni-conf.json /etc/cni/net.d/10-flannel.conf; while true; do sleep 3600; done
    │ │   │ └─29459 sleep 3600
    │ │   ├─docker-4e6300b6e0ffad097742117f504c101008768e68e3b35e67aee090a6bfd4d07b.scope
    │ │   │ └─6596 /opt/bin/flanneld --ip-masq --kube-subnet-mgr
    │ │   └─docker-05c4dc203857b25b83a3beda3f25074e69fb34757f88e030e9f2a71146695c3b.scope
    │ │     └─25990 /pause

In our case to see the login server pod details we ca see the kube pod slice to be **kubepods-podad7ab520\_9ed8\_11e7\_979f\_02ecb0828e42.slice** while the container id is **docker-a60d3c296f60af17f289a855a8412b6a98cb43cd625a1b39d33a8cfc7e3de207.scope**
To get for example the memory stats of the **docker-a60d3c296f60af17f289a855a8412b6a98cb43cd625a1b39d33a8cfc7e3de207.scope** container we will need to cat the file `/sys/fs/cgroup/memory/kubepods.slice/kubepods-podacd4d7e2_9ed8_11e7_979f_02ecb0828e42.slice/docker-998e9560a6ce1a0f4fe468d8448187060960583881414c4f370df6799b296d41.scope/memory.stat`

    [root@k00003-b0ie ~]# cat /sys/fs/cgroup/memory/kubepods.slice/kubepods-podacd4d7e2_9ed8_11e7_979f_02ecb0828e42.slice/docker-998e9560a6ce1a0f4fe468d8448187060960583881414c4f370df6799b296d41.scope/memory.stat
    cache 137879552
    rss 887947264
    rss_huge 773849088
    mapped_file 18018304
    swap 1064960
    pgpgin 15134965
    pgpgout 15121112
    pgfault 66788875
    pgmajfault 721
    inactive_anon 480665600
    active_anon 407281664
    inactive_file 51650560
    active_file 86228992
    unevictable 0
    hierarchical_memory_limit 1073741824
    hierarchical_memsw_limit 2147483648
    total_cache 137879552
    total_rss 887947264
    total_rss_huge 773849088
    total_mapped_file 18018304
    total_swap 1064960
    total_pgpgin 15134965
    total_pgpgout 15121112
    total_pgfault 66788875
    total_pgmajfault 721
    total_inactive_anon 480665600
    total_active_anon 407281664
    total_inactive_file 51650560
    total_active_file 86228992
    total_unevictable 0

For us two groups are very important and we will continue to focus on them. One is the memory group and the cpu cgroups.

1.  Memory cgroup

    The memory subsystem generates automatic reports on memory resources used by the tasks in a cgroup, and sets limits on memory use of those tasks. The information about the memory group are available on the machine at *sys/fs/cgroup/memory* location:
    
        [root@k00001-b0ie ~]# ls -lp /sys/fs/cgroup/memory/
        total 0
        -rw-r--r--   1 root root 0 Sep  1 14:55 cgroup.clone_children
        --w--w--w-   1 root root 0 Sep  1 14:55 cgroup.event_control
        -rw-r--r--   1 root root 0 Sep  1 14:55 cgroup.procs
        -r--r--r--   1 root root 0 Sep  1 14:55 cgroup.sane_behavior
        drwxr-xr-x   4 root root 0 Sep  4 12:37 kubepods.slice/
        drwxr-xr-x   2 root root 0 Sep  4 12:37 machine.slice/
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.failcnt
        --w-------   1 root root 0 Sep  1 14:55 memory.force_empty
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.kmem.failcnt
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.kmem.limit_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.kmem.max_usage_in_bytes
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.kmem.slabinfo
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.kmem.tcp.failcnt
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.kmem.tcp.limit_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.kmem.tcp.max_usage_in_bytes
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.kmem.tcp.usage_in_bytes
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.kmem.usage_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.limit_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.max_usage_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.memsw.failcnt
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.memsw.limit_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.memsw.max_usage_in_bytes
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.memsw.usage_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.move_charge_at_immigrate
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.numa_stat
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.oom_control
        ----------   1 root root 0 Sep  1 14:55 memory.pressure_level
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.soft_limit_in_bytes
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.stat
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.swappiness
        -r--r--r--   1 root root 0 Sep  1 14:55 memory.usage_in_bytes
        -rw-r--r--   1 root root 0 Sep  1 14:55 memory.use_hierarchy
        -rw-r--r--   1 root root 0 Sep  1 14:55 notify_on_release
        -rw-r--r--   1 root root 0 Sep  1 14:55 release_agent
        drwxr-xr-x 106 root root 0 Oct  2 12:00 system.slice/
        -rw-r--r--   1 root root 0 Sep  1 14:55 tasks
        drwxr-xr-x   2 root root 0 Sep  4 12:14 user.slice/
    
    What stands out for us at this location is the `user.slice` (user processes), `system.slice` (system process) and `kubepods.slice` (kubernetes pods).
    You can use the kubepods.slice to drill down in terms of memory usage to the entire slice (all pods running), per pod, or per container inside the pod. Each of them will present you the same hierarchy. The following files are of importance for understanding the memory usage of the application inside the respective memory group:
    
    -   memory.stat. This file reports a wide range of memory statistics, as described in the following table:
        
        <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
        
        
        <colgroup>
        <col  class="org-left" />
        
        <col  class="org-left" />
        </colgroup>
        <thead>
        <tr>
        <th scope="col" class="org-left">Statistic</th>
        <th scope="col" class="org-left">Description</th>
        </tr>
        </thead>
        
        <tbody>
        <tr>
        <td class="org-left">cache</td>
        <td class="org-left">page cache, including tmpfs (shmem), in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">rss</td>
        <td class="org-left">anonymous and swap cache, not including tmpfs (shmem), in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">mapped\_file</td>
        <td class="org-left">size of memory-mapped mapped files, including tmpfs (shmem), in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">pgpgin</td>
        <td class="org-left">number of pages paged into memory</td>
        </tr>
        
        
        <tr>
        <td class="org-left">pgpgout</td>
        <td class="org-left">number of pages paged out of memory</td>
        </tr>
        
        
        <tr>
        <td class="org-left">swap</td>
        <td class="org-left">swap usage, in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">active\_anon</td>
        <td class="org-left">anonymous and swap cache on active least-recently-used (LRU) list, including tmpfs (shmem)</td>
        </tr>
        
        
        <tr>
        <td class="org-left">inactive\_anon</td>
        <td class="org-left">anonymous and swap cache on inactive LRU list, including tmpfs (shmem), in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">active\_file</td>
        <td class="org-left">file-backed memory on active LRU list, in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">inactive\_file</td>
        <td class="org-left">file-backed memory on inactive LRU list, in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">unevictable</td>
        <td class="org-left">memory that cannot be reclaimed, in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">hierarchical\_memory\_limit</td>
        <td class="org-left">memory limit for the hierarchy that contains the memory cgroup, in bytes</td>
        </tr>
        
        
        <tr>
        <td class="org-left">hierarchical\_memsw\_limit</td>
        <td class="org-left">memory plus swap limit for the hierarchy that contains the memory cgroup, in bytes</td>
        </tr>
        </tbody>
        </table>
        
        Additionally, each of these files other than hierarchical\_memory\_limit and hierarchical\_memsw\_limit has a counterpart prefixed total\_ that reports not only on the cgroup, but on all its children as well. For example, swap reports the swap usage by a cgroup and total\_swap reports the total swap usage by the cgroup and all its child groups.
          When you interpret the values reported by memory.stat, note how the various statistics inter-relate:
        
        -   active\_anon + inactive\_anon = anonymous memory + file cache for tmpfs + swap cache
        
        Therefore, active\_anon + inactive\_anon ≠ rss, because rss does not include tmpfs.
        
        -   active\_file + inactive\_file = cache - size of tmpfs
    -   memory.usage\_in\_bytes. This file reports the total current memory usage by processes in the cgroup (in bytes).
    -   memory.memsw.usage\_in\_bytes. This file reports the sum of current memory usage plus swap space used by processes in the cgroup (in bytes).
    -   memory.max\_usage\_in\_bytes. This file reports the maximum memory used by processes in the cgroup (in bytes).
    -   memory.memsw.max\_usage\_in\_bytes. This file reports the maximum amount of memory and swap space used by processes in the cgroup (in bytes).
    -   memory.limit\_in\_bytes. This file sets the maximum amount of user memory (including file cache). If no units are specified, the value is interpreted as bytes. However, it is possible to use suffixes to represent larger units — k or K for kilobytes, m or M for megabytes, and g or G for gigabytes.
    -   memory.memsw.limit\_in\_bytes. This file sets the maximum amount for the sum of memory and swap usage. If no units are specified, the value is interpreted as bytes.
    -   memory.failcnt. This file reports the number of times that the memory limit has reached the value set in memory.limit\_in\_bytes.
    -   memory.memsw.failcnt. This file reports the number of times that the memory plus swap space limit has reached the value set in memory.memsw.limit\_in\_bytes.
    -   memory.swappiness. This file sets the tendency of the kernel to swap out process memory used by tasks in this cgroup instead of reclaiming pages from the page cache. This is the same tendency, calculated the same way, as set in /proc/sys/vm/swappiness for the system as a whole. The default value is 60. Values lower than 60 decrease the kernel's tendency to swap out process memory, values greater than 60 increase the kernel's tendency to swap out process memory, and values greater than 100 permit the kernel to swap out pages that are part of the address space of the processes in this cgroup.Note that a value of 0 does not prevent process memory being swapped out; swap out might still happen when there is a shortage of system memory because the global virtual memory management logic does not read the cgroup value. To lock pages completely, use mlock() instead of cgroups.
    -   memory.oom\_control. This file contains a flag (0 or 1) that enables or disables the Out of Memory killer for a cgroup. If enabled (0), tasks that attempt to consume more memory than they are allowed are immediately killed by the OOM killer. The OOM killer is enabled by default in every cgroup using the memory subsystem; to disable it, write 1.
        
            ~]# echo 1 > /cgroup/memory/lab1/memory.oom_control

2.  CPU cgroup

    The cpu subsystem schedules CPU access to cgroups. Access to CPU resources can be scheduled using two schedulers:
    
    -   Completely Fair Scheduler (CFS) — a proportional share scheduler which divides the CPU time (CPU bandwidth) proportionately between groups of tasks (cgroups) depending on the priority/weight of the task or shares assigned to cgroups. This is the **default** scheduler used by Docker on Linux (and Kubernetes)
    -   Real-Time scheduler (RT) — a task scheduler that provides a way to specify the amount of CPU time that real-time tasks can use.
    
    In CFS, a cgroup can get more than its share of CPU if there are enough idle CPU cycles available in the system, due to the work conserving nature of the scheduler. This is usually the case for cgroups that consume CPU time based on relative shares. Ceiling enforcement can be used for cases when a hard limit on the amount of CPU that a cgroup can utilize is required (that is, tasks cannot use more than a set amount of CPU time).
    The following options can be used to configure ceiling enforcement or relative sharing of CPU:
    **Ceiling Enforcement Tunable Parameters**
    
    -   cpu.cfs\_period\_us. Specifies a period of time in microseconds (µs, represented here as "us") for how regularly a cgroup's access to CPU resources should be reallocated. If tasks in a cgroup should be able to access a single CPU for 0.2 seconds out of every 1 second, set cpu.cfs\_quota\_us to 200000 and cpu.cfs\_period\_us to 1000000. The upper limit of the cpu.cfs\_quota\_us parameter is 1 second and the lower limit is 1000 microseconds.
    -   cpu.cfs\_quota\_us. Specifies the total amount of time in microseconds (µs, represented here as "us") for which all tasks in a cgroup can run during one period (as defined by cpu.cfs\_period\_us). As soon as tasks in a cgroup use up all the time specified by the quota, they are throttled for the remainder of the time specified by the period and not allowed to run until the next period. If tasks in a cgroup should be able to access a single CPU for 0.2 seconds out of every 1 second, set cpu.cfs\_quota\_us to 200000 and cpu.cfs\_period\_us to 1000000. Note that the quota and period parameters operate on a CPU basis. To allow a process to fully utilize two CPUs, for example, set cpu.cfs\_quota\_us to 200000 and cpu.cfs\_period\_us to 100000.Setting the value in cpu.cfs\_quota\_us to -1 indicates that the cgroup does not adhere to any CPU time restrictions. This is also the default value for every cgroup (except the root cgroup).
    -   cpu.stat. Reports CPU time statistics using the following values:
        -   nr\_periods — number of period intervals (as specified in cpu.cfs\_period\_us) that have elapsed.
        -   nr\_throttled — number of times tasks in a cgroup have been throttled (that is, not allowed to run because they have exhausted all of the available time as specified by their quota).
        -   throttled\_time — the total time duration (in nanoseconds) for which tasks in a cgroup have been throttled.
    
    **Relative Shares Tunable Parameters** 
    
    -   cpu.shares. Contains an integer value that specifies a relative share of CPU time available to the tasks in a cgroup. For example, tasks in two cgroups that have cpu.shares set to 100 will receive equal CPU time, but tasks in a cgroup that has cpu.shares set to 200 receive twice the CPU time of tasks in a cgroup where cpu.shares is set to 100. The value specified in the cpu.shares file must be 2 or higher.
    
    Note that shares of CPU time are distributed per all CPU cores on multi-core systems. Even if a cgroup is limited to less than 100% of CPU on a multi-core system, it may use 100% of each individual CPU core. Consider the following example: if cgroup A is configured to use 25% and cgroup B 75% of the CPU, starting four CPU-intensive processes (one in A and three in B) on a system with four cores results in the following division of CPU shares:
    
    <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
    
    
    <colgroup>
    <col  class="org-right" />
    
    <col  class="org-left" />
    
    <col  class="org-right" />
    
    <col  class="org-left" />
    </colgroup>
    <thead>
    <tr>
    <th scope="col" class="org-right">PID</th>
    <th scope="col" class="org-left">cgroup</th>
    <th scope="col" class="org-right">CPU</th>
    <th scope="col" class="org-left">CPU share</th>
    </tr>
    </thead>
    
    <tbody>
    <tr>
    <td class="org-right">100</td>
    <td class="org-left">A</td>
    <td class="org-right">0</td>
    <td class="org-left">100% of CPU0</td>
    </tr>
    
    
    <tr>
    <td class="org-right">101</td>
    <td class="org-left">B</td>
    <td class="org-right">1</td>
    <td class="org-left">100% of CPU1</td>
    </tr>
    
    
    <tr>
    <td class="org-right">102</td>
    <td class="org-left">B</td>
    <td class="org-right">2</td>
    <td class="org-left">00% of CPU2</td>
    </tr>
    
    
    <tr>
    <td class="org-right">103</td>
    <td class="org-left">B</td>
    <td class="org-right">3</td>
    <td class="org-left">100% of CPU3</td>
    </tr>
    </tbody>
    </table>
    
    Using relative shares to specify CPU access has two implications on resource management that should be considered:
    
    -   Because the CFS does not demand equal usage of CPU, it is hard to predict how much CPU time a cgroup will be allowed to utilize. When tasks in one cgroup are idle and are not using any CPU time, the leftover time is collected in a global pool of unused CPU cycles. Other cgroups are allowed to borrow CPU cycles from this pool.
    -   The actual amount of CPU time that is available to a cgroup can vary depending on the number of cgroups that exist on the system. If a cgroup has a relative share of 1000 and two other cgroups have a relative share of 500, the first cgroup receives 50% of all CPU time in cases when processes in all cgroups attempt to use 100% of the CPU. However, if another cgroup is added with a relative share of 1000, the first cgroup is only allowed 33% of the CPU (the rest of the cgroups receive 16.5%, 16.5%, and 33% of CPU).
    
    Another important part of the cgroup CPU scheduling is the accounting that linux keeps. This helps us determine if the allocated shares are sufficient, if the CFS is actively throttling the application and so on. This information can be found in the `/sys/fs/cpuacct` section.
    **CPU Accounting(cpuacct)**
    The CPU cgroup will also provide additional files under the prefix "cpuacct".
    Those files provide accounting statistics and were previously provided by the separate cpuacct controller. Although the cpuacct controller will still be kept around for compatibility reasons, its usage is discouraged. If both the CPU and cpuacct controllers are present in the system, distributors are encouraged to always mount them together.
    The files where this information is available are under the `/sys/fs/cgroup/cpuacct` location. These files are:
    
    -   **cpu.shares**: The weight of each group living in the same hierarchy, that translates into the amount of CPU it is expected to get. Upon cgroup creation, each group gets assigned a default of 1024. The percentage of CPU assigned to the cgroup is the value of shares divided by the sum of all shares in all cgroups in the same level.
    -   **cpu.cfs\_period\_us**: The duration in microseconds of each scheduler period, for bandwidth decisions. This defaults to 100000us or 100ms. Larger periods will improve throughput at the expense of latency, since the scheduler will be able to sustain a cpu-bound workload for longer. The opposite of true for smaller periods. Note that this only affects non-RT tasks that are scheduled by the CFS scheduler.
    -   **cpu.cfs\_quota\_us**: The maximum time in microseconds during each cfs\_period\_us in for the current group will be allowed to run. For instance, if it is set to half of cpu\_period\_us, the cgroup will only be able to peak run for 50 % of the time. One should note that this represents aggregate time over all CPUs in the system. Therefore, in order to allow full usage of two CPUs, for instance, one should set this value to twice the value of cfs\_period\_us.
    -   **cpu.stat**: statistics about the bandwidth controls. No data will be presented if cpu.cfs\_quota\_us is not set. The file presents three numbers:
        -   nr\_periods: how many full periods have been elapsed.
        -   nr\_throttled: number of times we exausted the full allowed bandwidth
        -   throttled\_time: total time the tasks were not run due to being overquota
    -   **cpu.rt\_runtime\_us and cpu.rt\_period\_us**: Those files are the RT-tasks analogous to the CFS files cfs\_quota\_us and cfs\_period\_us. One important difference, though, is that while the cfs quotas are upper bounds that won't necessarily be met, the rt runtimes form a stricter guarantee. Therefore, no overlap is allowed. Implications of that are that given a hierarchy with multiple children, the sum of all rt\_runtime\_us may not exceed the runtime of the parent. Also, a rt\_runtime\_us of 0, means that no rt tasks can ever be run in this cgroup. For more information about rt tasks runtime assignments, see scheduler/sched-rt-group.txt
    -   **cpu.stat\_percpu**: Various scheduler statistics for the current group. The information provided in this file is akin to the one displayed in /proc/stat, except for the fact that it is cgroup-aware. The file format consists of a one-line header that describes the fields being listed.  No guarantee is given that the fields will be kept the same between kernel releases, and readers should always check the header in order to introspect it.
    -   **cpuacct.usage**: The aggregate CPU time, in nanoseconds, consumed by all tasks in this group.
    -   **cpuacct.usage\_percpu**: The CPU time, in nanoseconds, consumed by all tasks in this group, separated by CPU. The format is an space-separated array of time values, one for each present CPU.
    -   **cpuacct.stat**: aggregate user and system time consumed by tasks in this group.
        The format is
        -   user: x
        -   system: y


<a id="org0a9434b"></a>

#### Container security

1.  Intro

    With Docker people are using the layered security approach, which is "the practice of combining multiple mitigating security controls to protect resources and data."
    
    Basically, docker adds as many security barriers as possible to prevent a break out. If a privileged process can break out of one containment mechanism, we want to block them with the next. Docker takes advantage of as many security mechanisms of Linux as possible to make the system more secure.
    These include filesystem protections, Linux Capabilities, Namespaces, cgroups and SELinux:
    
    -   File System protections:
        -   Read-only mount points
            Some Linux kernel file systems have to be mounted in a container environment or processes would fail to run. Fortunately, most of these filesystems can be mounted as "read-only".  Most apps should never need to write to these file systems.
            Docker mounts these file systems into the container as "read-only" mount points.
            
                . /sys 
                . /proc/sys 
                . /proc/sysrq-trigger 
                . /proc/irq 
                . /proc/bus
            
            By mounting these file systems as read-only, privileged container processes cannot write to them. They cannot effect the host system. Of course, we also block the ability of the privileged container processes from remounting the file systems as read/write. We block the ability to mount any file systems at all within the container. I will explain how we block mounts when we get to capabilities.
        -   Copy-on-write file systems
            Docker uses copy-on-write file systems. This means containers can use the same file system image as the base for the container. When a container writes content to the image, it gets written to a container specific file system. This prevents one container from seeing the changes of another container even if they wrote to the same file system image. Just as important, one container can not change the image content to effect the processes in another container.
    -   Capabilities
        Linux capabilities are explained well on their main page:
        
        ---
        
        For the purpose of performing permission checks, traditional UNIX implementations distinguish two categories of processes: privileged processes (whose effective user ID is 0, referred to as superuser or root), and unprivileged processes (whose effective UID is nonzero). Privileged processes bypass all kernel permission checks, while unprivileged processes are subject to full permission checking based on the process's credentials (usually: effective UID, effective GID, and supplementary group list). Starting with kernel 2.2, Linux divides the privileges traditionally associated with superuser into distinct units, known as capabilities, which can be independently enabled and disabled. Capabilities are a per-thread attribute.
        
        ---
        
        Removing capabilities can cause applications to break, which means we have a balancing act in Docker between functionality, usability and security. Here is the current list of capabilities that Docker uses: chown, dac\_override, fowner, kill, setgid, setuid, setpcap, net\_bind\_service, net\_raw, sys\_chroot, mknod, setfcap, and audit\_write.
        
        It is continuously argued back and forth which capabilities should be allowed or denied by default. Docker allows customers to manipulate default list with the command line options for Docker run.
        
        -   Capabilities removed
            -   CAP\_SETPCAP - Modify process capabilities
            -   CAP\_SYS\_MODULE - Insert/Remove kernel modules
            -   CAP\_SYS\_RAWIO - Modify Kernel Memory
            -   CAP\_SYS\_PACCT - Configure process accounting
            -   CAP\_SYS\_NICE - Modify Priority of processes
            -   CAP\_SYS\_RESOURCE - Override Resource Limits
            -   CAP\_SYS\_TIME - Modify the system clock
            -   CAP\_SYS\_TTY\_CONFIG - Configure tty devices
            -   CAP\_AUDIT\_WRITE -	Write the audit log
            -   CAP\_AUDIT\_CONTROL - Configure Audit Subsystem
            -   CAP\_MAC\_OVERRIDE - Ignore Kernel MAC Policy
            -   CAP\_MAC\_ADMIN - Configure MAC Configuration
            -   CAP\_SYSLOG - Modify Kernel printk behavior
            -   CAP\_NET\_ADMIN - Configure the network
            -   CAP\_SYS\_ADMIN - Catch all
        
        Lets look closer at the last couple in the table. By removing CAP\_NET\_ADMIN for a container, the container processes cannot modify the systems network, meaning assigning IP addresses to network devices, setting up routing rules, modifying iptables.
        
        All networking is setup by the Docker daemon before the container starts. You can manage the containers network interface from outside the container but not inside.
        
        **CAP\_SYS\_ADMIN** is special capability. I believe it is the kernel catchall capability. When kernel engineers design new features into the kernel, they are supposed to pick the capability that best matches what the feature allows. Or, they were supposed to create a new capability. Problem is, there were originally only 32 capability slots available. When in doubt the kernel engineer would just fall back to using CAP\_SYS\_ADMIN. Here is the list of things CAP\_SYS\_ADMIN allows according to /usr/include/linux/capability.
        The two most important features that removing CAP\_SYS\_ADMIN from containers does is stops processes from executing the mount syscall or modifying namespaces. You don't want to allow your container processes to mount random file systems or to remount the read-only file systems. We however require these for Optymyze so we run with this capability enabled.

2.  SELinux

    SELinux is a LABELING system where you can define interactions between labels. It also has some nice properties like:
    
    -   Every process has a LABEL
    -   Every file, directory, and system object has a LABEL
    -   Policy rules control access between labeled processes and labeled objects
    -   The kernel enforces the rules
    
    The Docker SELinux security policy is similar to the libvirt security policy and is based on the libvirt security policy.The libvirt security policy is a series of SELinux policies that defines two ways of isolating virtual machines. Generally, virtual machines are prevented from accessing parts of the network. Specifically, individual virtual machines are denied access to one another’s resources. Red Hat extends the libvirt-SELinux model to Docker. The Docker SELinux role and Docker SELinux types are based on libvirt.
    SELinux implements a Mandatory Access Control system. This means the owners of an object have no control or discretion over the access to an object. The kernel enforces Mandatory Access Controls. An explanation on how SELinux enforcement works are available in the [visual guide to SELinux policy enforcement](https://opensource.com/business/13/11/selinux-policy-guide) (and subsequent, [SELinux Coloring Book](http://people.redhat.com/duffy/selinux/selinux-coloring-book_A4-Stapled.pdf)).
    The default type used for running Docker containers is **svirt\_lxc\_net\_t**. All container processes run with this type.
    All content within the container is labeled with the **svirt\_sandbox\_file\_t** type.
    **svirt\_lxc\_net\_t** is allowed to manage any content labeled with **svirt\_sandbox\_file\_t**.
    **svirt\_lxc\_net\_t** is also able to read/execute most labels under /usr on the host.
    
    Processes running witht he **svirt\_lxc\_net\_t** are not allowed to open/write to any other labels on the system. It is not allowed to read any default labels in /var, /root, /home etc.
    
        Basically, we want to allow the processes to read/execute system content, but we want to not allow it to use any "data" on the system unless it is in the container, by default.
        **Problem**
        If all container processes are run with svirt\_lxc\_net\_t and all content is labeled with svirt\_sandbox\_file\_t, wouldn't container processes be allowed to attack processes running in other containers and content owned by other containers? 
    This is where Multi Category Security enforcement comes in, described below.
        **Alternate Types**
        Notice that Docker used "net" in the type label. This is used to indicate that this type can use full networking.Then the processes inside of the container would not be allowed to use any network ports. Similarly, we could easily write an Apache policy that would only allow the container to listen on Apache ports, but not allowed to connect out on any ports. Using this type of policy you could prevent your container from becoming a spam bot even if it was cracked, and the hacker got control of the apache process within the container.
        **Multi Category Security enforcement**
        Multi Category Security is based on Multi Level Security (MLS). MCS takes advantage of the last component of the SELinux label the MLS Field. MCS enforcement protects containers from each other. When containers are launched the Docker daemon picks a random MCS label, for example s0:c1,c2, to assign to the container. The Docker daemon labels all of the content in the container with this MCS label. When the daemon launches the container process it tells the kernel to label the processes with the same MCS label. The kernel only allows container processes to read/write their own content as long as the process MCS label matches the filesystem content MCS label. The kernel blocks container processes from read/writing content labeled with a different MCS label.
    
    The full description of the SELinux policy docker uses is available at <https://www.mankier.com/8/docker_selinux>.

3.  SecCompare

    Secure Computing Mode (seccomp) is a kernel feature that allows you to filter system calls to the kernel from a container. The combination of restricted and allowed calls are arranged in profiles, and you can pass different profiles to different containers. Seccomp provides more fine-grained control than capabilities, giving an attacker a limited number of syscalls from the container.
    The default seccomp profile for docker is a JSON file and can be viewed here: <https://github.com/docker/docker/blob/master/profiles/seccomp/default.json>. It blocks 44 system calls out of more than 300 available.Making the list stricter would be a trade-off with application compatibility. A table with a significant part of the blocked calls and the reasoning for blocking can be found here: <https://docs.docker.com/engine/security/seccomp/>.
    Seccomp uses the Berkeley Packet Filter (BPF) system, which is programmable on the fly so you can make a custom filter. You can also limit a certain syscall by also customizing the conditions on how or when it should be limited. A seccomp filter replaces the syscall with a pointer to a BPF program, which will execute that program instead of the syscall. All children to a process with this filter will inherit the filter as well. The docker option which is used to operate with seccomp is `--security-opt`.


<a id="orgca10dae"></a>

#### Copy-on-write FS

Copy-on-write is a mechanism allowing to share data. The data appears to be a copy, but is only a link (or reference) to the original data. The actual copy happens only when someone tries to change the shared data. Whoever changes the shared data ends up sharing their own copy instead. There are multiple file systems which support copy-on-write semantics but the default one for RHEL systems is **devicemapper**. 

The device mapper graphdriver uses the device mapper thin provisioning module (dm-thinp) to implement CoW snapshots. The preferred model is to have a thin pool reserved outside of Docker and passed to the daemon via the &#x2013;storage-opt dm.thinpooldev option. Alternatively, the device mapper graphdriver can setup a block device to handle this for you via the &#x2013;storage-opt dm.directlvm\_device option.

As a fallback if no thin pool is provided, loopback files will be created. Loopback is very slow, but can be used without any pre-configuration of storage. It is strongly recommended that you do not use loopback in production. Ensure your Docker daemon has a &#x2013;storage-opt dm.thinpooldev argument provided.

In loopback, a thin pool is created at /var/lib/docker/devicemapper (devicemapper graph location) based on two block devices, one for data and one for metadata. By default these block devices are created automatically by using loopback mounts of automatically created sparse files.
As of docker-1.4.1 and later, docker info when using the devicemapper storage driver will display something like:

    $ sudo docker info
    [...]
    Storage Driver: devicemapper
     Pool Name: docker-pool00
     Pool Blocksize: 65.54 kB
     Base Device Size: 21.47 GB
     Backing Filesystem: xfs
     Data file:
     Metadata file:
     Data Space Used: 3.395 GB
     Data Space Total: 107.2 GB
     Data Space Available: 103.8 GB
     Metadata Space Used: 5.034 MB
     Metadata Space Total: 104.9 MB
     Metadata Space Available: 99.82 MB
     Thin Pool Minimum Free Space: 10.72 GB
     Udev Sync Supported: true
     Deferred Removal Enabled: true
     Deferred Deletion Enabled: false
     Deferred Deleted Device Count: 0
     Library Version: 1.02.135-RHEL7 (2016-11-16)
    [...]

There is also the option of OverlayFS, but it's still experimental on RHEL 7. Probably when RHEL8 is released it will be the default copy-on-write backend for containers. 
 Docker relies heavily on the copy-on-write filesystem for it's image layers approach to work. Each image is usualy made of several layers which build on top of one another to get to the final state. These layers can also be re-used across images. For example image you have a base os image with the packages needed, then another layer with java and java dependencies and lastly a layer with your java based application. The advantage of this approach is you can reuse layers 1 and 2 for all your java apps so they occupy less space and work from the same source.
 The major difference between a container and an image is the top writable layer. All writes to the container that add new or modify existing data are stored in this writable layer. When the container is deleted, the writable layer is also deleted. The underlying image remains unchanged.
 Because each container has its own writable container layer, and all changes are stored in this container layer, multiple containers can share access to the same underlying image and yet have their own data state. The diagram below shows multiple containers sharing the same Ubuntu 15.04 image:
Inline-style: 
![layers](http://blog-assets.risingstack.com/2015/05/docker-layers.png "Layers")



Docker uses storage drivers to manage the contents of the image layers and the writable container layer. Each storage driver handles the implementation differently, but all drivers use stackable image layers and the copy-on-write (CoW) strategy.


<a id="org721e4c5"></a>

### Kubernetes Control Plane

Kubernetes components can be grouped in two categories, master components and node components.

The Kubernetes control plane is split into a set of components, which can all run on a single master node, or can be replicated in order to support high-availability clusters, or can even be run on Kubernetes itself ([AKA self-hosted](https://github.com/kubernetes/community/blob/master/contributors/design-proposals/architecture/self-hosted-kubernetes.md#what-is-self-hosted)).

Kubernetes provides a REST API supporting primarily CRUD operations on (mostly) persistent resources, which serve as the hub of its control plane. Kubernetes’s API provides IaaS-like container-centric primitives such as Pods, Services, and Ingress, and also lifecycle APIs to support orchestration (self-healing, scaling, updates, termination) of common types of workloads, such as ReplicaSet (simple fungible/stateless app manager), Deployment (orchestrates updates of stateless apps), Job (batch), CronJob (cron), DaemonSet (cluster services), and StatefulSet (stateful apps). We deliberately decoupled service naming/discovery and load balancing from application implementation, since the latter is diverse and open-ended.

Both user clients and components containing asynchronous controllers interact with the same API resources, which serve as coordination points, common intermediate representation, and shared state. Most resources contain metadata, including labels and annotations, fully elaborated desired state (spec), including default values, and observed state (status).

Controllers work continuously to drive the actual state towards the desired state, while reporting back the currently observed state for users and for other controllers.

While the controllers are level-based to maximize fault tolerance, they typically watch for changes to relevant resources in order to minimize reaction latency and redundant work. This enables decentralized and decoupled choreography-like coordination without a message bus.

The following master components are available on Kubernetes:

-   **kube-apiserver**. kube-apiserver exposes the Kubernetes API. It is the front-end for the Kubernetes control plane. It is designed to scale horizontally – that is, it scales by deploying more instances.
-   **etcd** - etcd is used as Kubernetes’ backing store. All cluster data is stored here. Always have a backup plan for etcd’s data for your Kubernetes cluster.
-   **kube-controller-manager**. kube-controller-manager runs controllers, which are the background threads that handle routine tasks in the cluster. Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

-   Node Controller: Responsible for noticing and responding when nodes go down.
-   Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
-   Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
-   Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

-   **cloud-controller-manager**. cloud-controller-manager runs controllers that interact with the underlying cloud providers. The cloud-controller-manager binary is an alpha feature introduced in Kubernetes release 1.6. cloud-controller-manager runs cloud-provider-specific controller loops only. You must disable these controller loops in the kube-controller-manager. You can disable the controller loops by setting the &#x2013;cloud-provider flag to external when starting the kube-controller-manager.

cloud-controller-manager allows cloud vendors code and the Kubernetes core to evolve independent of each other. In prior releases, the core Kubernetes code was dependent upon cloud-provider-specific code for functionality. In future releases, code specific to cloud vendors should be maintained by the cloud vendor themselves, and linked to cloud-controller-manager while running Kubernetes.
The following controllers have cloud provider dependencies:

-   Node Controller: For checking the cloud provider to determine if a node has been deleted in the cloud after it stops responding
-   Route Controller: For setting up routes in the underlying cloud infrastructure
-   Service Controller: For creating, updating and deleting cloud provider load balancers
-   Volume Controller: For creating, attaching, and mounting volumes, and interacting with the cloud provider to orchestrate volumes

-   **kube-scheduler**. kube-scheduler watches newly created pods that have no node assigned, and selects a node for them to run on.
-   **addons**. Addons are pods and services that implement cluster features. The pods may be managed by Deployments, ReplicationControllers, and so on. Namespaced addon objects are created in the kube-system namespace. Addon manager creates and maintains addon resources.
-   **DNS**. While the other addons are not strictly required, all Kubernetes clusters should have cluster DNS, as many examples rely on it. Cluster DNS is a DNS server, in addition to the other DNS server(s) in your environment, which serves DNS records for Kubernetes services. Containers started by Kubernetes automatically include this DNS server in their DNS searches.
-   **Web UI (Dashboard)**. Dashboard is a general purpose, web-based UI for Kubernetes clusters. It allows users to manage and troubleshoot applications running in the cluster, as well as the cluster itself.
-   **Container Resource Monitoring**. Container Resource Monitoring records generic time-series metrics about containers in a central database, and provides a UI for browsing that data.
-   **Cluster-level Logging**. A Cluster-level logging mechanism is responsible for saving container logs to a central log store with search/browsing interface.

Reference: 
![alt text][architecture]

[architecture]: https://blog.jetstack.io/images/k8s/architecture-small.png "Kube architecture"


<a id="orgf660e6b"></a>

#### Kube API Server

The API server serves up the Kubernetes API. It is intended to be a relatively simple server, with most/all business logic implemented in separate components or in plug-ins. It mainly processes REST operations, validates them, and updates the corresponding objects in etcd (and perhaps eventually other stores). Note that, for a number of reasons, Kubernetes deliberately does not support atomic transactions across multiple resources.
Additionally, the API server acts as the gateway to the cluster. By definition, the API server must be accessible by clients from outside the cluster, whereas the nodes, and certainly containers, may not be. Clients authenticate the API server and also use it as a bastion and proxy/tunnel to nodes and pods (and services).

The API Server is the central management entity and the only component that directly talks with the distributed storage component etcd. It provides the following core functionality:

-   Serves the Kubernetes API, used cluster-internally by the worker nodes as well as externally by kubectl
-   Proxies cluster components such as the Kubernetes UI
-   Allows the manipulation of the state of objects, for example pods and services
-   Persists the state of objects in a distributed storage (etcd)

The Kubernetes API is a HTTP API with JSON as its primary serialization schema, however it also supports Protocol Buffers, mainly for cluster-internal communication. For extensibility reasons Kubernetes supports multiple API versions at different API paths, such as  /api/v1 or /apis/extensions/v1beta1. Different API versions imply different levels of stability and support:

-   Alpha level, for example v1alpha1 is disabled by default, support for a feature may be dropped at any time without notice and should only be used in short-lived testing clusters.
-   Beta level, for example v2beta3, is enabled by default, means that the code is well tested but the semantics of objects may change in incompatible ways in a subsequent beta or stable release.
-   Stable level, for example, v1 will appear in released software for many subsequent versions.

Let’s now have a look at how the HTTP API space is constructed. At the top level we distinguish between the core group (everything below /api/v1, for historic reasons under this path and not under /apis/core/v1), the named groups (at path /apis/$NAME/$VERSION) and system-wide entities such as /metrics.
A part of the HTTP API space (based on v1.5) is shown in the following:

![alt text][api_server]

[api_server]:https://blog.openshift.com/wp-content/uploads/API-server-space-1024x604.png "Api Server"


Going forward we’ll be focusing on a concrete example: batch operations. In Kubernetes 1.5, two versions of batch operations exist: /apis/batch/v1 and /apis/batch/v2alpha1, exposing different sets of entities that can be queried and manipulated.

Now we turn our attention to an exemplary interaction with the API (we are using Minishift and the proxy command oc proxy &#x2013;port=8080 here to get direct access to the API): 

    $ curl http://127.0.0.1:8080/apis/batch/v1
    {
      "kind": "APIResourceList",
      "apiVersion": "v1",
      "groupVersion": "batch/v1",
      "resources": [
        {
          "name": "jobs",
          "namespaced": true,
          "kind": "Job"
        },
        {
          "name": "jobs/status",
          "namespaced": true,
          "kind": "Job"
        }
      ]
    }

And further, using the new, alpha version:

    $ curl http://127.0.0.1:8080/apis/batch/v2alpha1
    {
      "kind": "APIResourceList",
      "apiVersion": "v1",
      "groupVersion": "batch/v2alpha1",
      "resources": [
        {
          "name": "cronjobs",
          "namespaced": true,
          "kind": "CronJob"
        },
        {
          "name": "cronjobs/status",
          "namespaced": true,
          "kind": "CronJob"
        },
        {
          "name": "jobs",
          "namespaced": true,
          "kind": "Job"
        },
        {
          "name": "jobs/status",
          "namespaced": true,
          "kind": "Job"
        },
        {
          "name": "scheduledjobs",
          "namespaced": true,
          "kind": "ScheduledJob"
        },
        {
          "name": "scheduledjobs/status",
          "namespaced": true,
          "kind": "ScheduledJob"
        }
      ]
    }

In general the Kubernetes API supports create, update, delete, and retrieve operations at the given path via the standard HTTP verbs POST, PUT, DELETE, and GET with JSON as the default payload.

Most API objects make a distinction between the specification of the desired state of the object and the status of the object at the current time. A specification is a complete description of the desired state and is persisted in stable storage.

1.  Terminology

    After this brief overview of the API Server and the HTTP API space and its properties, we now define the terms used in this context more formally. Primitives like pods, services, endpoints, deployment, etc. make up the objects of the Kubernetes type universe. We use the following terms:
    
    Kind is the type of an entity. Each object has a field Kind which tells a client—such as kubectl or oc—that it represents, for example, a pod:
    
        apiVersion: v1
        kind: Pod
        metadata:
          name: webserver
        spec:
          containers:
          - name: nginx
            image: nginx:1.9
            ports:
            - containerPort: 80
    
    There are three categories of Kinds:
    
    -   Objects represent a persistent entity in the system. An object may have multiple resources that clients can use to perform specific actions. Examples: Pod and Namespace.
    -   Lists are collections of resources of one or more kinds of entities. Lists have a limited set of common metadata. Examples: PodLists and NodeLists.
    -   Special purpose kinds are for example used for specific actions on objects and for non-persistent entities such as /binding or /status, discovery uses APIGroup and APIResource, error results use Status, etc.
    
    **API Group** is a collection of Kinds that are logically related. For example, all batch objects like Job or ScheduledJob are in the batch API Group.
    
    **Version**. Each API Group can exist in multiple versions. For example, a group first appears as v1alpha1 and is then promoted to v1beta1 and finally graduates to v1. An object created in one version (e.g. v1beta1) can be retrieved in each of the supported versions (for example as v1). The API server does lossless conversion to return objects in the requested version.
    
    **Resource** is the representation of a system entity sent or retrieved as JSON via HTTP; can be exposed as an individual resource (such as &#x2026;/namespaces/default) or collections of resources (like &#x2026;/jobs).
    
    An API Group, a Version and a Resource (GVR) uniquely defines a HTTP path:
    
    More precisely, the actual path for jobs is /apis/batch/v1/namespaces/$NAMESPACE/jobs because jobs are not a cluster-wide resource, in contrast to, for example, node resources. For brevity, we omit the $NAMESPACE segment of the paths throughout the post.
    
    Note that Kinds may not only exist in different versions, but also in different API Groups simultaneously. For example, Deployment started as an alpha Kind in the extensions group and was eventually promoted to a GA version in its own group apps.k8s.io. Hence, to identify Kinds uniquely you need the API Group, the version and the kind name (GVK).

2.  Request Flow and Processing

    Now that we’ve reviewed the terminology used in the Kubernetes API we move on to how API requests are processed. The API lives in k8s.io/pkg/api and handles requests from within the cluster as well as to clients outside of the cluster.
    
    So, what actually happens now when an HTTP request hits the Kubernetes API? On a high level, the following interactions take place:
    
    -   The HTTP request is processed by a chain of filters registered in DefaultBuildHandlerChain() (see config.go) that applies an operation on it (see below for more details on the filters). Either the filter passes and attaches respective infos to ctx.RequestInfo, such as authenticated user or returns an appropriate HTTP response code.
    -   Next, the multiplexer (see container.go) routes the HTTP request to the respective handler, depending on the HTTP path.
    -   The routes (as defined in routes/\*) connect handlers with HTTP paths.
    -   The handler, registered per API Group (see groupversion.go and installer.go) takes the HTTP request and context (like user, rights, etc.) and delivers the requested object from storage.
    
    The complete flow looks as follows:
![alt text][api_server_flow]

[api_server_flow]: https://blog.openshift.com/wp-content/uploads/API-server-flow-768x835.png "Api server flow"
	


3.  Additional Reading

    -   <https://blog.openshift.com/kubernetes-deep-dive-api-server-part-1/>
    -   <https://blog.openshift.com/kubernetes-deep-dive-api-server-part-2/>
    -   <https://blog.openshift.com/kubernetes-deep-dive-api-server-part-3a/>


<a id="org32908be"></a>

#### ETCD (Cluster state store)

From your \*nix operating system you know that /etc is used to store config data and, in fact, the name etcd is inspired by this, adding the “d” for distributed. Any distributed system will likely need something like etcd to store data about the state of the system, enabling it to retrieve the state in a consistent and reliable fashion. To coordinate the data access in a distributed setup, etcd uses the Raft protocol. 
In many ways etcd is similar to zookeeper as they are both used for cluster coordination. You can implement on top of both different strategies like distributed lock or service discovery. Both of these are not designed to store lots of data.
In our production systems we run etcd in nodes of 3 (etcd is deployed with 2n+1 peer services to ensure quorum in case of partition), but it can also be run as a single node for testing and development environments. 
To interact with etcd you can use their command line client [etcdctl](https://github.com/coreos/etcd/tree/master/etcdctl). You can also use plain \`curl\` to interact with it. A small guide on how to interact with it is available [here](https://coreos.com/etcd/docs/latest/demo.html).

Conceptually, the data model etcd supports is that of key-value store. In etcd2 the keys formed a hierarchy and with the introduction of etcd3 this has turned into a flat model, while maintaining backwards compatibility concerning hierarchical keys.


Using a containerized version of etcd, we can create the above tree and then retrieve it as follows:

    $ docker run --rm -d -p 2379:2379 \ 
     --name test-etcd3 quay.io/coreos/etcd:v3.1.0 /usr/local/bin/etcd \
     --advertise-client-urls http://0.0.0.0:2379 --listen-client-urls http://0.0.0.0:2379
    $ curl localhost:2379/v2/keys/foo -XPUT -d value="some value"
    $ curl localhost:2379/v2/keys/bar/this -XPUT -d value=42
    $ curl localhost:2379/v2/keys/bar/that -XPUT -d value=take
    $ http localhost:2379/v2/keys/?recursive=true
    HTTP/1.1 200 OK 
    Content-Length: 327
    Content-Type: application/json
    Date: Tue, 06 Jun 2017 12:28:28 GMT
    X-Etcd-Cluster-Id: 10e5e39849dab251
    X-Etcd-Index: 6
    X-Raft-Index: 7
    X-Raft-Term: 2
    
    {
        "action": "get",
        "node": {
    	"dir": true,
    	"nodes": [
    	    {
    		"createdIndex": 4,
    		"key": "/foo",
    		"modifiedIndex": 4,
    		"value": "some value"
    	    },
    	    {
    		"createdIndex": 5,
    		"dir": true,
    		"key": "/bar",
    		"modifiedIndex": 5,
    		"nodes": [
    		    {
    			"createdIndex": 5,
    			"key": "/bar/this",
    			"modifiedIndex": 5,
    			"value": "42"
    		    },
    		    {
    			"createdIndex": 6,
    			"key": "/bar/that",
    			"modifiedIndex": 6,
    			"value": "take"
    		    }
    		]
    	    }
    	]
        }
    }

To get a real feel for ETCD you can use their online simulator available at <http://play.etcd.io/play>. 
Now that we’ve established how etcd works in principle, let’s move on to the subject of how etcd is used in Kubernetes.

1.  Cluster state in etcd

    In Kubernetes, etcd is an independent component of the control plane. Up to Kubernetes 1.5.2, we used etcd2 and from then on switched to etcd3. Note that in Kubernetes 1.5.x etcd3 is still used in v2 API mode and going forward this is changing to the v3 API, including the data model used. From a developer’s point of view this doesn’t have direct implications, because the API Server takes care of abstracting the interactions away—compare the storage backend implementation for v2 vs. v3. However, from a cluster admin’s perspective, it’s relevant to know which etcd version is used, as maintenance tasks such as backup and restore need to be handled differently.
    
    You can influence the way the API Server is using etcd via a number of options at start-up time; also, note that the output below was edited to highlight the most important bits:
    
        $ kube-apiserver -h
        ...
        --etcd-cafile string   SSL Certificate Authority file used to secure etcd communication.
        --etcd-certfile string SSL certification file used to secure etcd communication.
        --etcd-keyfile string  SSL key file used to secure etcd communication.
        ...
        --etcd-quorum-read     If true, enable quorum read.
        --etcd-servers         List of etcd servers to connect with (scheme://ip:port) …
        ...
    
    Kubernetes stores its objects in etcd either as a JSON string or in Protocol Buffers (“protobuf” for short) format. Let’s have a look at a concrete example: We launch a pod webserver in namespace apiserver-sandbox :
    
        $ cat pod.yaml
        apiVersion: v1
        kind: Pod
        metadata:
          name: webserver
        spec:
          containers:
          - name: nginx
            image: tomaskral/nonroot-nginx
            ports:
            - containerPort: 80
        
        $ kubectl create -f pod.yaml 
        
        $ etcdctl ls /
        /kubernetes.io
        /openshift.io
        
        $ etcdctl get /kubernetes.io/pods/apiserver-sandbox/webserver
        {
          "kind": "Pod",
          "apiVersion": "v1",
          "metadata": {
            "name": "webserver",
        ...
    
    So, how does the object payload end up in etcd, starting from kubectl create -f pod.yaml?
      
    1.  A client such as kubectl provides an desired object state, for example, YAML in version v1.
    2.  kubectl converts the YAML into JSON to send it over the wire.
    3.  Between different versions of the same kind, the API server can perform a lossless conversion leveraging annotations to store information that cannot be expressed in older API versions.
    4.  The API Server turns the input object state into a canonical storage version, depending on the API Server version itself, usually the newest stable one, for example, v1.
    5.  Last but not least comes actual storage process in etcd, at a certain key, into a value with the encoding to JSON or protobuf.


<a id="org6a990b2"></a>

#### Controller-Manager Server

kube-controller-manager runs controllers, which are the background threads that handle routine tasks in the cluster. Logically, each controller is a separate process, but to reduce complexity, they are all compiled into a single binary and run in a single process.

These controllers include:

-   Node Controller: Responsible for noticing and responding when nodes go down.
-   Replication Controller: Responsible for maintaining the correct number of pods for every replication controller object in the system.
-   Endpoints Controller: Populates the Endpoints object (that is, joins Services & Pods).
-   Service Account & Token Controllers: Create default accounts and API access tokens for new namespaces.

1.  Node Controller

     The node controller is a Kubernetes master component which manages various aspects of nodes.
    The node controller has multiple roles in a node’s life. The first is assigning a CIDR block to the node when it is registered (if CIDR assignment is turned on).
    The second is keeping the node controller’s internal list of nodes up to date with the cloud provider’s list of available machines. When running in a cloud environment, whenever a node is unhealthy, the node controller asks the cloud provider if the VM for that node is still available. If not, the node controller deletes the node from its list of nodes.
    The third is monitoring the nodes’ health. The node controller is responsible for updating the NodeReady condition of NodeStatus to ConditionUnknown when a node becomes unreachable (i.e. the node controller stops receiving heartbeats for some reason, e.g. due to the node being down), and then later evicting all the pods from the node (using graceful termination) if the node continues to be unreachable. (The default timeouts are 40s to start reporting ConditionUnknown and 5m after that to start evicting pods.) The node controller checks the state of each node every &#x2013;node-monitor-period seconds.
    In Kubernetes 1.4, the logic of the node controller was updated to better handle cases when a large number of nodes have problems with reaching the master (e.g. because the master has networking problem). Starting with 1.4, the node controller will look at the state of all nodes in the cluster when making a decision about pod eviction.
    In most cases, node controller limits the eviction rate to &#x2013;node-eviction-rate (default 0.1) per second, meaning it won’t evict pods from more than 1 node per 10 seconds.
    The node eviction behavior changes when a node in a given availability zone becomes unhealthy. The node controller checks what percentage of nodes in the zone are unhealthy (NodeReady condition is ConditionUnknown or ConditionFalse) at the same time. If the fraction of unhealthy nodes is at least &#x2013;unhealthy-zone-threshold (default 0.55) then the eviction rate is reduced: if the cluster is small (i.e. has less than or equal to &#x2013;large-cluster-size-threshold nodes - default 50) then evictions are stopped, otherwise the eviction rate is reduced to &#x2013;secondary-node-eviction-rate (default 0.01) per second. The reason these policies are implemented per availability zone is because one availability zone might become partitioned from the master while the others remain connected. If your cluster does not span multiple cloud provider availability zones, then there is only one availability zone (the whole cluster).
    A key reason for spreading your nodes across availability zones is so that the workload can be shifted to healthy zones when one entire zone goes down. Therefore, if all nodes in a zone are unhealthy then node controller evicts at the normal rate &#x2013;node-eviction-rate. The corner case is when all zones are completely unhealthy (i.e. there are no healthy nodes in the cluster). In such case, the node controller assumes that there’s some problem with master connectivity and stops all evictions until some connectivity is restored.
    Starting in Kubernetes 1.6, the NodeController is also responsible for evicting pods that are running on nodes with NoExecute taints, when the pods do not tolerate the taints. Additionally, as an alpha feature that is disabled by default, the NodeController is responsible for adding taints corresponding to node problems like node unreachable or not ready. See this documentation for details about NoExecute taints and the alpha feature.
    Starting in version 1.8, the node controller can be made responsible for creating taints that represent Node conditions. This is an alpha feature of version 1.8.

2.  Replication Controller

    A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.
    If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the ReplicationController starts more pods. Unlike manually created pods, the pods maintained by a ReplicationController are automatically replaced if they fail, are deleted, or are terminated. For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade. For this reason, you should use a ReplicationController even if your application requires only a single pod. A ReplicationController is similar to a process supervisor, but instead of supervising individual processes on a single node, the ReplicationController supervises multiple pods across multiple nodes.
    ReplicationController is often abbreviated to “rc” or “rcs” in discussion, and as a shortcut in kubectl commands.
    A simple case is to create one ReplicationController object to reliably run one instance of a Pod indefinitely. A more complex use case is to run several identical replicas of a replicated service, such as web servers.

3.  Endpoints controller

    Kubernetes Pods are mortal. They are born and when they die, they are not resurrected. ReplicationControllers in particular create and destroy Pods dynamically (e.g. when scaling up or down or when doing rolling updates). While each Pod gets its own IP address, even those IP addresses cannot be relied upon to be stable over time. This leads to a problem: if some set of Pods (let’s call them backends) provides functionality to other Pods (let’s call them frontends) inside the Kubernetes cluster, how do those frontends find out and keep track of which backends are in that set? Enter Services.
    A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. The set of Pods targeted by a Service is (usually) determined by a Label Selector (see below for why you might want a Service without a selector).
    As an example, consider an image-processing backend which is running with 3 replicas. Those replicas are fungible - frontends do not care which backend they use. While the actual Pods that compose the backend set may change, the frontend clients should not need to be aware of that or keep track of the list of backends themselves. The Service abstraction enables this decoupling.
    For Kubernetes-native applications, Kubernetes offers a simple Endpoints API that is updated whenever the set of Pods in a Service changes. For non-native applications, Kubernetes offers a virtual-IP-based bridge to Services which redirects to the backend Pods.
    
    Endpoints are managed by the endpoints controller. 


<a id="orgedb5187"></a>

#### Scheduler

The Kubernetes scheduler is a policy-rich, topology-aware, workload-specific function that significantly impacts availability, performance, and capacity. The scheduler needs to take into account individual and collective resource requirements, quality of service requirements, hardware/software/policy constraints, affinity and anti-affinity specifications, data locality, inter-workload interference, deadlines, and so on. Workload-specific requirements will be exposed through the API as necessary.

1.  The scheduling algorithm

        For given pod:
        
            +---------------------------------------------+
            |               Schedulable nodes:            |
            |                                             |
            | +--------+    +--------+      +--------+    |
            | | node 1 |    | node 2 |      | node 3 |    |
            | +--------+    +--------+      +--------+    |
            |                                             |
            +-------------------+-------------------------+
        			|
        			|
        			v
            +-------------------+-------------------------+
        
            Pred. filters: node 3 doesn't have enough resource
        
            +-------------------+-------------------------+
        			|
        			|
        			v
            +-------------------+-------------------------+
            |             remaining nodes:                |
            |   +--------+                 +--------+     |
            |   | node 1 |                 | node 2 |     |
            |   +--------+                 +--------+     |
            |                                             |
            +-------------------+-------------------------+
        			|
        			|
        			v
            +-------------------+-------------------------+
        
            Priority function:    node 1: p=2
        			  node 2: p=5
        
            +-------------------+-------------------------+
        			|
        			|
        			v
        	    select max{node priority} = node 2
    
    The Scheduler tries to find a node for each Pod, one at a time.
    
    -   First it applies a set of "predicates" to filter out inappropriate nodes. For example, if the PodSpec specifies resource requests, then the scheduler will filter out nodes that don't have at least that much resources available (computed as the capacity of the node minus the sum of the resource requests of the containers that are already running on the node).
    -   Second, it applies a set of "priority functions" that rank the nodes that weren't filtered out by the predicate check. For example, it tries to spread Pods across nodes and zones while at the same time favoring the least (theoretically) loaded nodes (where "load" - in theory - is measured as the sum of the resource requests of the containers running on the node, divided by the node's capacity).
    -   Finally, the node with the highest priority is chosen (or, if there are multiple such nodes, then one of them is chosen at random).

2.  Predicates and priorities policies

    Predicates are a set of policies applied one by one to filter out inappropriate nodes. Priorities are a set of policies applied one by one to rank nodes (that made it through the filter of the predicates). By default Kuberneres provides the following built-in policies:
    
    1.  Filtering the nodes
    
        The purpose of filtering the nodes is to filter out the nodes that do not meet certain requirements of the Pod. For example, if the free resource on a node (measured by the capacity minus the sum of the resource requests of all the Pods that already run on the node) is less than the Pod's required resource, the node should not be considered in the ranking phase so it is filtered out. Currently, there are several "predicates" implementing different filtering policies, including:
        
        -   **NoDiskConflict**: Evaluate if a pod can fit due to the volumes it requests, and those that are already mounted. Currently supported volumes are: AWS EBS, GCE PD, ISCSI and Ceph RBD. Only Persistent Volume Claims for those supported types are checked. Persistent Volumes added directly to pods are not evaluated and are not constrained by this policy.
        -   **NoVolumeZoneConflict**: Evaluate if the volumes a pod requests are available on the node, given the Zone restrictions.
        -   **PodFitsResources**: Check if the free resource (CPU and Memory) meets the requirement of the Pod. The free resource is measured by the capacity minus the sum of requests of all Pods on the node. To learn more about the resource QoS in Kubernetes, please check QoS proposal.
        -   **PodFitsHostPorts**: Check if any HostPort required by the Pod is already occupied on the node.
        -   **HostName**: Filter out all nodes except the one specified in the PodSpec's NodeName field.
        -   **MatchNodeSelector**: Check if the labels of the node match the labels specified in the Pod's nodeSelector field and, as of Kubernetes v1.2, also match the scheduler.alpha.kubernetes.io/affinity pod annotation if present. See here for more details on both.
        -   **MaxEBSVolumeCount**: Ensure that the number of attached ElasticBlockStore volumes does not exceed a maximum value (by default, 39, since Amazon recommends a maximum of 40 with one of those 40 reserved for the root volume &#x2013; see Amazon's documentation). The maximum value can be controlled by setting the KUBE\_MAX\_PD\_VOLS environment variable.
        -   **MaxGCEPDVolumeCount**: Ensure that the number of attached GCE PersistentDisk volumes does not exceed a maximum value (by default, 16, which is the maximum GCE allows &#x2013; see GCE's documentation). The maximum value can be controlled by setting the KUBE\_MAX\_PD\_VOLS environment variable.
        -   **CheckNodeMemoryPressure**: Check if a pod can be scheduled on a node reporting memory pressure condition. Currently, no BestEffort should be placed on a node under memory pressure as it gets automatically evicted by kubelet.
        -   **CheckNodeDiskPressure**: Check if a pod can be scheduled on a node reporting disk pressure condition. Currently, no pods should be placed on a node under disk pressure as it gets automatically evicted by kubelet.
        
        All predicates mentioned above can be used in combination to perform a sophisticated filtering policy. Kubernetes uses some, but not all, of these predicates by default. 
    
    2.  Ranking the nodes
    
        The filtered nodes are considered suitable to host the Pod, and it is often that there are more than one nodes remaining. Kubernetes prioritizes the remaining nodes to find the "best" one for the Pod. The prioritization is performed by a set of priority functions. For each remaining node, a priority function gives a score which scales from 0-10 with 10 representing for "most preferred" and 0 for "least preferred". Each priority function is weighted by a positive number and the final score of each node is calculated by adding up all the weighted scores. For example, suppose there are two priority functions, priorityFunc1 and priorityFunc2 with weighting factors weight1 and weight2 respectively, the final score of some NodeA is:
            `finalScoreNodeA = (weight1 * priorityFunc1) + (weight2 * priorityFunc2)`
        
        After the scores of all nodes are calculated, the node with highest score is chosen as the host of the Pod. If there are more than one nodes with equal highest scores, a random one among them is chosen.
        
        Currently, Kubernetes scheduler provides some practical priority functions, including:
            **LeastRequestedPriority**: The node is prioritized based on the fraction of the node that would be free if the new Pod were scheduled onto the node. (In other words, (capacity - sum of requests of all Pods already on the node - request of Pod that is being scheduled) / capacity). CPU and memory are equally weighted. The node with the highest free fraction is the most preferred. Note that this priority function has the effect of spreading Pods across the nodes with respect to resource consumption.
            **BalancedResourceAllocation**: This priority function tries to put the Pod on a node such that the CPU and Memory utilization rate is balanced after the Pod is deployed.
            **SelectorSpreadPriority**: Spread Pods by minimizing the number of Pods belonging to the same service, replication controller, or replica set on the same node. If zone information is present on the nodes, the priority will be adjusted so that pods are spread across zones and nodes.
            **CalculateAntiAffinityPriority**: Spread Pods by minimizing the number of Pods belonging to the same service on nodes with the same value for a particular label.
            **ImageLocalityPriority**: Nodes are prioritized based on locality of images requested by a pod. Nodes with larger size of already-installed packages required by the pod will be preferred over nodes with no already-installed packages required by the pod or a small total size of already-installed packages required by the pod.
            **NodeAffinityPriority**: (Kubernetes v1.2) Implements preferredDuringSchedulingIgnoredDuringExecution node affinity; see here for more details.
            Kubernetes uses some, but not all, of these priority functions by default. Similar as predicates, you can combine the above priority functions and assign weight factors (positive number) to them as you want.


<a id="org83f617f"></a>

#### Addons

Cluster add-ons are resources like Services and Deployments (with pods) that are shipped with the Kubernetes binaries and are considered an inherent part of the Kubernetes clusters.

There are currently two classes of add-ons:

-   Add-ons that will be reconciled.
-   Add-ons that will be created if they don't exist.

1.  Addon Manager

    addon-manager manages two classes of addons with given template files.
    
    -   Addons with label `addonmanager.kubernetes.io/mode=Reconcile` will be periodically reconciled. Direct manipulation to these addons through apiserver is discouraged because addon-manager will bring them back to the original state. In particular:
        -   Addon will be re-created if it is deleted.
        -   Addon will be reconfigured to the state given by the supplied fields in the template file periodically.
        -   Addon will be deleted when its manifest file is deleted.
    -   Addons with label `addonmanager.kubernetes.io/mode=EnsureExists` will be checked for existence only. Users can edit these addons as they want. In particular:
        -   Addon will only be created/re-created with the given template file when there is no instance of the resource with that name.
        -   Addon will not be deleted when the manifest file is deleted.
    
    ---
    
    **Notes**:
    
    -   Label `kubernetes.io/cluster-service=true` is deprecated (only for Addon Manager). In future release (after one year), Addon Manager may not respect it anymore. Addons have this label but without `addonmanager.kubernetes.io/mode=EnsureExists` will be treated as "reconcile class addons" for now.
    -   Resources under $ADDON\_PATH (default *etc/kubernetes/addons*) needs to have either one of these two labels. Meanwhile namespaced resources need to be in kube-system namespace. Otherwise it will be omitted.
    -   The above label and namespace rule does not stand for *opt/namespace.yaml and resources under /etc/kubernetes/admission-controls*. addon-manager will attempt to create them regardless during startup.
    
    ---

2.  Cooperating Horizontal / Vertical Auto-Scaling with "reconcile class addons"

    "Reconcile" class addons will be periodically reconciled to the original state given by the initial config. In order to make Horizontal / Vertical Auto-scaling functional, the related fields in config should be left unset. More specifically, leave replicas in ReplicationController / Deployment / ReplicaSet unset for Horizontal Scaling, leave resources for container unset for Vertical Scaling. The periodic reconcile won't clobbered these fields, hence they could be managed by Horizontal / Vertical Auto-scaler.


<a id="org710e14a"></a>

#### DNS

    As of Kubernetes 1.3, DNS is a built-in service launched automatically using the addon manager cluster add-on.
Kubernetes DNS schedules a DNS Pod and Service on the cluster, and configures the kubelets to tell individual containers to use the DNS Service’s IP to resolve DNS names.
     Every Service defined in the cluster (including the DNS server itself) is assigned a DNS name. By default, a client Pod’s DNS search list will include the Pod’s own namespace and the cluster’s default domain. This is best illustrated by example:
Assume a Service named foo in the Kubernetes namespace bar. A Pod running in namespace bar can look up this service by simply doing a DNS query for foo. A Pod running in namespace quux can look up this service by doing a DNS query for foo.bar

1.  Supported DNS schema

    The following sections detail the supported record types and layout that is supported. Any other layout or names or queries that happen to work are considered implementation details and are subject to change without warning. For more up-to-date specification, see [Kubernetes DNS-Based Service Discovery](https://github.com/kubernetes/dns/blob/master/docs/specification.md).
    
    1.  Services
    
          **A records**
        “Normal” (not headless) Services are assigned a DNS A record for a name of the form my-svc.my-namespace.svc.cluster.local. This resolves to the cluster IP of the Service.
        “Headless” (without a cluster IP) Services are also assigned a DNS A record for a name of the form my-svc.my-namespace.svc.cluster.local. Unlike normal Services, this resolves to the set of IPs of the pods selected by the Service. Clients are expected to consume the set or else use standard round-robin selection from the set.
        **SRV records**
        SRV Records are created for named ports that are part of normal or Headless Services. For each named port, the SRV record would have the form `_my-port-name._my-port-protocol.my-svc.my-namespace.svc.cluster.local`. For a regular service, this resolves to the port number and the CNAME: `my-svc.my-namespace.svc.cluster.local`. For a headless service, this resolves to multiple answers, one for each pod that is backing the service, and contains the port number and a CNAME of the pod of the form `auto-generated-name.my-svc.my-namespace.svc.cluster.local`.
        **Pods**
        
        -   A Records
        
        When enabled, pods are assigned a DNS A record in the form of pod-ip-address.my-namespace.pod.cluster.local.
        For example, a pod with IP 1.2.3.4 in the namespace default with a DNS name of cluster.local would have an entry: 1-2-3-4.default.pod.cluster.local
        
        -   A Records and hostname based on Pod’s hostname and subdomain fields
        
        Currently when a pod is created, its hostname is the Pod’s metadata.name value.
        
        -   With v1.2, users can specify a Pod annotation, `pod.beta.kubernetes.io/hostname`, to specify what the Pod’s hostname should be. The Pod annotation, if specified, takes precedence over the Pod’s name, to be the hostname of the pod. For example, given a Pod with annotation `pod.beta.kubernetes.io/hostname`: my-pod-name, the Pod will have its hostname set to “my-pod-name”.
        -   With v1.3, the PodSpec has a hostname field, which can be used to specify the Pod’s hostname. This field value takes precedence over the pod.beta.kubernetes.io/hostname annotation value.
        -   v1.2 introduces a beta feature where the user can specify a Pod annotation, `pod.beta.kubernetes.io/subdomain`, to specify the Pod’s subdomain. The final domain will be “ &#x2026;svc.". For example, a Pod with the hostname annotation set to "foo", and the subdomain annotation set to "bar", in namespace "my-namespace", will have the FQDN `"foo.bar.my-namespace.svc.cluster.local"`
        -   With v1.3, the PodSpec has a subdomain field, which can be used to specify the Pod’s subdomain. This field value takes precedence over the pod.beta.kubernetes.io/subdomain annotation value.
        
        Example:
        
            apiVersion: v1
            kind: Service
            metadata:
              name: default-subdomain
            spec:
              selector:
                name: busybox
              clusterIP: None
              ports:
              - name: foo # Actually, no port is needed.
                port: 1234
                targetPort: 1234
            ---
            apiVersion: v1
            kind: Pod
            metadata:
              name: busybox1
              labels:
                name: busybox
            spec:
              hostname: busybox-1
              subdomain: default-subdomain
              containers:
              - image: busybox
                command:
                  - sleep
                  - "3600"
                name: busybox
            ---
            apiVersion: v1
            kind: Pod
            metadata:
              name: busybox2
              labels:
                name: busybox
            spec:
              hostname: busybox-2
              subdomain: default-subdomain
              containers:
              - image: busybox
                command:
                  - sleep
                  - "3600"
                name: busybox


<a id="org040054a"></a>

#### WebUI

Dashboard is a web-based Kubernetes user interface. You can use Dashboard to deploy containerized applications to a Kubernetes cluster, troubleshoot your containerized application, and manage the cluster itself along with its attendant resources. You can use Dashboard to get an overview of applications running on your cluster, as well as for creating or modifying individual Kubernetes resources (such as Deployments, Jobs, DaemonSets, etc). For example, you can scale a Deployment, initiate a rolling update, restart a pod or deploy new applications using a deploy wizard.
Dashboard also provides information on the state of Kubernetes resources in your cluster, and on any errors that may have occurred.

![img](./img/kube-ui-dashboard.png "Kubernetes UI Dashboard")

Documentation on the dashboard is available at [kubernetes dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).


<a id="org451cbaa"></a>

#### Container resource monitoring

Understanding how an application behaves when deployed is crucial to scaling the application and providing a reliable service. In a Kubernetes cluster, application performance can be examined at many different levels: containers, pods, services, and whole clusters. As part of Kubernetes we want to provide users with detailed resource usage information about their running applications at all these levels. This will give users deep insights into how their applications are performing and where possible application bottlenecks may be found. In comes Heapster, a project meant to provide a base monitoring platform on Kubernetes.
Heapster is a cluster-wide aggregator of monitoring and event data. It currently supports Kubernetes natively and works on all Kubernetes setups. Heapster runs as a pod in the cluster, similar to how any Kubernetes application would run. The Heapster pod discovers all nodes in the cluster and queries usage information from the nodes’ Kubelets, the on-machine Kubernetes agent. The Kubelet itself fetches the data from cAdvisor. Heapster groups the information by pod along with the relevant labels. This data is then pushed to a configurable backend for storage and visualization. Currently supported backends include InfluxDB (with Grafana for visualization), Google Cloud Monitoring and many others described in more details here. The overall architecture of the service can be seen below:

![img](./img/kube-monitoring-architecture.png "Kubernetes Monitoring Architecture")

1.  cAdvisor

         cAdvisor is an open source container resource usage and performance analysis agent. It is purpose-built for containers and supports Docker containers natively. In Kubernetes, cAdvisor is integrated into the Kubelet binary. cAdvisor auto-discovers all containers in the machine and collects CPU, memory, filesystem, and network usage statistics. cAdvisor also provides the overall machine usage by analyzing the ‘root’ container on the machine.
    On most Kubernetes clusters, cAdvisor exposes a simple UI for on-machine containers on port 4194. Here is a snapshot of part of cAdvisor’s UI that shows the overall machine usage:
    
    ![img](./img/kube-cadvisor.png "Kubernetes cAdvisor")

2.  Kubelet

    The Kubelet acts as a bridge between the Kubernetes master and the nodes. It manages the pods and containers running on a machine. Kubelet translates each pod into its constituent containers and fetches individual container usage statistics from cAdvisor. It then exposes the aggregated pod resource usage statistics via a REST API.

3.  Prometheus

    Prometheus supports service discovery using the Kubernetes API. This allows us to define a configuration that instantly adapts to changes in the targets we want to monitor.
    The Kubernetes API provides service discovery information about the cluster’s kubelets and API servers. The following configuration section instructs Prometheus to retrieve this information, and to update its configuration as it detects changes from the Kubernetes API:
    
         - job_name: 'kubernetes_components'
        kubernetes_sd_configs:
        - api_servers:
        - 'https://kubernetes'
        in_cluster: true
        # This configures Prometheus to identify itself when scraping
        # metrics from Kubernetes cluster components.
        tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
        bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
        # Prometheus provides meta labels for each monitoring target. We use
        # these to select targets we want to monitor and to modify labels attached
        # to scraped metrics.
        relabel_configs:
        # Only scrape apiserver and kubelets.
        - source_labels: [__meta_kubernetes_role]
        action: keep
        regex: (?:apiserver|node)
        # Redefine the Prometheus job based on the monitored Kubernetes component.
        - source_labels: [__meta_kubernetes_role]
        target_label: job
        replacement: kubernetes_$1
        # Attach all node labels to the metrics scraped from the components running
        # on that node.
        - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
    
    Prometheus stores service discovery information in meta labels. These meta labels are used in Prometheus configurations to select and drop targets, or to generate proper labels for collected metrics. This happens by applying relabeling rules. The Prometheus documentation provides more detail on relabeling rules and the meta labels exposed by the Kubernetes service discovery integration. Meta labels are discarded after the targets are correctly generated.
    **Services**
    In addition to monitoring the critical components of our Kubernetes cluster, we need to monitor the services running inside of it. We can write a generic Prometheus configuration section to automatically detect new services and adjust to any changes to their endpoints:
    
         - job_name: 'kubernetes_services'
        kubernetes_sd_configs:
        - api_servers:
        - 'https://kubernetes'
        in_cluster: true
        Relabel_configs:
        # We only monitor endpoints of services that were annotated with
        # prometheus.io/scrape=true in Kubernetes
        - source_labels: [__meta_kubernetes_role, __meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: endpoint;true
        # Rewrite the Kubernetes service name into the Prometheus job label.
        - source_labels: [__meta_kubernetes_service_name]
        target_label: job
        # Attach the namespace as a label to the monitoring targets.
        - source_labels: [__meta_kubernetes_namespace]
        target_label: namespace
        # Attach all service labels to the monitoring targets.
        - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
    
    **Node exporter**
    Kubernetes keeps track of many metrics that Prometheus can read by default. Even so, there’s often more information we want to know about our cluster and infrastructure that needs to be translated to a compatible format for Prometheus. To do so, Prometheus uses exporters, small programs that read metrics from other sources and translate them to the Prometheus format. The node exporter can read system-level statistics about bare-metal nodes or virtual machines and export them for Prometheus.

4.  Monitor Node Health

     Node problem detector is a DaemonSet monitoring the node health. It collects node problems from various daemons and reports them to the apiserver as NodeCondition and Event.
    It supports some known kernel issue detection now, and will detect more and more node problems over time.
    Currently Kubernetes won’t take any action on the node conditions and events generated by node problem detector. In the future, a remedy system could be introduced to deal with node problems. The node problem detector is available [here](https://github.com/kubernetes/node-problem-detector).

5.  Cluster capacity analysis framework

     As new pods get scheduled on nodes in a cluster, more resources get consumed. Monitoring available resources in the cluster is very important as operators can increase the current resources in time before all of them get exhausted. Or, carry different steps that lead to increase of available resources.
    Cluster capacity consists of capacities of individual cluster nodes. Capacity covers CPU, memory, disk space and other resources.
    Overall remaining allocatable capacity is a rough estimation since it does not assume all resources being distributed among nodes. Goal is to analyze remaining allocatable resources and estimate available capacity that is still consumable in terms of a number of instances of a pod with given requirements that can be scheduled in a cluster.
    A tool which implements the capacity analysis support for kubernetes is available [here](https://github.com/kubernetes-incubator/cluster-capacity).


<a id="org85690c1"></a>

### Kubernetes Nodes

A node is a worker machine in Kubernetes, previously known as a minion. A node may be a VM or physical machine, depending on the cluster. Each node has the services necessary to run pods and is managed by the master components. The services on a node include Docker, kubelet and kube-proxy. 


<a id="org7db6b00"></a>

#### Kubelet

The most important and most prominent controller in Kubernetes is the Kubelet, which is the primary implementer of the Pod and Node APIs that drive the container execution layer. Without these APIs, Kubernetes would just be a CRUD-oriented REST application framework backed by a key-value store (and perhaps the API machinery will eventually be spun out as an independent project).

Kubernetes executes isolated application containers as its default, native mode of execution, as opposed to processes and traditional operating-system packages. Not only are application containers isolated from each other, but they are also isolated from the hosts on which they execute, which is critical to decoupling management of individual applications from each other and from management of the underlying cluster physical/virtual infrastructure.

Kubernetes provides Pods that can host multiple containers and storage volumes as its fundamental execution primitive in order to facilitate packaging a single application per container, decoupling deployment-time concerns from build-time concerns, and migration from physical/virtual machines. The Pod primitive is key to glean the primary benefits of deployment on modern cloud platforms, such as Kubernetes.

API admission control may reject pods or add additional scheduling constraints to them, but Kubelet is the final arbiter of what pods can and cannot run on a given node, not the schedulers or DaemonSets.

Kubelet also currently links in the cAdvisor resource monitoring agent.


<a id="org5fc764d"></a>

#### Container runtime

Each node runs a container runtime, which is responsible for downloading images and running containers.
Kubelet does not link in the base container runtime. Instead, we're defining a Container Runtime Interface to control the underlying runtime and facilitate pluggability of that layer. This decoupling is needed in order to maintain clear component boundaries, facilitate testing, and facilitate pluggability. Runtimes supported today, either upstream or by forks, include at least docker (for Linux and Windows), rkt, cri-o, and frakti.
For Kubernets 1.8 and bellow the default implementation for a container runtime is docker. However in the future this will likely change to the CRI-O implementation. Support for the CRI interface was also added in 1.5. This is available [here](https://github.com/kubernetes-incubator/cri-o).


<a id="orga98b392"></a>

#### Kube Proxy

Every node in a Kubernetes cluster runs a kube-proxy. kube-proxy is responsible for implementing a form of virtual IP for Services of type other than ExternalName. In Kubernetes v1.0 the proxy was purely in userspace. In Kubernetes v1.1 an iptables proxy was added, but was not the default operating mode. Since Kubernetes v1.2, the iptables proxy is the default.
As of Kubernetes v1.0, Services are a “layer 4” (TCP/UDP over IP) construct. In Kubernetes v1.1 the Ingress API was added (beta) to represent “layer 7” (HTTP) services.

1.  Proxy-mode: userspace

    In this mode, kube-proxy watches the Kubernetes master for the addition and removal of Service and Endpoints objects. For each Service it opens a port (randomly chosen) on the local node. Any connections to this “proxy port” will be proxied to one of the Service’s backend Pods (as reported in Endpoints). Which backend Pod to use is decided based on the SessionAffinity of the Service. Lastly, it installs iptables rules which capture traffic to the Service’s clusterIP (which is virtual) and Port and redirects that traffic to the proxy port which proxies the backend Pod.
    The net result is that any traffic bound for the Service’s IP:Port is proxied to an appropriate backend without the clients knowing anything about Kubernetes or Services or Pods.
    By default, the choice of backend is round robin. Client-IP based session affinity can be selected by setting service.spec.sessionAffinity to "ClientIP" (the default is "None"), and you can set the max session sticky time by setting the field `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` if you have already set service.spec.sessionAffinity to "ClientIP" (the default is “10800”).
    

2.  Proxy-mode: iptables

     In this mode, kube-proxy watches the Kubernetes master for the addition and removal of Service and Endpoints objects. For each Service it installs iptables rules which capture traffic to the Service’s clusterIP (which is virtual) and Port and redirects that traffic to one of the Service’s backend sets. For each Endpoints object it installs iptables rules which select a backend Pod.
    By default, the choice of backend is random. Client-IP based session affinity can be selected by setting service.spec.sessionAffinity to "ClientIP" (the default is "None"), and you can set the max session sticky time by setting the field `service.spec.sessionAffinityConfig.clientIP.timeoutSeconds` if you have already set service.spec.sessionAffinity to "ClientIP" (the default is “10800”).
    As with the userspace proxy, the net result is that any traffic bound for the Service’s `IP:Port` is proxied to an appropriate backend without the clients knowing anything about Kubernetes or Services or Pods. This should be faster and more reliable than the userspace proxy. However, unlike the userspace proxier, the iptables proxier cannot automatically retry another Pod if the one it initially selects does not respond, so it depends on having working readiness probes.
    

3.  Future work

      In the future we envision that the proxy policy can become more nuanced than simple round robin balancing, for example master-elected or sharded. We also envision that some Services will have “real” load balancers, in which case the VIP will simply transport the packets there.
     **Avoiding collisions**
     One of the primary philosophies of Kubernetes is that users should not be exposed to situations that could cause their actions to fail through no fault of their own. In this situation, we are looking at network ports - users should not have to choose a port number if that choice might collide with another user. That is an isolation failure.
     In order to allow users to choose a port number for their Services, we must ensure that no two Services can collide. We do that by allocating each Service its own IP address.
     To ensure each service receives a unique IP, an internal allocator atomically updates a global allocation map in etcd prior to creating each service. The map object must exist in the registry for services to get IPs, otherwise creations will fail with a message indicating an IP could not be allocated. A background controller is responsible for creating that map (to migrate from older versions of Kubernetes that used in memory locking) as well as checking for invalid assignments due to administrator intervention and cleaning up any IPs that were allocated but which no service currently uses.
    **IPs and VIPs**
     Unlike Pod IP addresses, which actually route to a fixed destination, Service IPs are not actually answered by a single host. Instead,  they use iptables (packet processing logic in Linux) to define virtual IP addresses which are transparently redirected as needed. When clients connect to the VIP, their traffic is automatically transported to an appropriate endpoint. The environment variables and DNS for Services are actually populated in terms of the Service’s VIP and port.
     They support two proxy modes - userspace and iptables, which operate slightly differently.
    
    -   *Userspace*
    
    As an example, consider the image processing application described above. When the backend Service is created, the Kubernetes master assigns a virtual IP address, for example 10.0.0.1. Assuming the Service port is 1234, the Service is observed by all of the kube-proxy instances in the cluster. When a proxy sees a new Service, it opens a new random port, establishes an iptables redirect from the VIP to this new port, and starts accepting connections on it.
    When a client connects to the VIP the iptables rule kicks in, and redirects the packets to the Service proxy’s own port. The Service proxy chooses a backend, and starts proxying traffic from the client to the backend.
    This means that Service owners can choose any port they want without risk of collision. Clients can simply connect to an IP and port, without being aware of which Pods they are actually accessing.
    
    -   *Iptables*
    
    Again, consider the image processing application described above. When the backend Service is created, the Kubernetes master assigns a virtual IP address, for example 10.0.0.1. Assuming the Service port is 1234, the Service is observed by all of the kube-proxy instances in the cluster. When a proxy sees a new Service, it installs a series of iptables rules which redirect from the VIP to per-Service rules. The per-Service rules link to per-Endpoint rules which redirect (Destination NAT) to the backends.
    When a client connects to the VIP the iptables rule kicks in. A backend is chosen (either based on session affinity or randomly) and packets are redirected to the backend. Unlike the userspace proxy, packets are never copied to userspace, the kube-proxy does not have to be running for the VIP to work, and the client IP is not altered.
    This same basic flow executes when traffic comes in through a node-port or through a load-balancer, though in those cases the client IP does get altered.


<a id="org1e4e4e9"></a>

### Kubernets Objects

Kubernetes Objects are persistent entities in the Kubernetes system. Kubernetes uses these entities to represent the state of your cluster. Specifically, they can describe:

-   What containerized applications are running (and on which nodes)
-   The resources available to those applications
-   The policies around how those applications behave, such as restart policies, upgrades, and fault-tolerance

A Kubernetes object is a “record of intent”–once you create the object, the Kubernetes system will constantly work to ensure that object exists. By creating an object, you’re effectively telling the Kubernetes system what you want your cluster’s workload to look like; this is your cluster’s desired state.
 To work with Kubernetes objects–whether to create, modify, or delete them–you’ll need to use the Kubernetes API. When you use the kubectl command-line interface, for example, the CLI makes the necessary Kubernetes API calls for you; you can also use the Kubernetes API directly in your own programs. Kubernetes currently provides a golang client library for this purpose, and other language libraries (such as Python) are being developed.

1.  Object Spec and Status

     Every Kubernetes object includes two nested object fields that govern the object’s configuration: the object spec and the object status. The spec, which you must provide, describes your desired state for the object–the characteristics that you want the object to have. The status describes the actual state of the object, and is supplied and updated by the Kubernetes system. At any given time, the Kubernetes Control Plane actively manages an object’s actual state to match the desired state you supplied.
    For example, a Kubernetes Deployment is an object that can represent an application running on your cluster. When you create the Deployment, you might set the Deployment spec to specify that you want three replicas of the application to be running. The Kubernetes system reads the Deployment spec and starts three instances of your desired application–updating the status to match your spec. If any of those instances should fail (a status change), the Kubernetes system responds to the difference between spec and status by making a correction–in this case, starting a replacement instance.
    When you create an object in Kubernetes, you must provide the object spec that describes its desired state, as well as some basic information about the object (such as a name). When you use the Kubernetes API to create the object (either directly or via kubectl), that API request must include that information as JSON in the request body. Most often, you provide the information to kubectl in a .yaml file. kubectl converts the information to JSON when making the API request.
    Here’s an example .yaml file that shows the required fields and object spec for a Kubernetes Deployment:
    
        apiVersion: apps/v1beta1
        kind: Deployment
        metadata:
          name: nginx-deployment
        spec:
          replicas: 3
          template:
            metadata:
              labels:
        	app: nginx
            spec:
              containers:
              - name: nginx
        	image: nginx:1.7.9
        	ports:
        	- containerPort: 80
    
    In the .yaml file for the Kubernetes object you want to create, you’ll need to set values for the following fields:
    
    -   *apiVersion* - Which version of the Kubernetes API you’re using to create this object
    -   *kind* - What kind of object you want to create
    -   *metadata* - Data that helps uniquely identify the object, including a name string, UID, and optional namespace
    
    You’ll also need to provide the object spec field. The precise format of the object spec is different for every Kubernetes object, and contains nested fields specific to that object. [The Kubernetes API reference](https://kubernetes.io/docs/concepts/overview/kubernetes-api/) can help you find the spec format for all of the objects you can create using Kubernetes. 


<a id="org94f4738"></a>

#### Pods

A Pod is the basic building block of Kubernetes – the smallest and simplest unit in the Kubernetes object model that you create or deploy. A Pod represents a running process on your cluster.
A Pod encapsulates an application container (or, in some cases, multiple containers), storage resources, a unique network IP, and options that govern how the container(s) should run. A Pod represents a unit of deployment: a single instance of an application in Kubernetes, which might consist of either a single container or a small number of containers that are tightly coupled and that share resources.
Docker is the most common container runtime used in a Kubernetes Pod, but Pods support other container runtimes as well.
Pods in a Kubernetes cluster can be used in two main ways:

-   *Pods that run a single container*. The “one-container-per-Pod” model is the most common Kubernetes use case; in this case, you can think of a Pod as a wrapper around a single container, and Kubernetes manages the Pods rather than the containers directly.
-   *Pods that run multiple containers that need to work together*. A Pod might encapsulate an application composed of multiple co-located containers that are tightly coupled and need to share resources. These co-located containers might form a single cohesive unit of service–one container serving files from a shared volume to the public, while a separate “sidecar” container refreshes or updates those files. The Pod wraps these containers and storage resources together as a single manageable entity.

1.  How Pods manage multiple Containers

     Pods are designed to support multiple cooperating processes (as containers) that form a cohesive unit of service. The containers in a Pod are automatically co-located and co-scheduled on the same physical or virtual machine in the cluster. The containers can share resources and dependencies, communicate with one another, and coordinate when and how they are terminated.
    Note that grouping multiple co-located and co-managed containers in a single Pod is a relatively advanced use case. You should use this pattern only in specific instances in which your containers are tightly coupled. For example, you might have a container that acts as a web server for files in a shared volume, and a separate “sidecar” container that updates those files from a remote source, as in the following diagram:
    
    Pods provide two kinds of shared resources for their constituent containers: networking and storage.
    
    1.  Networking
    
        Each Pod is assigned a unique IP address. Every container in a Pod shares the network namespace, including the IP address and network ports. Containers inside a Pod can communicate with one another using localhost. When containers in a Pod communicate with entities outside the Pod, they must coordinate how they use the shared network resources (such as ports). The *pause container* is used for this role, as when a pod starts it launches the pause container which acts as a pivot and takes the IP address for the pod and shares it with the rest of the containers in the pod though the network namespace.
    
    2.  Storage
    
        A Pod can specify a set of shared storage volumes. All containers in the Pod can access the shared volumes, allowing those containers to share data. Volumes also allow persistent data in a Pod to survive in case one of the containers within needs to be restarted. See Volumes for more information on how Kubernetes implements shared storage in a Pod.

2.  Motivation for pods

    1.  Management
    
        Pods are a model of the pattern of multiple cooperating processes which form a cohesive unit of service. They simplify application deployment and management by providing a higher-level abstraction than the set of their constituent applications. Pods serve as unit of deployment, horizontal scaling, and replication. Colocation (co-scheduling), shared fate (e.g. termination), coordinated replication, resource sharing, and dependency management are handled automatically for containers in a pod.
    
    2.  Resource sharing and communication
    
        Pods enable data sharing and communication among their constituents.
        The applications in a pod all use the same network namespace (same IP and port space), and can thus “find” each other and communicate using localhost. Because of this, applications in a pod must coordinate their usage of ports. Each pod has an IP address in a flat shared networking space that has full communication with other physical computers and pods across the network.
        The hostname is set to the pod’s Name for the application containers within the pod. In addition to defining the application containers that run in the pod, the pod specifies a set of shared storage volumes. Volumes enable data to survive container restarts and to be shared among the applications within the pod.

3.  Uses of pods

    Pods can be used to host vertically integrated application stacks (e.g. LAMP), but their primary motivation is to support co-located, co-managed helper programs, such as:
    
    -   content management systems, file and data loaders, local cache managers, etc.
    -   log and checkpoint backup, compression, rotation, snapshotting, etc.
    -   data change watchers, log tailers, logging and monitoring adapters, event publishers, etc.
    -   proxies, bridges, and adapters
    -   controllers, managers, configurators, and updaters
    
    Individual pods are not intended to run multiple instances of the same application, in general. For a longer explanation, see [The Distributed System ToolKit: Patterns for Composite Containers](http://blog.kubernetes.io/2015/06/the-distributed-system-toolkit-patterns.html).

4.  Pod Lifecycle

    A Pod’s status field is a PodStatus object, which has a phase field. The phase of a Pod is a simple, high-level summary of where the Pod is in its lifecycle. The phase is not intended to be a comprehensive rollup of observations of Container or Pod state, nor is it intended to be a comprehensive state machine.
    Here are the possible values for phase:
    
    -   Pending: The Pod has been accepted by the Kubernetes system, but one or more of the Container images has not been created. This includes time before being scheduled as well as time spent downloading images over the network, which could take a while.
    -   Running: The Pod has been bound to a node, and all of the Containers have been created. At least one Container is still running, or is in the process of starting or restarting.
    -   Succeeded: All Containers in the Pod have terminated in success, and will not be restarted.
    -   Failed: All Containers in the Pod have terminated, and at least one Container has terminated in failure. That is, the Container either exited with non-zero status or was terminated by the system.
    -   Unknown: For some reason the state of the Pod could not be obtained, typically due to an error in communicating with the host of the Pod.
    
    1.  Container probes
    
        A Probe is a diagnostic performed periodically by the kubelet on a Container. To perform a diagnostic, the kubelet calls a Handler implemented by the Container. There are three types of handlers:
        
        -   ExecAction: Executes a specified command inside the Container. The diagnostic is considered successful if the command exits with a status code of 0.
        -   TCPSocketAction: Performs a TCP check against the Container’s IP address on a specified port. The diagnostic is considered successful if the port is open.
        -   HTTPGetAction: Performs an HTTP Get request against the Container’s IP address on a specified port and path. The diagnostic is considered successful if the response has a status code greater than or equal to 200 and less than 400.
        
        Each probe has one of three results:
        
        -   Success: The Container passed the diagnostic.
        -   Failure: The Container failed the diagnostic.
        -   Unknown: The diagnostic failed, so no action should be taken.
        
        The kubelet can optionally perform and react to two kinds of probes on running Containers:
        
        -   livenessProbe: Indicates whether the Container is running. If the liveness probe fails, the kubelet kills the Container, and the Container is subjected to its restart policy. If a Container does not provide a liveness probe, the default state is Success.
        -   readinessProbe: Indicates whether the Container is ready to service requests. If the readiness probe fails, the endpoints controller removes the Pod’s IP address from the endpoints of all Services that match the Pod. The default state of readiness before the initial delay is Failure. If a Container does not provide a readiness probe, the default state is Success.
        
        1.  Using liveness or readiness probes
        
               If the process in your Container is able to crash on its own whenever it encounters an issue or becomes unhealthy, you do not necessarily need a liveness probe; the kubelet will automatically perform the correct action in accordance with the Pod’s restartPolicy.
            If you’d like your Container to be killed and restarted if a probe fails, then specify a liveness probe, and specify a restartPolicy of Always or OnFailure.
            If you’d like to start sending traffic to a Pod only when a probe succeeds, specify a readiness probe. In this case, the readiness probe might be the same as the liveness probe, but the existence of the readiness probe in the spec means that the Pod will start without receiving any traffic and only start receiving traffic after the probe starts succeeding.
            If you want your Container to be able to take itself down for maintenance, you can specify a readiness probe that checks an endpoint specific to readiness that is different from the liveness probe.
            Note that if you just want to be able to drain requests when the Pod is deleted, you do not necessarily need a readiness probe; on deletion, the Pod automatically puts itself into an unready state regardless of whether the readiness probe exists. The Pod remains in the unready state while it waits for the Containers in the Pod to stop.

5.  Pod lifetime

    In general, Pods do not disappear until someone destroys them. This might be a human or a controller. The only exception to this rule is that Pods with a phase of Succeeded or Failed for more than some duration (determined by the master) will expire and be automatically destroyed.
    Three types of controllers are available:
    
    -   Use a Job for Pods that are expected to terminate, for example, batch computations. Jobs are appropriate only for Pods with restartPolicy equal to OnFailure or Never.
    -   Use a ReplicationController, ReplicaSet, or Deployment for Pods that are not expected to terminate, for example, web servers. ReplicationControllers are appropriate only for Pods with a restartPolicy of Always.
    -   Use a DaemonSet for Pods that need to run one per machine, because they provide a machine-specific system service.
    
    All three types of controllers contain a PodTemplate. It is recommended to create the appropriate controller and let it create Pods, rather than directly create Pods yourself. That is because Pods alone are not resilient to machine failures, but controllers are.
    If a node dies or is disconnected from the rest of the cluster, Kubernetes applies a policy for setting the phase of all Pods on the lost node to Failed.


<a id="org43841fb"></a>

#### Services

Kubernetes Pods are mortal. They are born and when they die, they are not resurrected. ReplicationControllers in particular create and destroy Pods dynamically (e.g. when scaling up or down or when doing rolling updates). While each Pod gets its own IP address, even those IP addresses cannot be relied upon to be stable over time. This leads to a problem: if some set of Pods (let’s call them backends) provides functionality to other Pods (let’s call them frontends) inside the Kubernetes cluster, how do those frontends find out and keep track of which backends are in that set?
Enter Services.
A Kubernetes Service is an abstraction which defines a logical set of Pods and a policy by which to access them - sometimes called a micro-service. The set of Pods targeted by a Service is (usually) determined by a Label Selector (see below for why you might want a Service without a selector).
As an example, consider an image-processing backend which is running with 3 replicas. Those replicas are fungible - frontends do not care which backend they use. While the actual Pods that compose the backend set may change, the frontend clients should not need to be aware of that or keep track of the list of backends themselves. The Service abstraction enables this decoupling.
For Kubernetes-native applications, Kubernetes offers a simple Endpoints API that is updated whenever the set of Pods in a Service changes. For non-native applications, Kubernetes offers a virtual-IP-based bridge to Services which redirects to the backend Pods.

1.  Defining a service

    A Service in Kubernetes is a REST object, similar to a Pod. Like all of the REST objects, a Service definition can be POSTed to the apiserver to create a new instance. For example, suppose you have a set of Pods that each expose port 9376 and carry a label "app=MyApp".
    
        kind: Service
        apiVersion: v1
        metadata:
          name: my-service
        spec:
          selector:
            app: MyApp
          ports:
          - protocol: TCP
            port: 80
            targetPort: 9376
    
    This specification will create a new Service object named “my-service” which targets TCP port 9376 on any Pod with the "app=MyApp" label. This Service will also be assigned an IP address (sometimes called the “cluster IP”), which is used by the service proxies (see below). The Service’s selector will be evaluated continuously and the results will be POSTed to an Endpoints object also named “my-service”.
    Note that a Service can map an incoming port to any targetPort. By default the targetPort will be set to the same value as the port field. Perhaps more interesting is that targetPort can be a string, referring to the name of a port in the backend Pods. The actual port number assigned to that name can be different in each backend Pod. This offers a lot of flexibility for deploying and evolving your Services. For example, you can change the port number that pods expose in the next version of your backend software, without breaking clients.
    Kubernetes Services support TCP and UDP for protocols. The default is TCP
    
    1.  Services without selectors
    
        Services generally abstract access to Kubernetes Pods, but they can also abstract other kinds of backends. For example:
        
        -   You want to have an external database cluster in production, but in test you use your own databases.
        -   You want to point your service to a service in another Namespace or on another cluster.
        -   You are migrating your workload to Kubernetes and some of your backends run outside of Kubernetes.
        
        In any of these scenarios you can define a service without a selector:
        
            kind: Service
            apiVersion: v1
            metadata:
              name: my-service
            spec:
              ports:
              - protocol: TCP
                port: 80
                targetPort: 9376
        
        Because this service has no selector, the corresponding Endpoints object will not be created. You can manually map the service to your own specific endpoints:
        
            kind: Endpoints
            apiVersion: v1
            metadata:
              name: my-service
            subsets:
              - addresses:
                  - ip: 1.2.3.4
                ports:
                  - port: 9376
        
        NOTE: Endpoint IPs may not be loopback (127.0.0.0/8), link-local (169.254.0.0/16), or link-local multicast (224.0.0.0/24).
        Accessing a Service without a selector works the same as if it had a selector. The traffic will be routed to endpoints defined by the user (1.2.3.4:9376 in this example).
        An ExternalName service is a special case of service that does not have selectors. It does not define any ports or endpoints. Rather, it serves as a way to return an alias to an external service residing outside the cluster.
        
            kind: Service
            apiVersion: v1
            metadata:
              name: my-service
              namespace: prod
            spec:
              type: ExternalName
              externalName: my.database.example.com
        
        When looking up the host *my-service.prod.svc.CLUSTER*, the cluster DNS service will return a CNAME record with the value *my.database.example.com*. Accessing such a service works in the same way as others, with the only difference that the redirection happens at the DNS level and no proxying or forwarding occurs. Should you later decide to move your database into your cluster, you can start its pods, add appropriate selectors or endpoints and change the service type.


<a id="org31435ef"></a>

#### Controllers

1.  Replica Sets

    ReplicaSet is the next-generation Replication Controller. The only difference between a ReplicaSet and a Replication Controller right now is the selector support. ReplicaSet supports the new set-based selector requirements as described in the [labels user guide](https://kubernetes.io/docs/concepts/overview/working-with-objects/labels/#label-selectors) whereas a Replication Controller only supports equality-based selector requirements.
    
    1.  How to use a ReplicaSet
    
              Most kubectl commands that support Replication Controllers also support ReplicaSets. One exception is the rolling-update command. If you want the rolling update functionality please consider using Deployments instead. Also, the rolling-update command is imperative whereas Deployments are declarative, so we recommend using Deployments through the rollout command.
        While ReplicaSets can be used independently, today it’s mainly used by Deployments as a mechanism to orchestrate pod creation, deletion and updates. When you use Deployments you don’t have to worry about managing the ReplicaSets that they create. Deployments own and manage their ReplicaSets.
    
    2.  When to use a ReplicaSet
    
              A ReplicaSet ensures that a specified number of pod replicas are running at any given time. However, a Deployment is a higher-level concept that manages ReplicaSets and provides declarative updates to pods along with a lot of other useful features. Therefore, we recommend using Deployments instead of directly using ReplicaSets, unless you require custom update orchestration or don’t require updates at all.
        This actually means that you may never need to manipulate ReplicaSet objects: use a Deployment instead, and define your application in the spec section.
            Example:
        
            apiVersion: apps/v1beta2 # for versions before 1.6.0 use extensions/v1beta1
            kind: ReplicaSet
            metadata:
              name: frontend
              labels:
                app: guestbook
                tier: frontend
            spec:
              # this replicas value is default
              # modify it according to your case
              replicas: 3
              selector:
                matchLabels:
                  tier: frontend
                matchExpressions:
                  - {key: tier, operator: In, values: [frontend]}
              template:
                metadata:
                  labels:
            	app: guestbook
            	tier: frontend
                spec:
                  containers:
                  - name: php-redis
            	image: gcr.io/google_samples/gb-frontend:v3
            	resources:
            	  requests:
            	    cpu: 100m
            	    memory: 100Mi
            	env:
            	- name: GET_HOSTS_FROM
            	  value: dns
            	  # If your cluster config does not include a dns service, then to
            	  # instead access environment variables to find service host
            	  # info, comment out the 'value: dns' line above, and uncomment the
            	  # line below.
            	  # value: env
            	ports:
            	- containerPort: 80
        
        For more details on replica sets read the documentation available at: [replicaset](https://kubernetes.io/docs/concepts/workloads/controllers/replicaset/).

2.  Replication Controller

    **NOTE**: A Deployment that configures a ReplicaSet is now the recommended way to set up replication.
    
    A ReplicationController ensures that a specified number of pod replicas are running at any one time. In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available. 
    Details on how replication controllers work are available [here](https://kubernetes.io/docs/concepts/workloads/controllers/replicationcontroller/).

3.  Deployments

    A Deployment controller provides declarative updates for Pods and ReplicaSets. This is the most widely used object in kubernetes and it's also used by OCM to deploy optymyze.
    You describe a desired state in a Deployment object, and the Deployment controller changes the actual state to the desired state at a controlled rate. You can define Deployments to create new ReplicaSets, or to remove existing Deployments and adopt all their resources with new Deployments.
    The following are typical use cases for Deployments:
    
    -   Create a Deployment to rollout a ReplicaSet. The ReplicaSet creates Pods in the background. Check the status of the rollout to see if it succeeds or not.
    -   Declare the new state of the Pods by updating the PodTemplateSpec of the Deployment. A new ReplicaSet is created and the Deployment manages moving the Pods from the old ReplicaSet to the new one at a controlled rate. Each new ReplicaSet updates the revision of the Deployment.
    -   Rollback to an earlier Deployment revision if the current state of the Deployment is not stable. Each rollback updates the revision of the Deployment.
    -   Scale up the Deployment to facilitate more load.
    -   Pause the Deployment to apply multiple fixes to its PodTemplateSpec and then resume it to start a new rollout.
    -   Use the status of the Deployment as an indicator that a rollout has stuck.
    -   Clean up older ReplicaSets that you don’t need anymore.
    
    For details on how to create, update and use deployments see the official documentation available at: [kubernetes deployments](https://kubernetes.io/docs/concepts/workloads/controllers/deployment/). 

4.  StatefulSets

    StatefulSet is the workload API object used to manage stateful applications. StatefulSets are beta in 1.8.
    Manage the deployment and scaling of a set of Pods, and provide guarantees about ordering. They do so by maintaining a unique, sticky identity for each of their Pods.
    Like Deployments, StatefulSets manage Pods that are based on an identical container spec. However, although their specs are the same, the Pods in a StatefulSet are not interchangeable. Each Pod has a persistent identifier that it maintains across any rescheduling.
    StatefulSets also operate according to the Controller pattern. You define your desired state in a StatefulSet object, and the StatefulSet controller makes any necessary updates to the get there from the current state.
    
    1.  Using StatefulSets
    
        StatefulSets are valuable for applications that require one or more of the following.
        
        -   Stable, unique network identifiers.
        -   Stable, persistent storage.
        -   Ordered, graceful deployment and scaling.
        -   Ordered, graceful deletion and termination.
        -   Ordered, automated rolling updates.
        
        In the above, stable is synonymous with persistence across Pod (re)scheduling. If an application doesn’t require any stable identifiers or ordered deployment, deletion, or scaling, you should deploy your application with a controller that provides a set of stateless replicas. Controllers such as Deployment or ReplicaSet may be better suited to your stateless needs.
    
    2.  Limitations
    
        -   StatefulSet is a beta resource, not available in any Kubernetes release prior to 1.5.
        -   The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner based on the requested storage class, or pre-provisioned by an admin.
        -   Deleting and/or scaling a StatefulSet down will not delete the volumes associated with the StatefulSet. This is done to ensure data safety, which is generally more valuable than an automatic purge of all related StatefulSet resources.
        -   StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods. You are responsible for creating this Service.
        
        More details on statefull sets are available [here](https://kubernetes.io/docs/concepts/workloads/controllers/statefulset/).

5.  Daemon Sets

    A DaemonSet ensures that all (or some) Nodes run a copy of a Pod. As nodes are added to the cluster, Pods are added to them. As nodes are removed from the cluster, those Pods are garbage collected. Deleting a DaemonSet will clean up the Pods it created.
    Some typical uses of a DaemonSet are:
    
    -   running a cluster storage daemon, such as glusterd, ceph, on each node.
    -   running a logs collection daemon on every node, such as fluentd or logstash.
    -   running a node monitoring daemon on every node, such as Prometheus Node Exporter, collectd, Datadog agent, New Relic agent, or Ganglia gmond.
    
    In a simple case, one DaemonSet, covering all nodes, would be used for each type of daemon. A more complex setup might use multiple DaemonSets for a single type of daemon, but with different flags and/or different memory and cpu requests for different hardware types.
    Details about daemon sets are available [here](https://kubernetes.io/docs/concepts/workloads/controllers/daemonset/).

6.  Garbage Collection

     The role of the Kubernetes garbage collector is to delete certain objects that once had an owner, but no longer have an owner.
    Some Kubernetes objects are owners of other objects. For example, a ReplicaSet is the owner of a set of Pods. The owned objects are called dependents of the owner object. Every dependent object has a `metadata.ownerReferences` field that points to the owning object.
    Sometimes, Kubernetes sets the value of ownerReference automatically. For example, when you create a ReplicaSet, Kubernetes automatically sets the ownerReference field of each Pod in the ReplicaSet. In 1.6, Kubernetes automatically sets the value of ownerReference for objects created or adopted by ReplicationController, ReplicaSet, StatefulSet, DaemonSet, and Deployment.
    
    1.  Controlling how the garbage collector deletes dependents
    
        When you delete an object, you can specify whether the object’s dependents are also deleted automatically. Deleting dependents automatically is called cascading deletion. There are two modes of cascading deletion: background and foreground.
        If you delete an object without deleting its dependents automatically, the dependents are said to be orphaned.
        
        1.  Background cascading deletion
        
            In background cascading deletion, Kubernetes deletes the owner object immediately and the garbage collector then deletes the dependents in the background.
        
        2.  Foreground cascading deletion
        
            In foreground cascading deletion, the root object first enters a “deletion in progress” state. In the “deletion in progress” state, the following things are true:
            
            -   The object is still visible via the REST API
            -   The object’s deletionTimestamp is set
            -   The object’s metadata.finalizers contains the value “foregroundDeletion”.
            
            Once the “deletion in progress” state is set, the garbage collector deletes the object’s dependents. Once the garbage collector has deleted all “blocking” dependents (objects with ownerReference.blockOwnerDeletion=true), it delete the owner object.
            Note that in the “foregroundDeletion”, only dependents with ownerReference.blockOwnerDeletion block the deletion of the owner object. Kubernetes version 1.7 will add an admission controller that controls user access to set blockOwnerDeletion to true based on delete permissions on the owner object, so that unauthorized dependents cannot delay deletion of an owner object.
            If an object’s ownerReferences field is set by a controller (such as Deployment or ReplicaSet), blockOwnerDeletion is set automatically and you do not need to manually modify this field.
        
        3.  Setting the cascading deletion policy
        
            To control the cascading deletion policy, set the deleteOptions.propagationPolicy field on your owner object. Possible values include “Orphan”, “Foreground”, or “Background”.
            The default garbage collection policy for many controller resources is orphan, including ReplicationController, ReplicaSet, StatefulSet, DaemonSet, and Deployment. So unless you specify otherwise, dependent objects are orphaned.
            
            A more in-depth description of the garbage collection resouce and how it works is available [here](https://kubernetes.io/docs/concepts/workloads/controllers/garbage-collection/).

7.  Jobs - Run to Completion

    A job creates one or more pods and ensures that a specified number of them successfully terminate. As pods successfully complete, the job tracks the successful completions. When a specified number of successful completions is reached, the job itself is complete. Deleting a Job will cleanup the pods it created.
    A simple case is to create one Job object in order to reliably run one Pod to completion. The Job object will start a new Pod if the first pod fails or is deleted (for example due to a node hardware failure or a node reboot).
    A Job can also be used to run multiple pods in parallel.
    Here is an example Job config. It computes π to 2000 places and prints it out. It takes around 10s to complete:
    
        apiVersion: batch/v1
        kind: Job
        metadata:
          name: pi
        spec:
          template:
            metadata:
              name: pi
            spec:
              containers:
              - name: pi
        	image: perl
        	command: ["perl",  "-Mbignum=bpi", "-wle", "print bpi(2000)"]
              restartPolicy: Never
              backoffLimit: 4
    
    The "Job" is a very powerfull concept and can have lot's of applications. To learn more about jobs see the documentation available [here](https://kubernetes.io/docs/concepts/workloads/controllers/jobs-run-to-completion/).

8.  Cron Jobs

    A Cron Job manages time based Jobs, namely:
    
    -   Once at a specified point in time
    -   Repeatedly at a specified point in time
    -   One CronJob object is like one line of a crontab (cron table) file. It runs a job periodically on a given schedule, written in Cron format.
    
    Note: CronJob resource in batch/v2alpha1 API group has been deprecated starting from cluster version 1.8. You should switch to using batch/v1beta1, instead, which is enabled by default in the API server. Further in this document, we will be using batch/v1beta1 in all the examples.
    A typical use case is:
    
    -   Schedule a job execution at a given point in time.
    -   Create a periodic job, e.g. database backup, sending emails.
    
    More details about cron jobs are available at [cron-jobs](https://kubernetes.io/docs/concepts/workloads/controllers/cron-jobs/).


<a id="orga2b6eeb"></a>



# Resources
* Large-scale cluster management at Google with Borg - https://research.google.com/pubs/pub43438.html
* Apache Hadoop YARN: Yet Another Resource Negotiator - https://54e57bc8-a-62cb3a1a-s-sites.googlegroups.com/site/2013socc/home/program/a5-vavilapalli.pdf?attachauth=ANoY7cqMTFYK5q_YIaY8_NQe2QKXqYukFk0sJVmfjVDMzRWtZk1qPKL7KDGxWErVD2gVPa1KvbMfuVy-vH8r5-srCnIbQUWj1vaUY4IKizAcuKG5sM9lpRXlR8yWldsGmZTsm1MN3GXVRfgRBVMXzfw8AtyUJ18n-xW_TDaT4r2ntxZ1s7TW4yEugNtJqYH2dJpqtapqk5JnGXePAB-JCikcnpdpo3v8_T3fqxsIKC1cbIruSGEt04U%3D&attredirects=0
*Efficient Datacenter Resource Utilization Through Cloud Resource Overcommitment http://web.engr.orst.edu/~hamdaoui/papers/2015/mehiar-infocom-15.pdf
