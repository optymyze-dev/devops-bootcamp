# Public Cloud Providers
This lesson will focus on public cloud providers. We will discuss about Google Cloud, Amazon Web Services and Microsoft Azure, since they are the top 3 public cloud providers.

##  What is a public cloud

The public cloud is defined as computing services offered by third-party providers over the public Internet, making them available to anyone who wants to use or purchase them. They may be free or sold on-demand, allowing customers to pay only per usage for the CPU cycles, storage, or bandwidth they consume.

*You can think of them as other people's servers*

Cloud Computing is moving into 3 separate distinct directions:

* Public Cloud Computing – like Amazon’s EC2 and Google Cloud, designed to provide the lowest cost shared computing service with a utility delivery model where you can pay for services by the hour.

* Private Cloud Computing – the opposite end of the spectrum where the entire cloud infrastructure (servers, storage and network) is dedicated to a single company.

* Managed Cloud Computing – where a secure private cloud is sold by the server.

### The logic behind the Public Cloud Providers 
These go back to the basic of economics:

How does one increase productivity? Why do nations focus producing certain goods over others and then trade? These questions are addressed in the discussion of specialization in economics.
Definition of Specialization

Specialization is when a nation or individual concentrates its productive efforts on producing a limited variety of goods. It oftentimes has to forgo producing other goods and relies on obtaining those other goods through trade.

#### Division of Labor

Adam Smith was one of the first economists to explain the benefits one gets from specialization. He focused on describing the benefits of individuals specializing in labor. In An Inquiry into the Nature and Causes of the Wealth of Nations (1776), he describes the benefits of one type of specialization, division of labor, which is when cooperating individuals perform specialized tasks. Smith discusses this type of specialization in the context of a pin factory.

Smith explains that if the pin factory had each person making pins from start to finish, they may be able to make upward of 20 pins each a day. Now, consider if the pin-making process was divided into distinct operations, and each person were able to specialize in a particular operation in the pin-making process. He posits that an entire factory of 10 workers could produce 48,000 pins a day. With a factory of 10 workers, that would equate to 4,800 pins per person per day compared to the initial 20 pins per day without division of labor.

Here is an example of specialization - the assembly line. Each person specializes in one distinct operational function, thus improving the efficiency of production overall. We see this type of production frequently at cafes, restaurants, etc.


#### Opportunity Cost

David Ricardo addresses specialization on a global, macroeconomic level to explain why countries specialize and enter into trade. Ricardo's justification is based on the concept of opportunity cost. Opportunity cost is the cost of the next best alternative, or what you are giving up to do what you are currently doing. Ricardo explains why opportunity cost is an incentive for rational people and nations to specialize and enter into trade. This concept can be illustrated with a simple example.

Consider the situation where you and I are stranded in the wilderness. I can either pick 100 berries or catch 5 fish. You can either pick 50 berries or catch 10 fish. For me to catch 5 fish, it will cost me 100 berries. Therefore, the opportunity cost of each fish is 20 berries. For me to pick 100 berries, I have to forgo 5 fish. The opportunity cost of each berry is 1/20 fish.

Following the same logic, your opportunity cost of each fish is 5 berries and of each berry is 1/5 fish. I have a lower opportunity cost for berries, 1/20 fish compared to your 1/5 fish; therefore, I have a comparative advantage in picking berries.

#### Cloud providers manage servers better than you

The reality is that your company will need to specialize to be competitive in the global market. Since we're talking about software your company will need to specialize in software and let others specialize in hardware. Public cloud providers are specialized in hardware.
	
## Characteristics of the major cloud providers

All the major cloud providers share some common features:

* They allow for provisioning of capacity on demand
* They provide an API to interact with their services
* They provide a multitude of services but at least they all provide
  * computing services
	* virtual machines
	* bare metal servers
	* containers
  * disk services
	* object store
	* block devices
  * network services
	* isolation
* They run all of their services multi-tenant
* Custom services
  * database services
	* relational databases
	* NOSQL
  * machine learning
  * security and identity
  
## Cloud Providers and locations

Each cloud provider offers multiple locations, each location with it's own Availability Zones. 

Each region is wholly contained within a single country and all of its data and services stay within the designated region. Each region has multiple "Availability Zones", which consist of one or more discrete data centers, each with redundant power, networking and connectivity, housed in separate facilities. Availability Zones do not automatically provide additional scalability or redundancy within a region, since they are intentionally isolated from each other to prevent outages from spreading between Zones. Several services can operate across Availability Zones  while others can be configured to replicate across Zones to spread demand and avoid downtime from failures.

For example:
As of December 2014, Amazon Web Services operated an estimated 1.4 Million servers across 28 availability zones. The global network of AWS Edge locations consists of 54 points of presence worldwide, including locations in the United States, Europe, Asia, Australia, and South America.

## Compliance and security

When dealing with public clouds one of the main pain points of companies is security and compliance. Companies depending on their business type need to implement certain security standards. For example companies that deal with credit cards need to have the PCI-DSA standard. Also companies in health at least in the US need to implement HIPPA. These standards describe a series of controls and evidence to demonstrate to an external auditor that you're following these best practices. 

It's difficult for some companies to implement these controls and collect the necessary evidence when dealing in public cloud environments. Most of the cloud providers create a list of security standards each of these services implement so based on the domain each company needs to think very careful about the migration to a public cloud and what type of services the can use. 

## Service Outages


Significant service outages at AWS:

* On April 20, 2011, AWS suffered a major outage. Parts of the Elastic Block Store (EBS) service became "stuck" and could not fulfill read/write requests. It took at least two days for service to be fully restored.

* On June 29, 2012, several websites that rely on Amazon Web Services were taken offline due to a severe storm in Northern Virginia, where AWS' largest data center cluster is located.

* On October 22, 2012, a major outage occurred, affecting many sites such as Reddit, Foursquare, Pinterest, and others. The cause was a memory leak bug in an operational data collection agent.

* On December 24, 2012, AWS suffered another outage causing websites such as Netflix to be unavailable for customers in the Northeastern United States. AWS cited their Elastic Load Balancing (ELB) service as the cause.

* On February 28, 2017, AWS experienced a massive outage of S3 services in its Northern Virginia data center. A majority of websites which relied on AWS S3 either hung or stalled, and Amazon reported within five hours that AWS was fully online again. No data has been reported to have been lost due to the outage. The outage was caused by a human error made while debugging, that resulted in removing more server capacity than intended, which caused a domino effect of outages.

AWS currently has the best track record.

## Multi-Cloud Deployments

When moving all your assets to a public cloud provider you're basically dependent on them. Several big companies are now advocating a multi-cloud deployment model, where some of your most critical assets are replicated to more than one cloud provider (for example Netflix uses both AWS and Google Cloud). 

This is basically a balancing act as it increases your operational cost and complexity for events which might haven once every couple of years. Still when you're business relies on AWS for example it's best to at least move you're backups to more than one cloud, just in case.
