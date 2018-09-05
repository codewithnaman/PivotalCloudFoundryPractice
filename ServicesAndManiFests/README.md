# Services
Let's talk again about cloud native apps and 12-factor apps. In this section we will talk about below from 12-factor 
apps

1. Config
2. Backing Services

### 1. Config
The 12 App factors suggest that the configuration that is likely to vary deployments to deployments should store in
environment variables such as DB path, MQ locations, connected system URLS etc.

### 2. Backing Services
Backing service is any service the app consumes over the network as part of its normal operation such as
Data store, messaging queue, SMTP service or any other developed system. In 12 factor app
the service is considered as attached services which can be accessed via a URL which is deployed in loose coupled manner


## Pivotal Cloud Foundry service and configuration
PCF has 2 types of services to configure.
1. Managed Services
2. User provided service instances

Services configuration like their access URLs and other properties passed to application using environment variables.
When we bind a service to a application it's properties are set to Environment variables of container. We can connect
to these backend services using these environment variables. We will see in hand-on how these properties are set and used
by our applications.


### 1. Managed services
These are services like Datastores, MQs etc. which is managed by the pivotal and available in pivotal marketplace. These
backed service can be created, bind, unbind and delete. To create and bind the managed services with application we use
below commands<br/>
**cf m** - Checks for available marketplace services
```cmd
F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>cf m
Getting services from marketplace in org pcfdev-org / space pcfdev-space as admin...
OK

service        plans             description
local-volume   free-local-disk   Local service docs: https://github.com/cloudfoundry-incubator/local-volume-release/
p-mysql        512mb, 1gb        MySQL databases on demand
p-rabbitmq     standard          RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.
p-redis        shared-vm         Redis service to provide a key-value store

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.
```

To create a managed service instance we use below command<br/>
**cf create-service <service_name_from_marketplace> <plan_name> <service_name_to_initialize>**<br/>
```cmd
F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>cf create-service p-mysql 512mb attendee-mysql
Creating service instance attendee-mysql in org pcfdev-org / space pcfdev-space as admin...
OK
```
To bind service we use below command.<br/>
**cf bind-service <application_name> <service_name>**
```cmd
C:\Users\Naman Gupta\Desktop\02\application>cf bind-service attedee-service attendee-mysql
Binding service attendee-mysql to app attedee-service in org pcfdev-org / space pcfdev-space as admin...
OK
TIP: Use 'cf restage attedee-service' to ensure your env variable changes take effect

```

In out previous examples we were using articulate application. That application also uses the attendee service to consume
some REST APIs and store data to datastore. The attender service uses the DB to store data. We will use our created service
to bind it with the attendee service and then we create user managed service for "attendee service", and bind it with
articulate application. Steps we follow are
 1. Push attendee service application to CF
 2. See the error in log (Shows not able to connect to DB)
 3. Bind service with attendee-mysql
 4. Restart the application so the properties can be injected into the environment variables of container
 5. Check service on URL
 
We will see User managed services in next section.
```cmd
C:\Users\Naman Gupta\Desktop\02\application>REM 1. Push attendee service application to CF
C:\Users\Naman Gupta\Desktop\02\application>cf push attedee-service -p attendee-service-0.1.jar -m 768M --random-route
Pushing app attedee-service to org pcfdev-org / space pcfdev-space as admin...
Getting app info...
Creating app with these attributes...
+ name:       attedee-service
  path:       C:\Users\Naman Gupta\Desktop\02\application\attendee-service-0.1.jar
+ memory:     768M
  routes:
+   attedee-service-sleepy-emu.local.pcfdev.io

Creating app attedee-service...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 632.09 KiB / 632.09 KiB [=========================================================================================================================] 100.00% 1s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Downloading dotnet-core_buildpack...
   Downloading java_buildpack...
   Downloading ruby_buildpack...
   Downloading nodejs_buildpack...
   Downloaded nodejs_buildpack
   Downloading go_buildpack...
   Downloading python_buildpack...
   Downloaded ruby_buildpack
   Downloading php_buildpack...
   Downloaded java_buildpack
   Downloading staticfile_buildpack...
   Downloaded go_buildpack
   Downloading binary_buildpack...
   Downloaded dotnet-core_buildpack
   Downloaded php_buildpack
   Downloaded binary_buildpack
   Downloaded staticfile_buildpack
   Downloaded python_buildpack
   Creating container
   Successfully created container
   Downloading app package...
   Downloaded app package (26.6M)
   Staging...
   -----> Java Buildpack Version: v3.13 (offline) | https://github.com/cloudfoundry/java-buildpack.git#03b493f
   -----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.3s)
   -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculat
or-2.0.2_RELEASE.tar.gz (found in cache)
          Memory Settings: -Xmx681574K -XX:MaxMetaspaceSize=104857K -Xss349K -Xms681574K -XX:MetaspaceSize=104857K
   -----> Downloading Container Certificate Trust Store 2.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-certificate-trust-store/container-c
ertificate-trust-store-2.0.0_RELEASE.jar (found in cache)
          Adding certificates to .java-buildpack/container_certificate_trust_store/truststore.jks (2.1s)
   -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_R
ELEASE.jar (found in cache)
   Exit status 0
   Staging complete
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploading droplet...
   Uploaded build artifacts cache (109B)
   Uploaded droplet (72.1M)
   Uploading complete
   Destroying container

Waiting for app to start...
Start unsuccessful

TIP: use 'cf logs attedee-service --recent' for more information
FAILED

C:\Users\Naman Gupta\Desktop\02\application>REM 2. See the error in log (Shows not able to connect to DB)
C:\Users\Naman Gupta\Desktop\02\application>cf logs attedee-service --recent
Retrieving logs for app attedee-service in org pcfdev-org / space pcfdev-space as admin...

   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR     at org.springframework.boot.autoconfigure.data.rest.SpringBootRepositoryRestMvcConfiguration$$EnhancerBy
SpringCGLIB$$ad0a43d2$$FastClassBySpringCGLIB$$ab76e4df.invoke(<generated>)
   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR     at org.springframework.cglib.proxy.MethodProxy.invokeSuper(MethodProxy.java:228)
   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR     at org.springframework.boot.autoconfigure.data.rest.SpringBootRepositoryRestMvcConfiguration$$EnhancerBy
SpringCGLIB$$ad0a43d2.resourceMappings(<generated>)
   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR     at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR     at java.lang.reflect.Method.invoke(Method.java:498)
   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR     at org.springframework.beans.factory.support.SimpleInstantiationStrategy.instantiate(SimpleInstantiation
Strategy.java:162)
   2018-09-03T17:00:05.63+0530 [APP/PROC/WEB/0] ERR Caused by: org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'attendeeR
epository': Cannot create inner bean '(inner bean)#6a076631' of type [org.springframework.orm.jpa.SharedEntityManagerCreator] while setting bean property 'entit
yManager'; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name '(inner bean)#6a076631': Cannot resolve re
ference to bean 'entityManagerFactory' while setting constructor argument; nested exception is org.springframework.beans.factory.BeanCreationException: Error cr
eating bean with name 'org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration': Injection of autowired dependencies failed; nested excepti
on is org.springframework.beans.factory.BeanCreationException: Could not autowire field: private javax.sql.DataSource org.springframework.boot.autoconfigure.orm
.jpa.JpaBaseConfiguration.dataSource; nested exception is org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'dataSource' de
fined in class path resource [org/springframework/boot/autoconfigure/jdbc/DataSourceAutoConfiguration$NonEmbeddedConfiguration.class]: Bean instantiation via fa
ctory method failed; nested exception is org.springframework.beans.BeanInstantiationException: Failed to instantiate [javax.sql.DataSource]: Factory method 'dat
aSource' threw exception; nested exception is org.springframework.boot.autoconfigure.jdbc.DataSourceProperties$DataSourceBeanCreationException: Cannot determine
 embedded database driver class for database type NONE. If you want an embedded database please put a supported one on the classpath. If you have database setti
ngs to be loaded from a particular profile you may need to active it (the profiles "cloud" are currently active).
.
.
.


C:\Users\Naman Gupta\Desktop\02\application>REM 3. Bind service with attendee-mysql
C:\Users\Naman Gupta\Desktop\02\application>cf bind-service attedee-service attendee-mysql
Binding service attendee-mysql to app attedee-service in org pcfdev-org / space pcfdev-space as admin...
OK
TIP: Use 'cf restage attedee-service' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\application>REM 4. Restart the application so the properties can be injected into the environment variables of container
C:\Users\Naman Gupta\Desktop\02\application>cf restart attedee-service
Restarting app attedee-service in org pcfdev-org / space pcfdev-space as admin...

Stopping app...

Waiting for app to start...

name:              attedee-service
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            attedee-service-sleepy-emu.local.pcfdev.io
last uploaded:     Mon 03 Sep 16:57:36 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   java-main open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10...
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                   -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100%
                   -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY
                   -Djavax.net.ssl.trustStore=$PWD/.java-buildpack/container_certificate_trust_store/truststore.jks
                   -Djavax.net.ssl.trustStorePassword=java-buildpack-trust-store-password" && SERVER_PORT=$PORT eval exec
                   $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher

     state     since                  cpu     memory           disk             details
#0   running   2018-09-03T11:39:36Z   75.7%   308.5M of 768M   152.4M of 512M

C:\Users\Naman Gupta\Desktop\02\application>REM  5. Check service on URL
```
Below is screenshot of the service
![Attendee-service](images/Attendee-service.PNG?raw=true)

**How it internally works?**<br/>
For bindable services, Cloud Foundry adds connection details to the VCAP_SERVICES environment variable when you restart
 your application, after binding a service instance to your application. The results are returned as a JSON document 
 that contains an object for each service for which one or more instances are bound to the application. 
 The service object contains a child object for each service instance of that service that is bound to the application.
 To view this JSON document or environment details use below command.<br/>
 **cf env <service_name>**
```cmd
C:\Users\Naman Gupta\Desktop\02\application>cf env attedee-service
Getting env variables for app attedee-service in org pcfdev-org / space pcfdev-space as admin...
OK

System-Provided:
{
 "VCAP_SERVICES": {
  "p-mysql": [
   {
    "credentials": {
     "hostname": "mysql-broker.local.pcfdev.io",
     "jdbcUrl": "jdbc:mysql://mysql-broker.local.pcfdev.io:3306/cf_806794de_7fd7_4572_91c9_f13cd42c047c?user=C1oUGslNEaFOD15f\u0026password=2EgSfHfYeJ9aESVJ",
     "name": "cf_806794de_7fd7_4572_91c9_f13cd42c047c",
     "password": "2EgSfHfYeJ9aESVJ",
     "port": 3306,
     "uri": "mysql://C1oUGslNEaFOD15f:2EgSfHfYeJ9aESVJ@mysql-broker.local.pcfdev.io:3306/cf_806794de_7fd7_4572_91c9_f13cd42c047c?reconnect=true",
     "username": "C1oUGslNEaFOD15f"
    },
    "label": "p-mysql",
    "name": "attendee-mysql",
    "plan": "512mb",
    "provider": null,
    "syslog_drain_url": null,
    "tags": [
     "mysql"
    ],
    "volume_mounts": []
   }
  ]
 }
}

{
 "VCAP_APPLICATION": {
  "application_id": "a1891cf4-b374-4db9-805b-ddd5cf5bf9cc",
  "application_name": "attedee-service",
  "application_uris": [
   "attedee-service-sleepy-emu.local.pcfdev.io"
  ],
  "application_version": "3bea2d5e-c763-4f23-9984-3bf9ebd4a0a1",
  "cf_api": "http://api.local.pcfdev.io",
  "limits": {
   "disk": 512,
   "fds": 16384,
   "mem": 768
  },
  "name": "attedee-service",
  "space_id": "567e1f01-4a79-468b-87f1-28591747f13d",
  "space_name": "pcfdev-space",
  "uris": [
   "attedee-service-sleepy-emu.local.pcfdev.io"
  ],
  "users": null,
  "version": "3bea2d5e-c763-4f23-9984-3bf9ebd4a0a1"
 }
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set


C:\Users\Naman Gupta\Desktop\02\application>

```

### 2. User provided service instances
As we had seen in previous section, managed services. The user can also create service and provide to application as
attached service. This is required because every service can not provided by cloud foundry. Consider the databases
running on mainframe system. This can be provided by creating manually created services user.We are creating a service 
which provide the URL for calling the other systems. There are other type of services which may give different argument.
Below code snippet show how to create and bind application.
1. Get application information(attendee-service)
2. Create user service with the URL of application(attendee-service)
3. Bind service to application in which need to be inject(articulate)
4. Restart application to inject environment application(articulate)
5. See environment for application(articulate)
6. Check if application can see as service and can insert data.

Pre-screenshot of application before attach to user created service.
![Pre_Check_For_Service](images/PreServiceArticulateApplication.png?raw=true)
```cmd
C:\Users\Naman Gupta\Desktop\02\application>REM 1. Get application information
C:\Users\Naman Gupta\Desktop\02\application>cf app attedee-service
Showing health and status for app attedee-service in org pcfdev-org / space pcfdev-space as admin...

name:              attedee-service
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            attedee-service-sleepy-emu.local.pcfdev.io
last uploaded:     Mon 03 Sep 16:57:36 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   java-main open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10...

     state     since                  cpu    memory           disk             details
#0   running   2018-09-03T11:39:36Z   0.3%   331.1M of 768M   152.4M of 512M

C:\Users\Naman Gupta\Desktop\02\application>REM 2. Create user service with the URL of application(attendee-service)
C:\Users\Naman Gupta\Desktop\02\application>cf create-user-provided-service attendee-service -p uri

uri> https://attedee-service-sleepy-emu.local.pcfdev.io
Creating user provided service attendee-service in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Desktop\02\application>REM 3. Bind service to application in which need to be inject(articulate)
C:\Users\Naman Gupta\Desktop\02\application>cf bind-service articulate attendee-service
Binding service attendee-service to app articulate in org pcfdev-org / space pcfdev-space as admin...
OK
TIP: Use 'cf restage articulate' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\application>REM 4. Restart application to inject environment application(articulate)
C:\Users\Naman Gupta\Desktop\02\application>cf restart articulate
Restarting app articulate in org pcfdev-org / space pcfdev-space as admin...

Stopping app...

Waiting for app to start...

name:              articulate
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            articulate-quiet-kob.local.pcfdev.io
last uploaded:     Mon 03 Sep 17:31:32 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   java-main open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10...
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                   -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100%
                   -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY
                   -Djavax.net.ssl.trustStore=$PWD/.java-buildpack/container_certificate_trust_store/truststore.jks
                   -Djavax.net.ssl.trustStorePassword=java-buildpack-trust-store-password" && SERVER_PORT=$PORT eval exec
                   $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher

     state     since                  cpu     memory           disk             details
#0   running   2018-09-03T12:12:12Z   74.1%   344.7M of 768M   159.8M of 512M

C:\Users\Naman Gupta\Desktop\02\application>REM 5. See environment for application(articulate)
C:\Users\Naman Gupta\Desktop\02\application>cf env articulate
Getting env variables for app articulate in org pcfdev-org / space pcfdev-space as admin...
OK

System-Provided:
{
 "VCAP_SERVICES": {
  "user-provided": [
   {
    "credentials": {
     "uri": "https://attedee-service-sleepy-emu.local.pcfdev.io"
    },
    "label": "user-provided",
    "name": "attendee-service",
    "syslog_drain_url": "",
    "tags": [],
    "volume_mounts": []
   }
  ]
 }
}

{
 "VCAP_APPLICATION": {
  "application_id": "86f4743a-589a-481c-b7c6-34fe7535c67a",
  "application_name": "articulate",
  "application_uris": [
   "articulate-quiet-kob.local.pcfdev.io"
  ],
  "application_version": "9c2c0e6f-c3a7-4535-a4db-d510cbf11b0a",
  "cf_api": "http://api.local.pcfdev.io",
  "limits": {
   "disk": 512,
   "fds": 16384,
   "mem": 768
  },
  "name": "articulate",
  "space_id": "567e1f01-4a79-468b-87f1-28591747f13d",
  "space_name": "pcfdev-space",
  "uris": [
   "articulate-quiet-kob.local.pcfdev.io"
  ],
  "users": null,
  "version": "9c2c0e6f-c3a7-4535-a4db-d510cbf11b0a"
 }
}

No user-defined env variables have been set

No running env variables have been set

No staging env variables have been set

C:\Users\Naman Gupta\Desktop\02\application>REM 6. Check if application can see as service and can insert data.
```
Below screenshot shows service and operation, we can perform after attaching service.
![Post_Check_For_Service](images/PostServiceArticulateApplication.png?raw=true)

For more on how to build the different type of user provided service please refer below:

https://docs.pivotal.io/pivotalcf/2-2/devguide/services/user-provided.html

# Manifest

You can imagine how tedious things would be if we had to repeatedly pass a handful of command arguments to the **cf cli** 
each time we push an application. There’s a better way: application manifests allow you to consolidate into a single 
file all of our application’s deployment metadata, including the application name, memory, path, hostname, and much 
more. We can associate services and environment variables to our application in a single manifest file.


To more about more of tags used in manifest file use below link:<br/>
https://docs.pivotal.io/pivotalcf/2-2/devguide/deploy-apps/manifest.html

Hands-on
For hands-on we will do reverse engineer for the application we deployed. We will ask cf-to create the manifest file 
for the attendee-service and then edit the yml generated for increasing number of instances. Below are the steps 
1. Generate manifest for the application
2. Edit the manifest file
3. Path is not in file so add it as well
4. Fire command cf push
```cmd
C:\Users\Naman Gupta\Desktop\02\application>REM 1. Generate manifest for the application
C:\Users\Naman Gupta\Desktop\02\application>cf create-app-manifest attendee-app -p ./manifest.yml
Creating an app manifest from current settings of app attendee-app in org pcfdev-org / space pcfdev-space as admin...
OK
Manifest file created successfully at ./manifest.yml

C:\Users\Naman Gupta\Desktop\02\application>REM 2. Edit the manifest file
C:\Users\Naman Gupta\Desktop\02\application>REM 3. Path is not in file so add it as well
C:\Users\Naman Gupta\Desktop\02\application>REM 4. Fire command cf push
C:\Users\Naman Gupta\Desktop\02\application>cf push
Pushing from manifest to org pcfdev-org / space pcfdev-space as admin...
Using manifest file C:\Users\Naman Gupta\Desktop\02\application\manifest.yml
Getting app info...
Updating app with these attributes...
  name:                attendee-app
  path:                C:\Users\Naman Gupta\Desktop\02\application\attendee-service-0.1.jar
  disk quota:          512M
  health check type:   port
  instances:           2
  memory:              768M
  stack:               cflinuxfs2
  services:
    attendee-mysql
  routes:
    attendee-app-talkative-panda.local.pcfdev.io

Updating app attendee-app...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 632.09 KiB / 632.09 KiB [=========================================================================================================================] 100.00% 1s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Downloading dotnet-core_buildpack...
   Downloading java_buildpack...
   Downloading ruby_buildpack...
   Downloading nodejs_buildpack...
   Downloaded dotnet-core_buildpack
   Downloading go_buildpack...
   Downloaded java_buildpack
   Downloading python_buildpack...
   Downloaded ruby_buildpack
   Downloading php_buildpack...
   Downloaded nodejs_buildpack
   Downloading staticfile_buildpack...
   Downloading binary_buildpack...
   Downloaded go_buildpack
   Downloaded python_buildpack
   Downloaded php_buildpack
   Downloaded staticfile_buildpack
   Downloaded binary_buildpack
   Creating container
   Successfully created container
   Downloading app package...
   Downloaded app package (26.6M)
   Downloading build artifacts cache...
   Downloaded build artifacts cache (109B)
   Staging...
   -----> Java Buildpack Version: v3.13 (offline) | https://github.com/cloudfoundry/java-buildpack.git#03b493f
   -----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.1s)
   -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculat
or-2.0.2_RELEASE.tar.gz (found in cache)
          Memory Settings: -XX:MaxMetaspaceSize=104857K -XX:MetaspaceSize=104857K -Xms681574K -Xss349K -Xmx681574K
   -----> Downloading Container Certificate Trust Store 2.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-certificate-trust-store/container-c
ertificate-trust-store-2.0.0_RELEASE.jar (found in cache)
          Adding certificates to .java-buildpack/container_certificate_trust_store/truststore.jks (0.9s)
   -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_R
ELEASE.jar (found in cache)
   Exit status 0
   Staging complete
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploading droplet...
   Uploaded build artifacts cache (109B)
   Uploaded droplet (72.1M)
   Uploading complete
   Destroying container

Waiting for app to start...

name:              attendee-app
requested state:   started
instances:         2/2
usage:             768M x 2 instances
routes:            attendee-app-talkative-panda.local.pcfdev.io
last uploaded:     Wed 05 Sep 23:26:47 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   java-main open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10...
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                   -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100%
                   -stackThreads=300 -totMemory=$MEMORY_LIMIT) && JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY
                   -Djavax.net.ssl.trustStore=$PWD/.java-buildpack/container_certificate_trust_store/truststore.jks
                   -Djavax.net.ssl.trustStorePassword=java-buildpack-trust-store-password" && SERVER_PORT=$PORT eval exec
                   $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp $PWD/. org.springframework.boot.loader.JarLauncher

     state     since                  cpu     memory           disk             details
#0   running   2018-09-05T17:58:44Z   35.1%   374.9M of 768M   152.4M of 512M
#1   running   2018-09-05T17:58:44Z   35.4%   363.3M of 768M   152.4M of 512M

C:\Users\Naman Gupta\Desktop\02\application>
```
YML file after editing
```yml
applications:
applications:
- name: attendee-app
  disk_quota: 512M
  instances: 2
  memory: 768M
  routes:
  - route: attendee-app-talkative-panda.local.pcfdev.io
  services:
  - attendee-mysql
  path: ./attendee-service-0.1.jar
  stack: cflinuxfs2
  timeout: 100
```
