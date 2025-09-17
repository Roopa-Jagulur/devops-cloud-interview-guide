**1. Difference between monitoring and observability?**
**Monitoring** : tell you **when** something is wrong with your system- uses **dashboard/Alerts**

**Observability**: tell you **why** something is wrong with your system. uses **Logs/Metrics/Traces**-shows like why your system is ran out of cpu / which processess is taking a lot of CPU / and also tell you why that process is taking a lot of cpu etc.,

**example**: car dashboard - monitoring 
             car mechanic - observability

**2. How to emit custom logs and metrics in your application?**
To emit any application related custom metric logs at **infomation, error / debug level-** every programming language has defined frameworks which can help you do this.
**Example**: For Java based applocations - popular one is Log4J 

For custom metrics - developers have to use packages like prometheus client - if prometheus is your observabilty tools for metrics, so with in the application, developers need to use Prometheus client and using the prometheus client and using metrics like gauage/ counter (for defining particular metrics), they need to actually instrument this metrics.**but what exactly this instrumentation of metrics?** 
**Example: to show what developers use prometheus client in their application to get custom metrics**
<img width="1326" height="604" alt="image" src="https://github.com/user-attachments/assets/38ab6c5b-3150-42dd-8e7a-ed9bfea0d193" />

**Opentelemetry** is like prometheus and it is vendor neutral. 

**3. what kind of metrics do you work with in your current project** 
We scrape metrics using Prometheus where the different types of metrics that we scrape.
- Infrastructure related metrics - like node exporter to get info related to the node
- kubestate metrics - to get the information of kubelet / kube API server / other kubernetes components like /etcd
- Application related metrics - at the end of the day, for a customer, it is important to have the application available and reliable. 
<img width="1702" height="686" alt="image" src="https://github.com/user-attachments/assets/6eea3fe4-08ff-4d80-b933-0910c898748b" />

**4. Have you worked on obersvabilty if yes, explian what did you do?**
**observability** - is the ability to understand internal state of the system which makes use of metrics, logs and traces using which observability can tell currently if your system is healthy or if there is any issue with the system.
**example**: Explian how I implemented observability in the current organization for a service called Payment Service?
Take a problem - we have a payments application. There are multiple requests to the payment application. Concurrent request from multiple users made Payments was going down. There were issues from customers. So to troubleshoot this issue and fix it permanently, I set up observability.

Initially I started with
**logs ->**So I asked application developers to use a logging framework such as log4J because they were 
           using Java application. And I requested the developers to log information at various levels 
           **such as info logging, debug logging,  error logging, trace logging,** and as much as possible,  
           so that whenever there is issue with the payments application in the lower environment, we can 
           immediately look at the logs, look at the particular time stamp when the payments application 
           went down, or when there is issue reported by the client and using the log, we will understand 
           what can be the issue, or at least this will help us with the initial troubleshooting.And because 
           we wanted to persist the logs. So I have implemented Elk stack in the organization. So all the 
           logs from the payment service are pushed to elastic using Logstash. And we used Kibana for visualization.
           
**metrics->** because logging provided us the basic information but for the historical information to understand what was the CPU 
             utilization memory utilization over the period of time. Also to see during which hour of the day is the payments    
             application taking more memory or consuming more memory? I need a dashboard to visualize this information. So for  
             that I have implemented metrics. So as part of metrics, I have set up Prometheus and Grafana along with the default  
             Prometheus Exporters such as node exporter, which help me with the CPU, memory and other information. I also asked 
             the developers to instrument custom metrics. For example, one of the metric that I wanted to scrape with Prometheus 
             was total number of payment requests. Because our application was actually going down when there are multiple users 
             trying to make payments. So I was looking at total number of payments requests as a metric so that I can get the 
             historical information and I can prepare a dashboard for it, which will help me understand and troubleshoot the    
             issue further. So as the development team to scrape that custom metrics and they have used open telemetry for the 
             scraping of metrics.

**tracing->**So I have used Jaeger to set up distributed tracing and I was using the tracing information to see if payments 
             application was making a request to the database and if it was making requests to the database or other 
             microservices. Is there any latency that is involved, or is there any bottleneck for the payments application when
             it is trying to connect to other microservices in? In a nutshell, I was trying to understand the journey of the 
             request to the payment service and from the payment service to the supporting microservices. 
             
  So this is how I implemented observability in the current organization for a service called Payment Service.

 **5. What is the difference between metrics, logs and traces?** 
 **Logs** - Logs are used to capture what happened with the application during a particular timestamp. For example, you want to check if there is any user request at 7:00 in the morning. You can look at the logs to understand if there is a user request and understand how application process the user request. Was there any issue with the application while processing the user request?
So **Logs** are often used to debug or troubleshoot issues with the application. At times, logs can also include stack traces for even better troubleshooting of the issue.

**metrics** - Metrics provide historical information of anything that you want to know about the application.
For example, you want to understand total number of HTTP requests. So you can look at the metrics for total HTTP requests and you can get the historical information about it.
For example, between 7 a.m. and 8 a.m. in the morning. Total number of HTTP requests is something that you can find, or you can find total number of HTTP requests throughout a day, throughout the month, or even throughout the year.

Even it can be total number of pod restarts. 

You can get these kind of historical information using metrics. They are mostly numerical values.

**traces** - Traces help you understand the journey of a request. Now, when a user makes a request with the application using traces, you can understand the complete journey of the request.

Let's say your service interacts with other service interfaces. You can also look at the journey of the request from the service to the other service, and you can understand if there is any issue between the service to service communication.

Usually traces are used to identify if there is any latency involved or if there is any bottleneck with the request from the user.
Maybe user sent a request, it went to the service, but when service tried to connect to the other service, there were some issue.
You can understand that through traces.

**6. What is the difference between push and pull based monitoring?**
**Prometheus, which is a popular tool in the space of monitoring, works on pull based approach.**
**What does that mean?** 
Let's say there is an application. And application exposes /metrics endpoint. Generally, Prometheus follows pull based monitoring where Prometheus scrapes the metrics from /metrics endpoint, for Prometheus scrapes metric from kubestate metrics.

Similarly, if you use node exporter, Prometheus can scrape the metrics from node.

So what Prometheus follows is what we call as pull based monitoring. And for that you provide simple configuration to Prometheus.

For example this is what we do in Prometheus configuration scrape configs job name. And we also provide targets to Prometheus. 
<img width="1036" height="230" alt="image" src="https://github.com/user-attachments/assets/35fa3c5a-82fd-4b94-a79e-7ffe6196758b" />

**statsD, Telegraf are example tools that follow push based metric approach**. So basically in the application or maybe in the script that you write, you have to forward the metrics to this target. Now let's say **statsD.local** is the end point or this is the location. So you have to forward whatever any application related information, number of requests, number of times your pod has restarted. You just need to forward that to statsD or similarly to Telegraph.
<img width="1064" height="46" alt="image" src="https://github.com/user-attachments/assets/a24708c9-1b10-4c75-9b61-b431770d1124" />

**pull based is much easier to configure because you don't have to change the application. And second thing, it is also very powerful because your tool takes care of scraping the metrics from the end point.**

push based monitoring - Target sends the data to the collector, whereas in 
pull based monitoring - data is collected from the target.

**7. which tools have you used to build your observability stack?**
If you worked on enterprise observability platforms, something like Datadog, Dynatrace, GroundCover, Manage engine.

In that case, you can talk about that enterprise platforms, because when it comes to enterprise platforms, they offer everything at one place at least, they offer metrics and traces at one place.

Otherwise, let's say you're planning to switch to DevOps or you haven't got an opportunity to work on enterprise observability platforms. Try to talk about open source tooling because many companies today use open source tooling for observability. There is no wrong in it, but just make sure you explain things as detailed as possible. Don't just say we use Prometheus Grafana instead. Talk from metrics, logs and traces point of view.

For example, this is how you can answer.
So we don't have a single tool for observability in our current organization. We have evaluated multiple open source softwares for metrics and logs. And we have built a custom observability stack where for metrics and monitoring we use Prometheus and
Grafana.

So Prometheus is used for scraping the metrics and Grafana is used for dashboards. We also configured alerts using Prometheus Alert Manager, and our developers use Opentelemetry to instrument the metrics within the applications.

When it comes to logging or logs -> We use elk stack where logstash is used to fetch the logs from the applications.
                                    We are in microservice architecture. So logstash fetch the logs from the microservices and 
                                    that stores the information in elastic.

Visualization of logs, Querying of logs, Debugging is performed through **Kibana**.

And when it comes to traces for distributed tracing we are using **Jaeger**. And we configure everything in Jaeger itself.

And just like metrics, even for instrumentation of traces, our developers use **Opentelemetry**.

So this is how you need to explain when it comes to observability tooling.

**8. User report slowness in app. Logs don't show errors, and cpu is good. Fix it?**
I came across these kind of scenarios in the past, while working with my previous organization, and also on one of the projects in the current organization, I encountered such issues and from my experience,

I will run list of troubleshooting steps:
- The first thing that I will do, I will recheck by taking application into the debug mode. So I'll take the application into
  the debug mode. I'll recheck the logs because sometimes in the normal mode or in the info mode, application does notshow any
  error. So I'll recheck by taking it into the debug mode.

- I will also recheck CPU and memory from Grafana. Just in case the previous DevOps engineer overlook something. If everything
  is perfect, I will head to the next step of troubleshooting.

- That is looking at average request time. So user reported slowness. But what is the average request time? So what is the
  average time when a user makes a request and application responds back? You can get this information from Prometheus and
  Grafana for example. In our organization we use a metric called HTTP request duration seconds. So I'll try to look at this
  metrics on Grafana or Prometheus. Then I'll try to understand what exactly is the request time that user is reporting about.
  User just reported slowness, but I need to know what is the ideal time and what is the time request is currently taking.Now,
  once I understand this now, if there is huge delta between the ideal and actual request time,

- I will head to distributed tracing. So in our company we use Jaeger. You can talk about any other distributed tracing tool.
  So in our company we use Jaeger. So I'll just go to Jaeger to follow the journey of user's request. Most of these cases issue
  is the backend service, or the service that user is trying to reach out is working absolutely fine, because it is already
  checked with the logs and also with the CPU utilization. But issue can be the backend might be communicating with a database,
  or backend might be communicating with other microservice. And this microservice is taking long time to process the request.
  So I'll be looking at p95 latency P99 latency to understand if there is any issue with the request forwarding to other service.

So a lot of times I have seen there is issue with DB connection pool. So unless you look at the database log you will not understand there is a connection pool issue with the database, or maybe there is CPU throttling with the database, which will not understand.

If you look at the logs of the service that user is trying to reach, and in some cases this service might contact another service. And there can be series of services which interact to respond back to the user. To understand the complete request journey and to understand what is the origin of latency, you need to look at distributed tracing tool and follow the complete journey of the request. So this is what I am going to do and identify.

What exactly is the step why user is reporting the slowness? So this should typically tell me what is the issue.

**9. How do you trace a request across multiple microservices in a kubernetes cluster**

when this request typically goes through multiple microservices we call this as distributed tracing. Now this is the question of the interviewer - How do you implement distributed tracing as part of observability?

client request crossing multiple microservices to fullfill the client request, example: 
<img width="848" height="502" alt="image" src="https://github.com/user-attachments/assets/290381d6-6b71-4870-8d95-c23eec848b0d" />

So you can just tell the interviewer we use tools like Jaeger. So Jaeger is a popular tool in the space of distributed tracing.
However, what Jaeger does, Jaeger collects the information. Maybe it gets this information from Otel collector, which is Opentelemetry collector.

**But interviewer is asking, how do you trace a request? That is, how do you instrumented trace?**
So in our current organization, we have a development team who work with these microservices. And whenever a request is received by the microservice or whenever a request is originated at the first microservice, a **trace ID** is created by the tracing SDK that our developers use. It can be open telemetry as well / etc.,So for every trace, as soon as it initializes, a trace ID is created. And this trace id is forwarded to the microservice next microservice and to the every hop within the request.
  
And also because there are multiple hops in the request, a **span** is also created. What does that mean?
You know, a span is created just to identify at which point the request is taking lot of time. Typically there are multiple hops.
You need to understand at which hop the request is taking more time.

So for this a **trace ID** is created and information about the **spans** are collected.

**And end of the day they are visualized onto the distributed tracing tools like Jaeger.**

**10. A pod crashes randomly with OOMKILLED. how do you identify and fix this?**
OOM (out of memory) 
So bluntly speaking, there are two ways to fix this issue: 
One - Go to the container. Look at limits that are set to the container and just increase the limits.
Two - Take the logs at the time of crash, forward the logs to the developer / if possible, once the container starts and it is about to crash one more time, you need to take a trace of the container that is stack trace / if you can't wait for the container to crash, you need to simulate this behavior. Like, you need to simulate in a way that container will crash and share that thread dump and heap dump to the developer.

put this in a better way for interviewer
As a first step, I will confirm OOMkilled by running **kubectl get pods**. So I look at the output of **kubectl get pods** to see if the status is crash loop back off and pod failure reason is OOMKILLED. So this will help me with the confirmation and once I confirm that the issue is same.
As second step, I will run **kubectl get deployment** and **describe the deployment** to check what are the limits that are set on the container. If the limits are too low I will take this into consideration because if the limits are too low, obviously
I can increase the limits- However, to understand if the limits are too low, I will head to the Prometheus or I will head to
the metrics to understand historical data about memory usage of this application.
Is this application historically crashing?
Is it that application is historically reaching the limit that is set in the container limits? [ How will I know if the limits are too low? I'll go to Prometheus.Look at the historic data.]

If yes, obviously I'll try to increase the limits of the container because application most of the times is reaching the limits that are mentioned in the container.

If it is very random, if it is a one time crash application is far away from the limits in terms of memory usage. Then I'll try to look at the logs at the time of crash. I'll try to share that with the developer if possible. I will ask / involve the QA team to create a test suite, or I will try to send multiple requests to the pod to simulate pod crash, and I will share thread dump and heap dump with the developer. Now this will help the developer to analyze why there is a random crash. And once they identify, probably they might release a new version of the application and I will deploy the new version of the application on the cluster.

**11. You've configured alerts but woken up at 2PM by false alrams. what's your strategy to reduce noise?**

You have configured 80% Percentage CPU utilization. If it exceeds, send a notification. In fact, CPU utilization hit 80.5% at 2 a.m. in the morning. Let's assume this is the scenario. Obviously you will get a notification. You are the SRE engineer. You woke up at 2 a.m. in the morning. As soon as you woke up, you realized that CPU utilization came down and it sustained at 40% for the next 2 to 3 hours, and then it came down to 35% and it remained at 35%. Only once CPU utilization exceeded 80%. And that was just for a small period of time. So you woke up at 2 a.m.. There is no real reason you could have checked this issue in the morning as well, because CPU utilization did not hit 90. It did not hit 95, but it just hit 80%. And that was also for a very short period of time. 
**So the question is what's your strategy to reduce the noise?**
Strategy to reduce the noise, not to eliminate the noise completely. Because no matter what you do, you will still see the false alarms as a SRE engineer, sometimes these false alarms cannot be ignored because, you know, let's say you don't set up all kind of alarms, some corner situations or some situations that you don't expect might hit.
So as a DevOps / SRE engineer, if you are taking care of observability, you should be prepared for false alarm.

**But the question is how do you reduce the noise as much as possible?**
There are two approaches:
**First approach is called as alert tuning approach** - where you will not configure the alerts aggressively.
Ex: You configured an alert at 80% utilization. Instead, you could have configured it 90 percentage, and you could have told your alerting system or monitoring system. Make sure to send an alert only if 90% is sustained for five minutes, or at least two minutes. If this condition is met. Send an alert.

**Second approach configure more SLO (Service Level Objective) based alerting** - you will define if one percentage of requests Turns out to be errors in the time span of five minutes. Send me an alert, because 100 number looks to you like a very big number, right?
When you say 100 errors, it looks like a very big number. But what if there are 10 million requests in the last five minutes?
Then 100 looks like a very small number. So instead of defining a random number, define in terms of percentage. Typically what you put in your service level objective that you share with your customer. So these are better ways of defining the alerts or reducing the noise.
