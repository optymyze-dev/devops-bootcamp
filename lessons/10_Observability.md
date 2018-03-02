# Observability

## What is observability

The observability is the quality of a system that offers information about its internals.

Why we need observability for our systems? We want to prevent failures as much as possible. 
It means the systems offer us a mechanism to observe its behavior. 

You can't monitor systems that don't offer observability (that are not 'monitorable'). Observability is a quality of systems but monitoring is our action. 

In general, we tend to measure three levels:

	* network
	* machine
	* application

We do that obtaining and processing specific values. Actually, we evaluate the internal of each level based on these output	values.

	
## Monitoring

A monitoring system should address two questions: What's broken and Why?
The answer to the first question is given by the values we receive but the answer to the second means processing of those values.

Ex: We receive streams of metrics for cpu, free memory, free disk space etc. When free memory is below a certain value we can experience problems with running applications. We have to determine why. 


### The pull vs push problem

There are two ways to collect metrics, the pull way and the push way.
The pull way means the monitoring system will fetch metrics and the push way means the monitored systems will push their metrics to the monitoring system.


#### Riemann

[Riemann](http://riemann.io/concepts.html) is a monitoring server that uses the push way. Each monitored system should push its metrics to the server. 
The Riemann server doesn't know about other systems. It just receives metrics and process them based on some rules. It decides to send alerts based on its processing.
Every system that wants to be monitored has to know about the Riemann server and send metrics in the right format (in the Riemann way).

#### Prometheus

[Prometheus](https://prometheus.io/docs/introduction/overview/) works the opposite way than Riemann. It uses the pull way. It is configured to know about the systems that have to be monitored and periodically will fetch metrics from them.
Obviously, the monitored systems should be up and available but this is not happening all the time. There are short time tasks (crons) that do their job then stop. 
For this case Prometheus provides a Push Gateway server that is live all the time. In this scenario, the cron job will do its job and build its metrics. It will push its metrics before stop.

## Logging

Logging is the action of saving time series information about what systems do.

The developers decide what information should be logged.
Logs can contain:

	* tracing information
	* debugging information
	* errors
	* warnings	

### The importance of logging

Logs are a critical part of any system. They give you an image about what happens in your system. 

Logs are important for sysadmins and developers. 
Sysdmins check logs to make sure the system is working as expected. Sometimes we cannot observe malfunction of a process but the process can log its status.

Developers write code and they need to find problems in their applications during and after development. Developer has no access on the client machine but can receive (see centralization of logs) the logs of the application and determine its behavior in certain situations.
Usually, applications have a log level that can be set. In debug level the application can log very detailed information. 

The operating system and application generates log files on local disks. In enterprise applications, when we use many systems, it is difficult nu analyze and access each log file in order to find some information. 
The solution is to use some tools that can aggregate and centralize the logs.


### Centralization of logs

The idea of centralization of logs needs to address some problems:

	* logs collection and aggregation
	* logs transport
	* logs storage
	* logs analysis (and alerting)

#### Collection	

Logs collection in a central location can be realized using a replication strategy. You can setup a cron job that periodically will rsync distributed log files to a central location. 
Here, we can analyze the logs. The accuracy depends on how fast this replication strategy works.


#### Transport 

Centralization of logs requires to transport the logs from the place they are generated to the central location. There are tools that generate logs and know how to send the logs to a central location.
Also, there are applications that generates logs in files and we need other tools that transport them to a central location. These tools are designed to transport large volume of logs.


#### Storage
	
The logs central location need to be able to store a large volume that can grow fast in time. The solution to this problem depends on what we want to do with the logs. 
If we want to store the logs for long time then we choose a solution to archive them but not immediate analysis. Here, the volume size is more important than the access time.
If we want immediate analysis of the logs we need a solution that offer real-time access and batch analysis. 	

For example, storing log data in ElasticSearch allow us to work with row data more effectively. ElasticSearch analyzes large data in batch pattern.
	
#### Analysis and Alerting

The main purpose of centralization of logs is their analysis. There are tools that process log data periodically and store results that can be query later.
Further, some front-end application (Kibana) can be used to display the results.

Some important log data are errors. Errors mean something went wrong in your system. You need to know as soon as possible when an error occurs.
There are tools that can be configured to alert or notify a group of persons for certain errors.

#### Tools

	* [elasticsearch](https://www.elastic.co/)
	* [fluentd](https://www.fluentd.org/)
	* [ok-log](https://github.com/oklog/oklog)

## Tracing

### What is a trace

Tracing is used to measure the internals of an application. For example, when an user makes a request then an entire stack of code is executed in order to create the response.
We can instrument the code and collect metrics about the execution of each level. At the end, these metrics are pushed to a tracing server.

Ideally, the tracing should not add complexity to the code and some times the instrumentation code is included in libraries, therefore it is transparent to the developer.

### The importance of tracing

An application supports several updates/upgrades over the time. We need to know if a code change cause changes in execution time. If we traced some data before upgrade we can compare it with new traced data.

Also, we want to know how our system responds when multiple users concurrently access our system. We can trace data and gradually increase the load of the application. We can see how our system responds in this situation. 

### Tools

	* [zipkin](https://zipkin.io/)
	* [jaeger](https://jaeger.readthedocs.io/en/latest/)
	* [opentracing](http://opentracing.io/) 


