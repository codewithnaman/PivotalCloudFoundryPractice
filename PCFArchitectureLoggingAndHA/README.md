# PCF Architecture, Logging and High availability
We are going to cover below topics in this module.
 1. Cloud Native Apps
 2. Elastic Runtime Architecture
 3. High Availability


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
  
  
  ### **Cloud Controller API :**
  
  ### **Routing :**


## 3. High Availability