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


### Riemann

Riemann is a monitoring server that uses the push way. Each monitored system should push its metrics to the server. 
The Riemann server doesn't know about other systems. It just receives metrics and process them based on some rules. It decides to send alerts based on its processing.
Every system that wants to be monitored has to know about the Riemann server and send metrics in the right format (in the Riemann way).

### Prometheus

Prometheus works the opposite way than Riemann. It uses the pull way. It is configured to know about the systems that have to be monitored and periodically will fetch metrics from them.
Obviously, the monitored systems should be up and available but this is not happening all the time. There are short time tasks (crons) that do their job then stop. 
For this case Prometheus provides a Push Gateway server that is live all the time. In this scenario, the cron job will do its job and build its metrics. It will push its metrics before stop.

## Logging

Logging is the action of saving time series information about what a system do.

The developers decide what information should be logged.
Logs can contain:

	* tracing information
	* debugging information
	* errors
	* warnings	

### The importance of logging

Logs are important for sysadmins and developers. Sysdmins check logs to make sure the system is working as expected. Sometimes we cannot observe malfunction of a process but the process can log its status.

Developers write code and they need to find problems in their applications during and after development. Developer has not access on the client machine but can receive the logs of the application and determine its behavior in certain situations.
Usually, applications have a log level that can be set. In debug level the application can log very detailed information. 

### Centralization of logs (fluentd   elastic-search   ok-log)



### Analyzing logs (kebana, ok-log)



## Tracing (zipkin, open tracing, jaeger) - nu folosim inca

### The importance of tracing

### What is a trace

### Examples of tracing in distributed applications