# PCF Architecture, Logging and High availability
We are going to cover below topics in this module.
 1. Cloud Native Apps
 2. Elastic Runtime Architecture
 3. High Availability
 4. Hands On for Logs and HA

## 1. Cloud Native Apps
When we think to start moving towards the microservice architecture, which indicates that we are embracing distributed 
systems. When we start thinking about to move in microservices pattern, we have to maintain a lot for deployings apps 
like their VMS, their datastores etc. Which multiplied by the services you have, so it's look like a lot of work while
developing microservices. When we think day 2(when application goes in maintenance phase) like updating or patching 
or updating application with SLA of 99.9%, which is generally people have, then it's a challenge.


Also these services lives in ephemeral(last for very short time) infrastructure like VMs, containers, which has downstream
effect on application design. We can prefer these infrastructure as immutable infrastructure. The advantage of the immutable
infrastructure is that they can create quickly and helps us to elastic scale up or down.


There are factors which need to consider when designing the cloud native apps. There are 12 apps factor, which need to
be discuss here and some in coming sections.

**1. Process :** While designing the application we need to take care the application has to be in one or more stateless
process. The sticky session or data need to be store in datastore as backend service. Will discuss that in upcoming modules.


**2. Concurrency :** The scale out is based of process model. This requires to increase instances of the application in 
surge load can be decrease when resources are under utilized.


**3. Disposability :** We have to take care of the application startup and shutdown. Maximum robustness wit startup and graceful
shutdown need to take care in case of cloud of native applications. Consider scenario where your got a high load and
for scale up you need to increase instance and application takes 10-15 min. to just start up that will make running system slow.
Also for data integrity and safety, graceful shutdown is needed.

  
**4. Logs :** In cloud native applications logs are treated as the event streams i.e. you don't need to write logs in file 
system. The logs use standard out or standard err channel and system determine what needs to happen with those logs.


This is all about cloud native application. We will touch other factors from 12 app factor later in sections. Let's 
understand the Elastic Runtime Architecture.


## 2. Elastic Runtime Architecture
Elastic Runtime Architecture is too large. We are going to cover some of important parts of this system. We can define
those in subsystems.
 <ul>
    <li>Diego</li>
    <li>Loggregator</li>
    <li>Cloud Controller API</li>
    <li>Routing</li>
 </ul>

  ### **Diego :** 
  Diego is a self-healing container management system that attempts to keep the correct number of instances 
  running in Diego Cells to avoid network failures and crashes. Diego schedules and runs Tasks and Long-Running Processes (LRP). 
  Task is guaranteed to run at most once e.g. : Staging an application. Long running process can be considered like web 
  applications, LRP can one or more instances. LRP run in immutable containers. These containers run within a cell in 
  cloud foundry. Cloud foundry has pool of cells. Diego Cell directly manages and maintains Tasks and LRPs with the 
  following components:<br/>
   #### **i. Garden :** 
   Garden is process run within cell. Garden manages number of backends like windows and linux to support 
   number of type applications environment.<br/>
   
   #### **ii. Rep** : Rep is the process run within cell. Rep manages container allocations against resource constraints on the 
   Cell, such as memory and disk space.Periodically ensures its set of Tasks and ActualLRPs in the BBS is in sync with 
   the containers actually present on the Cell.Participates in auctions to accept new Tasks and LRP instances.
   Executor is sub-process of rep responsible for understanding state of cell in terms of container allocation.
   Ex. If a node does not have memory to allocate new container on it does not take participate in auction.
   Streams stdout and stderr from container processes to the metron-agent running on the Cell, which in turn forwards 
   to the Loggregator system.

   #### **iii. Bulletin Board System (BBS)** :
   BBS is the API access the Diego database for task and LRP information. Maintains a real-time representation of the 
   state of the Diego cluster, including all desired LRPs, running LRP instances.
   
   #### **iv. Brain** :
   Diego brain has two components auctioneer and converger. Auctioneer is responsible for holding auctions for Tasks 
   and LRP instances.
   <br/><br/>
   ![Diego Architecture](images/diego-flow.png?raw=true)
  
  ### **Loggregator :**
  The Diego Executor collect logs to metron which then forwards app logs, errors, and app and Diego metrics to the 
  Loggregator Doppler component. Loggregator consist of below subsystems:
  #### **i. Doppler :**
  Dopplers gather logs from the Metron Agents, store them in temporary buffers, and forward them to the Traffic 
  Controller or to third-party syslog drains like Splunk or papertrail.
  #### **ii. Traffic Controller :**
  The Traffic Controller handles client requests for logs. It gathers and collates messages from all Doppler servers, \
  and provides external API and message translation as needed for legacy APIs.Traffic Controller also exposes the Firehose.
  #### **iii. Firehose :**
  A websockert endpoint that expose logs, HTTP events, and container metrics from all applications, and metrics from
  all Cloud Foundry system components.Generally the Firehose is plugged in to Nozzles. Nozzles are programs which 
  consume data from the Loggregator Firehose. Nozzles can be configured to select, buffer, and transform data, and 
  forward it to other applications and services. 
  <br/><br/>
  ![Loggregator Architecture](images/loggregator.png?raw=true)
     
  ### **Cloud Controller API :**
  The Cloud Controller provides REST API endpoints for clients to access and manage the system. 
  Cloud Controller maintains a database with tables for orgs, spaces, services, user roles, and more in CC_DB.
  Cloud Controller also maintains a database for the app packages and droplet in BLOB_STORE. Cloud Controller bridge 
  translates app specific language to generic language of tasks of LRPs.
  
  
  ### **Routing :**
  The router routes incoming traffic to the appropriate component, either a Cloud Controller component or a hosted 
  application running on a Diego Cell.Router periodically queries the Diego Bulletin Board System (BBS) to determine 
  which cells and containers each application currently runs on. Using this information, the router recomputes new 
  routing tables based on the IP addresses of each cell virtual machine (VM) and the host-side port numbers for the 
  cell’s containers.
  
  ### **BuildPacks :**
  Buildpacks provide framework and runtime support for your applications. 
  Buildpacks are responsible for building the droplets. We will talk about it in details in upcoming modules

## 3. High Availability
High availability in cloud foundry has 4 levels. There are availability zones like distributed systems and managed from 
zones. If a zone is down we have other availability zone to serve. Elastic runtime process monitored and automatically 
restarted. These task are carried out by BOSH.

There are two process running on each VM known as monit or BOSH agent, to monitor the processes on the component VMs 
that work together to keep your applications running, such as nsync, BBS, and Cell Rep. 
If monit detects a failure, it restarts the process and notifies the BOSH agent on the VM. 
The BOSH agent notifies the BOSH Health Monitor, which triggers responders through plugins such as email 
notifications or paging. If any application crashes the BOSH health monitor detects it by heartbeat signals transmitted by 
BOSH agent using the messaging queue. If it detects failure then it restart it and notify configured users.

The applications self-heal, if any instance got crashed. Rep gives the data to BBS which is used by
brain converger to compare the actual state and desired state, if found not in desired state then it perform 
action to achieve desired state.

## 4. Hands On for Logs and HA
In below section we will how to tail logs, view application events, and scale an application with ease.

First we login to our local dev using below commands<br/>
**cf api  https://api.local.pcfdev.io --skip-ssl-validation** <br/>
**cf login**<br/>
**cf target**<br/>


This will choose deploy an application and view logs of the Elastic Runtime environment and application and understand this.<br/>
**cf push <application_name> -p <path_for_artifact> -m <memory> --random-route --no-start**<br/>
**cf push articulate -p ./articulate-0.2.jar -m 512M --random-route --no-start**<br/>


Use cf logs to see the logs for application and EC runtime.<br/>
**cf logs <application_name>**<br/>
**cf logs articulate**

Below screen shows you the logs of application and cell.

![Logging Hands on](images/logging.png?raw=true)

We can also see logs in our app manager. Below screenshot shows the app manager logs screen.
![App Manager Logging Hands on](images/loggingUI.PNG?raw=true)

User **cf events** command to see the events of application.
```cmd
C:\Users\Naman Gupta\Documents\PCFPractice>cf events articulate
Getting events for app articulate in org pcfdev-org / space pcfdev-space as admin...

time                          event                 actor   description
2018-08-30T01:52:23.00+0530   audit.app.update      admin   state: STARTED
2018-08-30T01:48:01.00+0530   audit.app.update      admin   disk_quota: 512, instances: 1, memory: 512
2018-08-30T01:46:17.00+0530   audit.app.update      admin   disk_quota: 512, instances: 1, memory: 512
2018-08-30T01:44:43.00+0530   audit.app.map-route   admin
2018-08-30T01:44:43.00+0530   audit.app.create      admin   instances: 1, memory: 512, state: STOPPED, environment_json: PRIVATE DATA HIDDEN

C:\Users\Naman Gupta\Documents\PCFPractice>
``` 

Now we will look how to scale app horizontally and vertically with HA. Let's first see the vertical scaling i.e. increasing container resources.
To scale application vertically we use **cf scale -m <memory> <application_name>.**<br/>
**cf scale -m 1G articulate**

![App Manager Logging Hands on](images/scaleVertical.png?raw=true)

Application has been updated and running on 1 GB of memory now. Using this command we can scale vertically.
The below are events of articulate after scaling.
```cmd
C:\Users\Naman Gupta\Documents\PCFPractice>cf events articulate
Getting events for app articulate in org pcfdev-org / space pcfdev-space as admin...

time                          event                 actor   description
2018-08-30T02:17:17.00+0530   audit.app.update      admin   state: STARTED
2018-08-30T02:17:17.00+0530   audit.app.update      admin   state: STOPPED
2018-08-30T02:17:17.00+0530   audit.app.update      admin   memory: 1024
2018-08-30T01:52:23.00+0530   audit.app.update      admin   state: STARTED
2018-08-30T01:48:01.00+0530   audit.app.update      admin   disk_quota: 512, instances: 1, memory: 512
2018-08-30T01:46:17.00+0530   audit.app.update      admin   disk_quota: 512, instances: 1, memory: 512
2018-08-30T01:44:43.00+0530   audit.app.map-route   admin
2018-08-30T01:44:43.00+0530   audit.app.create      admin   instances: 1, memory: 512, state: STOPPED, environment_json: PRIVATE DATA HIDDEN

C:\Users\Naman Gupta\Documents\PCFPractice>
``` 
As seen in above picture the memory is updated and the container is restarted.<br/>

Now to scale horizontal we use **cf scale <application_name> -i <number_of_instances>**.<br/>
**cf scale articulate -i 3**

When we fired this command in local, the below is output
```cmd
C:\Users\Naman Gupta\Documents>cf scale articulate -i 3
Scaling app articulate in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Documents>cf app articulate
Showing health and status for app articulate in org pcfdev-org / space pcfdev-space as admin...

name:              articulate
requested state:   started
instances:         3/3
usage:             512M x 3 instances
routes:            articulate-cheerful-ardvark.local.pcfdev.io
last uploaded:     Thu 30 Aug 02:46:25 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE
                   java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f java-main
                   open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE
                   spring-auto-reconfiguration=1.10...

     state      since                  cpu     memory           disk             details
#0   running    2018-08-29T21:18:48Z   0.2%    264.3M of 512M   159.8M of 512M
#1   starting   2018-08-29T21:20:11Z   37.5%   201.1M of 512M   159.8M of 512M
#2   starting   2018-08-29T21:20:11Z   37.6%   191.7M of 512M   159.8M of 512M

C:\Users\Naman Gupta\Documents>cf app articulate
Showing health and status for app articulate in org pcfdev-org / space pcfdev-space as admin...

name:              articulate
requested state:   started
instances:         3/3
usage:             512M x 3 instances
routes:            articulate-cheerful-ardvark.local.pcfdev.io
last uploaded:     Thu 30 Aug 02:46:25 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE
                   java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f java-main
                   open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE
                   spring-auto-reconfiguration=1.10...

     state     since                  cpu    memory           disk             details
#0   running   2018-08-29T21:18:48Z   0.3%   258.3M of 512M   159.8M of 512M
#1   running   2018-08-29T21:21:36Z   0.2%   287.6M of 512M   159.8M of 512M
#2   running   2018-08-29T21:21:34Z   0.2%   279.6M of 512M   159.8M of 512M
```

On logs
```cmd
C:\Users\Naman Gupta>cf logs articulate | findstr "API CELL"
   2018-08-30T02:50:11.10+0530 [API/0] OUT Updated app with guid 12956c3d-626b-4fb7-9987-98e5d2001d4f ({"instances"=>3})
   2018-08-30T02:50:11.13+0530 [CELL/1] OUT Creating container
   2018-08-30T02:50:11.14+0530 [CELL/2] OUT Creating container
   2018-08-30T02:50:11.54+0530 [CELL/1] OUT Successfully created container
   2018-08-30T02:50:11.68+0530 [CELL/2] OUT Successfully created container
   2018-08-30T02:50:15.35+0530 [CELL/1] OUT Starting health monitoring of container
   2018-08-30T02:50:18.08+0530 [CELL/2] OUT Starting health monitoring of container
   2018-08-30T02:51:34.84+0530 [CELL/2] OUT Container became healthy
   2018-08-30T02:51:36.15+0530 [CELL/1] OUT Container became healthy
```

```cmd
C:\Users\Naman Gupta>cf events articulate
Getting events for app articulate in org pcfdev-org / space pcfdev-space as admin...

time                          event                 actor   description
2018-08-30T02:50:11.00+0530   audit.app.update      admin   instances: 3
2018-08-30T02:47:46.00+0530   audit.app.update      admin   state: STARTED
2018-08-30T02:46:24.00+0530   audit.app.update      admin   disk_quota: 512, instances: 1, memory: 512
2018-08-30T02:45:40.00+0530   audit.app.map-route   admin
2018-08-30T02:45:40.00+0530   audit.app.create      admin   instances: 1, memory: 512, state: STOPPED, environment_json: PRIVATE DATA HIDDEN
```

If application is crash then repush the app with -t argument which increase the health check timeout period

**cf push articulate -p .\PCFPractice\articulate-0.2.jar -m 512M --random-route --no-start -t 120**

These instances are load balanced, you can use page url and see in HA and availability tab of page, it returns a new IP 
every time. On same page we have kill button which unexpectly kill instance of the application, now we
can see using events and logs of EC Runtime that CF provide support for self healing and start a new container when detects
a crash of any container and application instance. Below shows output from the logs and events when we click kill.
<br/>Logs:
```cmd
C:\Users\Naman Gupta>cf logs articulate | findstr "API CELL"
   2018-08-30T02:59:43.80+0530 [CELL/1] OUT Exit status 0
   2018-08-30T02:59:43.82+0530 [CELL/1] OUT Destroying container
   2018-08-30T02:59:43.89+0530 [CELL/1] OUT Creating container
   2018-08-30T02:59:43.90+0530 [API/0] OUT Process has crashed with type: "web"
   2018-08-30T02:59:43.92+0530 [API/0] OUT App instance exited with guid 12956c3d-626b-4fb7-9987-98e5d2001d4f payload: {"instance"=>"3256800c-6395-4c98-4071-46cb", "index"=>1, "reason"=>"CRASHED", "exit_description"=>"2 error(s) occurred:\n\n* 2 error(s) occurred:\n\n* Codependent step exited\n* cancelled\n* cancelled", "crash_count"=>1, "crash_timestamp"=>1535578183810997701, "version"=>"ff58d4e0-ca24-4463-b0ce-c8c89898cca8"}
   2018-08-30T02:59:44.28+0530 [CELL/1] OUT Successfully destroyed container
   2018-08-30T02:59:44.30+0530 [CELL/1] OUT Successfully created container
   2018-08-30T02:59:47.92+0530 [CELL/1] OUT Starting health monitoring of container
   2018-08-30T03:00:21.24+0530 [CELL/1] OUT Container became healthy
```
Application and Events :
```cmd
C:\Users\Naman Gupta\Documents>cf app articulate
Showing health and status for app articulate in org pcfdev-org / space pcfdev-space as admin...

name:              articulate
requested state:   started
instances:         3/3
usage:             512M x 3 instances
routes:            articulate-cheerful-ardvark.local.pcfdev.io
last uploaded:     Thu 30 Aug 02:46:25 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE
                   java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f java-main
                   open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE
                   spring-auto-reconfiguration=1.10...

     state      since                  cpu    memory           disk             details
#0   running    2018-08-29T21:18:48Z   0.2%   293.5M of 512M   159.8M of 512M
#1   starting   2018-08-29T21:29:43Z   0.0%   165.7M of 512M   159.8M of 512M
#2   running    2018-08-29T21:21:34Z   0.2%   297.1M of 512M   159.8M of 512M

C:\Users\Naman Gupta\Documents>cf app articulate
Showing health and status for app articulate in org pcfdev-org / space pcfdev-space as admin...

name:              articulate
requested state:   started
instances:         3/3
usage:             512M x 3 instances
routes:            articulate-cheerful-ardvark.local.pcfdev.io
last uploaded:     Thu 30 Aug 02:46:25 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE
                   java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f java-main
                   open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE
                   spring-auto-reconfiguration=1.10...

     state     since                  cpu    memory           disk             details
#0   running   2018-08-29T21:18:48Z   0.3%   289.2M of 512M   159.8M of 512M
#1   running   2018-08-29T21:30:21Z   0.2%   278.7M of 512M   159.8M of 512M
#2   running   2018-08-29T21:21:34Z   0.2%   295.3M of 512M   159.8M of 512M

2018-08-30T02:59:43.00+0530   app.crash                 articulate   index: 1, reason: CRASHED, instance: 3256800c-6395-4c98-4071-46cb, exit_description: 2 error(s) occurred:

                                                                     * 2 error(s) occurred:

                                                                     * Codependent step exited
                                                                     * cancelled
                                                                     * cancelled
2018-08-30T02:59:43.00+0530   audit.app.process.crash   web          index: 1, reason: CRASHED, instance: 3256800c-6395-4c98-4071-46cb, exit_description: 2 error(s) occurred:
                                                                                                                                                                                             * 2 error(s) occurred:

                                                                     * Codependent step exited                                                                                               * cancelled
                                                                     * cancelled                                        2018-08-30T02:50:11.00+0530   audit.app.update          admin        instances: 3
2018-08-30T02:47:46.00+0530   audit.app.update          admin        state: STARTED
2018-08-30T02:46:24.00+0530   audit.app.update          admin        disk_quota: 512, instances: 1, memory: 512
2018-08-30T02:45:40.00+0530   audit.app.map-route       admin
2018-08-30T02:45:40.00+0530   audit.app.create          admin        instances: 1, memory: 512, state: STOPPED, environment_json: PRIVATE DATA HIDDEN
```
  
This is all about logging,scaling and HA in PCF. Below is link which explain HA in PCF details.

https://content.pivotal.io/blog/the-four-levels-of-ha-in-pivotal-cf
