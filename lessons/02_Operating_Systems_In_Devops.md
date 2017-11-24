# Operating systems as the foundation of Devops stack
This lesson tries to explore the convergence between operating systems and the devops movement.

## Table of Contents
* [Operating systems and the rise of microservices](#operating-systems)
    * [What is the purpose of an operating system](#purpose-of-an-operating-system)
    * [Microservices](#microservices)
* [Dependency Management And Provisioning](#dependency-management-and-provisioning)
    * [Package Managers](#package-managers)
    * [Orchestration and provisioning tools](#orchestration-and-provisioning-tools)
* [Delivery of services](#delivery-of-services)
    * [IaaS](#infrastructure-as-a-service)
    * [PaaS](#platform-as-a-service)
    * [Saas](#software-as-a-service)
* [Abstracting Resources](#abstracting-resources)
    * [Virtual machines](#virtual-machines)
    * [Containers](#containers)
    * [Container oriented operating systems](#container-oriented-operating-systems)
* [Serverless computing](#serverless-computing)
    * [Dynamic allocation of resources](#dynamic-allocation-of-resources)
    * [Advantages and disadvantages](#advantages-and-disadvantages)
* [Documentation](#resources-from-the-internet)


## Operating systems
### Purpose of an operating system
An operation system is a system software that manages computer hardware and software
resources and provides common services for computer programs. 
The main features required of an operating system are:
* Memory management
* Process management
* Interrupts
* Device drivers
* File system and I/O management
* Networking
* Security
### Microservices

#### What is a microservice

Microservices represent an architectural style of development that
structures the application as a collection of loosely coupled services.
 
https://martinfowler.com/articles/microservices.html

#### Characteristics of a microservice
1. Componentization via Services

A component is a unit of software that is independently replaceable 
and upgradeable
2. Organized around business capabilities

Any organization that designs a system (defined broadly) will produce a design whose structure is a copy of the organization's communication structure.

-- Melvyn Conway, 1967
3. Products not projects

This mentality brings power to the developers and makes them in charge of 
all the product lifecycle. Amazon are famous for this approach :
You build it, you run it!
4. Smart endpoints and dumb pipes

This brings a major difference from SOA, making the transport and communications
more coarse-grained from the ESB approaches, leaving the logic simple
and standard for the transport layer
5. Decentralized governance

This encourages the asynchronous calls between services and makes
the services more independent from each other
6. Decentralized data management

Transaction are hard and distributed transactions are even worse,
so new approaches have been developed making room for the eventual
consistency to be used.
There is a theorem in theoretical computer science called 
CAP theory that in a distributed system  you can not achieve all of the 
following:
* Consistency
* Availability
* Partition tolerance 
7. Infrastructure automation

This characteristic is best described by the Amazon principle:
Build it fast and fail fast!
In order to create agile software, the services must be developed,
tested and deployed as fast as possible and retrieved as fast if 
anything fails. This has brought the continuous integration and
continuous delivery movements to be a part of the software
development and project management.
8. Design for failure

This a natural response to the distributed nature of the microservices.
For a better understanding of the distributed system needs, take
a look at [Distributed systems fallacies](#https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing)
9. Evolutionary design
Any piece of software is meant to be replaced as the code deprecates in time.
This approach encourages for small iterations in adding features
and keeping it simple, as the needs of the users will change.

'Premature optimization is the root of all evil.' - Donald Knuth
 
## Dependency Management And Provisioning
Once the code is modularized and dependencies are created in order
to be reused, the need to organize and manage them becomes more and more
complex and can bring a lot of problems if not handled properly
https://en.wikipedia.org/wiki/DLL_Hell

### Package Managers
* Install, upgrade and remove software automatically
* Resolve package dependencies
* Use central trusted repositories
* Can search information on installed packages and files

#### Linux system package managers 
Main formats are 
1. rpm -  rpm, yum 
2. deb -  dpkg, apt

#### Windows package manager 
* Chocolatey
* OneGet

#### Programming Language Package Managers
* pip - python
* gem/rubygems - ruby
* npm - Nodejs

### Orchestration and provisioning tools
http://codefol.io/posts/deployment-versus-provisioning-versus-orchestration
https://devopscube.com/devops-tools-for-infrastructure-automation/

## Delivery of services
Once the need for many machines to serve as nodes in the distributed
system arised, manual management of a datacenter become a high cost and new solutions appeared.
 
### Infrastructure as a service

Infrastructure as a service (IaaS) refers to online services that provide 
high-level APIs used to dereference various low-level details of underlying 
network infrastructure like physical computing resources, location, 
data partitioning, scaling, security, backup.
Although the advantages can be really easy to seduce a company to 
use Iaas, the are things to take into consideration:
https://www.itworld.com/article/3237168/cloud-computing/10-cloud-mistakes-that-can-sink-your-business.html
 
### Platform as a service

Platform as a Service (PaaS) is a category of cloud computing services that 
provides a platform allowing customers to develop, run, and manage applications 
without the complexity of building and maintaining the 
infrastructure typically associated with developing and launching an app.

### Software as a service
Software as a service is a software licensing and delivery model in 
which software is licensed on a subscription basis and is 
centrally hosted.


## Abstracting Resources
### Virtual machines

https://en.wikipedia.org/wiki/Virtual_machine

### Containers

https://www.docker.com/what-container

### Container oriented operating systems
https://blog.codeship.com/container-os-comparison/

## Serverless computing
### Dynamic allocation of resources
### Advantages and disadvantages

## Resources from the internet
http://homepage.cs.uri.edu/~thenry/resources/unix_art/ch01s06.html
https://devopsbootcamp.osuosl.org/operating-systems.html
https://cs.uwec.edu/~buipj/teaching/cs.388.s14/static/pdf/upe.pdf