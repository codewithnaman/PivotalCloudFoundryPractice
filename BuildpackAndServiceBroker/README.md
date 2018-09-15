# BuildPack
Till now we had talked about the buildpack while creating droplets of application, which is chosen by detecting type of
application by the PCF itself. In this section we will talk about customizing buildpacks and use for our application
droplet creation. Let's first understand,how the BuildPack API and then we will do some hands on over customizing java
version for our application as an example.</br>

The buildpack API consist of below three parts:
1. bin/detect - Determines the whether the buildpack can stage application or not. Return 0 if it can.
2. bin/compile - Builds the droplet with runtime and libraries like exporter, required download etc.
3. bin/release - Provides information on how to run the application

For bin/detect the detection criteria is simple like for Ruby it checks the Gemfile exists, For Node Package.json exists
and so on for other and fit that buildpack for providing runtime for application.

When we use the -b <url|name> with cf push it will not call bin/detect.

For bin/compile provides the runtime support,app servers and support libraries according to detect or provided buildpack.

For bin/support the output is YAML document which provide information that how to run your app.

When we use cf push without -b argument it runs all the available local buildpacks and compile with one which returns 0 
as a result.

## Hands-on
Let's first observe our articulate application buildpack information.
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf app articulate
Showing health and status for app articulate in org techacademyglobal / space development as naman.gupta4182@gmail.com...

name:              articulate
requested state:   started
routes:            articulate-appreciative-civet.cfapps.io
last uploaded:     Tue 11 Sep 00:00:26 IST 2018
stack:             cflinuxfs2
buildpacks:        client-certificate-mapper=1.8.0_RELEASE container-security-provider=1.14.0_RELEASE
                   java-buildpack=v4.15-offline-https://github.com/cloudfoundry/java-buildpack.git#553f2c6 java-main java-opts java-security
                   jvmkill-agent=1.16.0_RELEASE open-jdk-...

type:           web
instances:      1/1
memory usage:   700M
     state     since                  cpu    memory           disk
#0   running   2018-09-11T16:51:04Z   0.4%   217.9M of 700M   162M of 1G

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```

As we can see in our output the java buildpack used by our application is offline one. What Offline one means here. PCF
contains some of buildpacks so it can use quickly for building application. we can listdown these buildpacks using below
command.</br>
**cf buildpacks**
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf buildpacks
Getting buildpacks...

buildpack                    position   enabled   locked   filename
staticfile_buildpack         1          true      false    staticfile_buildpack-cached-cflinuxfs3-v1.4.32.zip
java_buildpack               2          true      false    java-buildpack-offline-cflinuxfs3-v4.15.zip
ruby_buildpack               3          true      false    ruby_buildpack-cached-cflinuxfs3-v1.7.22.zip
dotnet_core_buildpack        4          true      false    dotnet-core_buildpack-cached-cflinuxfs3-v2.1.4.zip
nodejs_buildpack             5          true      false    nodejs_buildpack-cached-cflinuxfs3-v1.6.31.zip
go_buildpack                 6          true      false    go_buildpack-cached-cflinuxfs3-v1.8.27.zip
python_buildpack             7          true      false    python_buildpack-cached-cflinuxfs3-v1.6.21.zip
php_buildpack                8          true      false    php_buildpack-cached-cflinuxfs3-v4.3.60.zip
staticfile_buildpack         9          true      false    staticfile_buildpack-cached-cflinuxfs2-v1.4.32.zip
java_buildpack               10         true      false    java-buildpack-offline-cflinuxfs2-v4.15.zip
ruby_buildpack               11         true      false    ruby_buildpack-cached-cflinuxfs2-v1.7.22.zip
dotnet_core_buildpack        12         true      false    dotnet-core_buildpack-cached-cflinuxfs2-v2.1.4.zip
nodejs_buildpack             13         true      false    nodejs_buildpack-cached-cflinuxfs2-v1.6.31.zip
go_buildpack                 14         true      false    go_buildpack-cached-cflinuxfs2-v1.8.27.zip
python_buildpack             15         true      false    python_buildpack-cached-cflinuxfs2-v1.6.21.zip
php_buildpack                16         true      false    php_buildpack-cached-cflinuxfs2-v4.3.60.zip
dotnet_core_buildpack_beta   17         true      false    dotnet-core_buildpack-cached-v1.0.0.zip
hwc_buildpack                18         true      false    hwc_buildpack-cached-v2.3.19.zip
binary_buildpack             19         true      false    binary_buildpack-cached-v1.0.26.zip

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```

The above buildpacks are available offline in PCF, we can add, patch or update the buildpacks as well. The articulate 
application using the **java-buildpack=v4.15-offline** currently. We will use cloudfoundry online java buildpack and see
the difference. Current articulate java version shown in below image.

![Offline Buildpack JavaVersion](images/preBuildPack.PNG?raw=true)

Now we will provide custom java buildpack using below command then see the difference.<br/>
**cf push -p articulate-0.2.jar -m 700M -b https://github.com/cloudfoundry/java-buildpack.git**
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf push articulate -p articulate-0.2.jar -m 700M -b https://github.com/cloudfoundry/java-buildpack.git
.
.
.
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf app articulate
Showing health and status for app articulate in org techacademyglobal / space development as naman.gupta4182@gmail.com...

name:              articulate
requested state:   started
routes:            articulate-appreciative-civet.cfapps.io
last uploaded:     Wed 12 Sep 00:21:53 IST 2018
stack:             cflinuxfs2
buildpacks:        https://github.com/cloudfoundry/java-buildpack.git

type:           web
instances:      1/1
memory usage:   700M
     state     since                  cpu    memory           disk
#0   running   2018-09-11T18:52:25Z   0.3%   199.6M of 700M   162M of 1G
```

As in above output we can observe the buildpack is changed to our java buildpack. Also we will see the java version in application.

![New Buildpack JavaVersion](images/postGitBuildPack.PNG?raw=true)

The java version is same. May be both are up-to date that's why our version has not changed. Now what if we want to use
different java version or different argument to run. We will use below command to change java version or any other parameter
while providing buildpack.<br/>
**cf set-env articulate JBP_CONFIG_OPEN_JDK_JRE "{jre: {version : 1.8.0_45}}"**
```cmd
iculate>cf set-env articulate
JBP_CONFIG_OPEN_JDK_JRE "{jre: {version : 1.8.0_45}}"
Setting env variable 'JBP_CONFIG_OPEN_JDK_JRE' to '{jre: {version : 1.8.0_
45}}' for app articulate in org techacademyglobal / space development as n
aman.gupta4182@gmail.com...
OK
TIP: Use 'cf restage articulate' to ensure your env variable changes take
effect

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf restage articulate
Restaging app articulate in org techacademyglobal / space development as n
aman.gupta4182@gmail.com...

Staging app and tracing logs...
   Downloaded app package (32.8M)
   -----> Java Buildpack 2cd835b | https://github.com/cloudfoundry/java-bu
ildpack.git#2cd835b
   -----> Downloading Jvmkill Agent 1.16.0_RELEASE from https://java-build
pack.cloudfoundry.org/jvmkill/trusty/x86_64/jvmkill-1.16.0_RELEASE.so (0.0
s)
   -----> Downloading Open Jdk JRE 1.8.0_45 from https://java-buildpack.cl
oudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_45.tar.gz (2.3s)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (1.0s)
          JVM DNS caching disabled in lieu of BOSH DNS caching
   -----> Downloading Open JDK Like Memory Calculator 3.13.0_RELEASE from
https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/me
mory-calculator-3.13.0_RELEASE.tar.gz (0.0s)
          Loaded Classes: 18081, Threads: 250
   -----> Downloading Client Certificate Mapper 1.8.0_RELEASE from https:/
/java-buildpack.cloudfoundry.org/client-certificate-mapper/client-certific
ate-mapper-1.8.0_RELEASE.jar (0.0s)
   -----> Downloading Container Security Provider 1.16.0_RELEASE from http
s://java-buildpack.cloudfoundry.org/container-security-provider/container-
security-provider-1.16.0_RELEASE.jar (0.0s)
   -----> Downloading Spring Auto Reconfiguration 2.4.0_RELEASE from https
://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfigurati
on-2.4.0_RELEASE.jar (0.0s)
   Exit status 0
   Uploading droplet, build artifacts cache...
   Uploading droplet...
   Uploading build artifacts cache...
   Uploaded build artifacts cache (45.7M)
   Uploaded droplet (78.7M)
   Uploading complete
   Cell 8875efb3-ebd9-42dc-adfa-3978c9e35594 stopping instance 38c5d53e-31
7e-4541-83da-5d0cfe3dd4cb
   Cell 8875efb3-ebd9-42dc-adfa-3978c9e35594 destroying container for inst
ance 38c5d53e-317e-4541-83da-5d0cfe3dd4cb
   Cell 8875efb3-ebd9-42dc-adfa-3978c9e35594 successfully destroyed contai
ner for instance 38c5d53e-317e-4541-83da-5d0cfe3dd4cb

Waiting for app to start...

name:              articulate
requested state:   started
instances:         1/1
usage:             700M x 1 instances
routes:            articulate-appreciative-civet.cfapps.io
last uploaded:     Wed 12 Sep 00:21:07 IST 2018
stack:             cflinuxfs2
buildpack:         https://github.com/cloudfoundry/java-buildpack.git
start command:     JAVA_OPTS="-agentpath:$PWD/.java-buildpack/open_jdk_jre
/bin/jvmkill-1.16.0_RELEASE=printHeapHistogram=1
                   -Djava.io.tmpdir=$TMPDIR
                   -Djava.ext.dirs=$PWD/.java-buildpack/container_security
_provider:$PWD/.java-buildpack/open_jdk_jre/lib/ext
                   -Djava.security.properties=$PWD/.java-buildpack/java_se
curity/java.security
                   $JAVA_OPTS" &&
                   CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/b
in/java-buildpack-memory-calculator-3.13.0_RELEASE
                   -totMemory=$MEMORY_LIMIT -loadedClasses=18793
                   -poolType=metaspace -stackThreads=250
                   -vmOptions="$JAVA_OPTS") && echo JVM Memory
                   Configuration: $CALCULATED_MEMORY &&
                   JAVA_OPTS="$JAVA_OPTS $CALCULATED_MEMORY" &&
                   MALLOC_ARENA_MAX=2 SERVER_PORT=$PORT eval exec
                   $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS
                   -cp $PWD/. org.springframework.boot.loader.JarLauncher

     state     since                  cpu    memory           disk           details
#0   running   2018-09-11T19:06:55Z   0.0%   163.9M of 700M   161.2M of 1G

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
Below is the output after changing environment variable for JDK version.

![Buildpack JavaVersion change](images/postVersionChange.PNG?raw=true)

# Service Broker
Service broker is an API which specifies how a service can be created,bind,unbind and destroyed. When we were working 
with services(managed services), let's understand what happens behind the scene.

Managed services must implement the service broker API. The service broker API is a RESTful API over HTTP. So our service
can be reside anywhere in terms of deployment, it can be within cloud foundry or it can be at different location such as
different VM or different servers etc. Below is the brief summary of the service broker API which need be implement for
new service:<br/>
*  **Catalog management** - describe the plans offered by the service
*  **Provision** - create a service instance
*  **Deprovision** - delete a service instance
*  **Bind** - Create a binding between an app and service instance
*  **Unbind** - Delete binding

When we start implementing a service generally we start with catalog management API which tells about the plans offered by
our service like normal,gold,diamond for different level of services,it is just descriptive information about what is going
to be offered by service and plan.

Then comes to provision state, in provisioning the caller calls the Rest API for cloud controller, Cloud controller then 
calls Service broker API to create a new instance of service. The implementation depends on the implementer can vary 
from service to service.When we get this call we can provision a new VM, use existing VM and create a new instance or 
can provide shared instance with find boundary to access. Then it return instance information to Cloud controller, cloud
controller keep information in it's DB to give information of the instance created and then return back pointer to the caller.

For deprovisioing similar type of steps occur. So not describing here. You can think as reveres process to the provisioning.

When it comes to binding the CLI calls cloud controller then we give request to service broker, broker retrieve or create
credentials for the service and return back to cloud controller. Cloud controller save these variable to their DB to 
populate the environment variables in VCAP variable. Then control return back to CLI. Best practice is that whenever a call for bind is received then we need to create new
instance of the credentials and provide to CC.

Let's do the some hands-on over the service broker.

## Hands-ON
First for this demo we are again going to use dev environment and we used below two projects from Pivtol education and spring cloud repositories.<br/>
https://github.com/spring-cloud/spring-cloud-cloudfoundry-service-broker<br/>
https://github.com/pivotal-education/pcf-spring-music-code<br/>

We will first build the application which, I already build using IDE and gradle from IntelliJ. You can choose your own IDE 
for building application and explore code for understanding.How cloud foundry service broker can be write in simplest way.
 You can use below documentation for more understanding over it. For the basic knowledge I would like to highlight the 
 spring cloud framework for cloud foundry service broker provides the classes, interfaces and REST API mapping to create 
 service broker and register them as service in cloud foundry. 
 
Let's push our service broker application to cloud foundry. 
```cmd
C:\Users\Naman Gupta\Desktop\02\07\applications>cf push mongo-service-broker -p ./cloudfoundry-mongodb-service-broker.jar -m 768M --random-route --no-start
Pushing app mongo-service-broker to org pcfdev-org / space pcfdev-space as user...
Getting app info...
Creating app with these attributes...
+ name:       mongo-service-broker
  path:       C:\Users\Naman Gupta\Desktop\02\07\applications\cloudfoundry-mongodb-service-broker.jar
+ memory:     768M
  routes:
+   mongo-service-broker-wise-ostrich.local.pcfdev.io

Creating app mongo-service-broker...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 16.52 KiB / 16.52 KiB [===========================================================================================================================] 100.00% 1s

Waiting for API to complete processing files...

name:              mongo-service-broker
requested state:   stopped
instances:         0/1
usage:             768M x 1 instances
routes:            mongo-service-broker-wise-ostrich.local.pcfdev.io
last uploaded:     Sat 15 Sep 13:47:54 IST 2018
stack:             cflinuxfs2
buildpack:
start command:

There are no running instances of this app.


C:\Users\Naman Gupta\Desktop\02\07\applications>
```

## Set environment variables.
These environment variables get used by the broker to generate the catalog. These values should be unique across the 
entire Pivotal Cloud Foundry instance to meet the broker API specifications.
```cmd
C:\Users\Naman Gupta\Desktop\02\07\applications>cf set-env mongo-service-broker SERVICE_ID mongo-service-broker-naman
Setting env variable 'SERVICE_ID' to 'mongo-service-broker-naman' for app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...
OK
TIP: Use 'cf restage mongo-service-broker' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\07\applications>cf set-env mongo-service-broker SERVICE_NAME MongoDB-Naman
Setting env variable 'SERVICE_NAME' to 'MongoDB-Naman' for app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...
OK
TIP: Use 'cf restage mongo-service-broker' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\07\applications>cf set-env mongo-service-broker PLAN_ID monog-plan-standard
Setting env variable 'PLAN_ID' to 'monog-plan-standard' for app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...
OK
TIP: Use 'cf restage mongo-service-broker' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\07\applications>cf start mongo-service-broker
Starting app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...

Staging app and tracing logs...
   Downloading dotnet-core_buildpack...
   Downloading java_buildpack...
   Downloading ruby_buildpack...
   Downloading nodejs_buildpack...
   Downloaded dotnet-core_buildpack
   Downloading go_buildpack...
   Downloaded nodejs_buildpack
   Downloading python_buildpack...
   Downloaded ruby_buildpack
   Downloading php_buildpack...
   Downloading staticfile_buildpack...
   Downloaded php_buildpack
   Downloading binary_buildpack...
   Downloaded python_buildpack
   Downloaded staticfile_buildpack
   Downloaded java_buildpack
   Downloaded go_buildpack
   Downloaded binary_buildpack
   Creating container
   Successfully created container
   Downloading app package...
   Downloaded app package (17.3M)
   Staging...
   -----> Java Buildpack Version: v3.13 (offline) | https://github.com/cloudfoundry/java-buildpack.git#03b493f
   -----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.2s)
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
   Uploaded droplet (62.7M)
   Uploading complete
   Destroying container

Waiting for app to start...

name:              mongo-service-broker
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            mongo-service-broker-delightful-kookaburra.local.pcfdev.io
last uploaded:     Sat 15 Sep 14:06:36 IST 2018
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
#0   running   2018-09-15T08:38:40Z   51.9%   315.6M of 768M   142.1M of 512M

C:\Users\Naman Gupta\Desktop\02\07\applications>
```
We have deployed our application, set some environment variables and started application. Let's hit the catalog url. 
The URL has basic authentication so it need username and password which is pivotal and keepitsimple respectively.

![Catalog URL](images/CatalogURLPage.PNG?raw=true)

Now we will Register Service Broker.We will be creating a Space-Scoped broker. Space-Scoped brokers help you during the 
development/testing of your service broker, because they are private to a space and don’t require an admin to enable 
access (list it in the marketplace, provision service instances, etc).

To create service broker we will use below command. <br/>
**cf create-service-broker {SERVICE_BROKER} {USERNAME} {PASSWORD} {URL} [--space-scoped]**
```cmd
C:\Users\Naman Gupta\Desktop\02\07\applications>cf create-service-broker
Incorrect Usage: the required arguments `SERVICE_BROKER`, `USERNAME`, `PASSWORD` and `URL` were not provided

NAME:
   create-service-broker - Create a service broker

USAGE:
   cf create-service-broker SERVICE_BROKER USERNAME PASSWORD URL [--space-scoped]

ALIAS:
   csb

OPTIONS:
   --space-scoped      Make the broker's service plans only visible within the targeted space

SEE ALSO:
   enable-service-access, service-brokers, target

C:\Users\Naman Gupta\Desktop\02\07\applications>cf create-service-broker mongo-service-broker-naman pivotal keepitsimple http://mongo-service-broker-delightful-
kookaburra.local.pcfdev.io --space-scoped
Creating service broker mongo-service-broker-naman in org pcfdev-org / space pcfdev-space as user...
OK

C:\Users\Naman Gupta\Desktop\02\07\applications>cf service-brokers
Getting service brokers as user...

name                         url
mongo-service-broker-naman   http://mongo-service-broker-delightful-kookaburra.local.pcfdev.io

C:\Users\Naman Gupta\Desktop\02\07\applications>cf m
Getting services from marketplace in org pcfdev-org / space pcfdev-space as user...
OK

service         plans             description
MongoDB-Naman   standard          A simple MongoDB service broker implementation
local-volume    free-local-disk   Local service docs: https://github.com/cloudfoundry-incubator/local-volume-release/
p-mysql         512mb, 1gb        MySQL databases on demand
p-rabbitmq      standard          RabbitMQ is a robust and scalable high-performance multi-protocol messaging broker.
p-redis         shared-vm         Redis service to provide a key-value store

TIP:  Use 'cf marketplace -s SERVICE' to view descriptions of individual plans of a given service.

C:\Users\Naman Gupta\Desktop\02\07\applications>
```

As we can see in above output the MongoDB-Naman service is created under the managed services of the mongo. The deployment
of the mongo database I had done in the docker. Below shows the output from the docker toolbox.

```cmd
Naman Gupta@DESKTOP-OBR6B46 MINGW64 /c/Program Files/Docker Toolbox
$ docker ps
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                      NAMES
a4dac7afef9a        mongo               "docker-entrypoint.s…"   13 seconds ago      Up 6 seconds        0.0.0.0:27017->27017/tcp   condescending_volhard
```

Now we are going to give environment variables to service to connect with docker running on the docker. We will also see
the database after connecting and then we deploy spring-music application then bind it. Below are the commands and output
respectively.

```cmd
C:\Users\Naman Gupta\Desktop\02\07\applications>cf set-env mongo-service-broker MONGODB_HOST 192.168.99.1
Setting env variable 'MONGODB_HOST' to 'localhost' for app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...
OK
TIP: Use 'cf restage mongo-service-broker' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\07\applications>cf set-env mongo-service-broker MONGODB_PASSWORD password
Setting env variable 'MONGODB_PASSWORD' to '' for app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...
OK
TIP: Use 'cf restage mongo-service-broker' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\07\applications>cf restart mongo-service-broker
Restarting app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...

Stopping app...

Waiting for app to start...

name:              mongo-service-broker
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            mongo-service-broker-delightful-kookaburra.local.pcfdev.io
last uploaded:     Sat 15 Sep 14:06:36 IST 2018
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

     state     since                  cpu    memory           disk             details
#0   running   2018-09-15T09:03:51Z   0.0%   273.1M of 768M   142.1M of 512M

C:\Users\Naman Gupta\Desktop\02\07\applications>cf push spring-music -p spring-music.war -m 768M --random-route
Pushing app spring-music to org pcfdev-org / space pcfdev-space as user...
Getting app info...
Creating app with these attributes...
+ name:       spring-music
  path:       C:\Users\Naman Gupta\Desktop\02\07\applications\spring-music.war
+ memory:     768M
  routes:
+   spring-music-patient-roan.local.pcfdev.io

Creating app spring-music...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 345.92 KiB / 345.92 KiB [=========================================================================================================================] 100.00% 1s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Downloading dotnet-core_buildpack...
   Downloading java_buildpack...
   Downloading ruby_buildpack...
   Downloading nodejs_buildpack...
   Downloading go_buildpack...
   Downloaded dotnet-core_buildpack
   Downloading python_buildpack...
   Downloaded java_buildpack
   Downloading php_buildpack...
   Downloaded ruby_buildpack
   Downloading staticfile_buildpack...
   Downloaded nodejs_buildpack
   Downloading binary_buildpack...
   Downloaded binary_buildpack
   Downloaded staticfile_buildpack
   Downloaded php_buildpack
   Downloaded python_buildpack
   Downloaded go_buildpack
   Creating container
   Successfully created container
   Downloading app package...
   Downloaded app package (24.5M)
   Staging...
   -----> Java Buildpack Version: v3.13 (offline) | https://github.com/cloudfoundry/java-buildpack.git#03b493f
   -----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86_64/openjdk-1.8.0_121.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.7s)
   -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry.org/memory-calculator/trusty/x86_64/memory-calculat
or-2.0.2_RELEASE.tar.gz (found in cache)
          Memory Settings: -Xms681574K -XX:MetaspaceSize=104857K -Xss349K -Xmx681574K -XX:MaxMetaspaceSize=104857K
   -----> Downloading Container Certificate Trust Store 2.0.0_RELEASE from https://java-buildpack.cloudfoundry.org/container-certificate-trust-store/container-c
ertificate-trust-store-2.0.0_RELEASE.jar (found in cache)
          Adding certificates to .java-buildpack/container_certificate_trust_store/truststore.jks (2.0s)
   -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.org/auto-reconfiguration/auto-reconfiguration-1.10.0_R
ELEASE.jar (found in cache)
   -----> Downloading Tomcat Instance 8.0.41 from https://java-buildpack.cloudfoundry.org/tomcat/tomcat-8.0.41.tar.gz (found in cache)
          Expanding Tomcat Instance to .java-buildpack/tomcat (0.2s)
   -----> Downloading Tomcat Lifecycle Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-lifecycle-support/tomcat-lifecycle-support-2.5.
0_RELEASE.jar (found in cache)
   -----> Downloading Tomcat Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-logging-support/tomcat-logging-support-2.5.0_RELE
ASE.jar (found in cache)
   -----> Downloading Tomcat Access Logging Support 2.5.0_RELEASE from https://java-buildpack.cloudfoundry.org/tomcat-access-logging-support/tomcat-access-loggi
ng-support-2.5.0_RELEASE.jar (found in cache)
   Exit status 0
   Staging complete
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploading droplet...
   Uploaded build artifacts cache (108B)
   Uploaded droplet (77.3M)
   Uploading complete
   Destroying container

Waiting for app to start...

name:              spring-music
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            spring-music-patient-roan.local.pcfdev.io
last uploaded:     Sat 15 Sep 14:35:26 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10.0_RELEASE...
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                   -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100%
                   -stackThreads=300 -totMemory=$MEMORY_LIMIT) &&  JAVA_HOME=$PWD/.java-buildpack/open_jdk_jre JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY
                   -Djavax.net.ssl.trustStore=$PWD/.java-buildpack/container_certificate_trust_store/truststore.jks
                   -Djavax.net.ssl.trustStorePassword=java-buildpack-trust-store-password -Djava.endorsed.dirs=$PWD/.java-buildpack/tomcat/endorsed
                   -Daccess.logging.enabled=false -Dhttp.port=$PORT" exec $PWD/.java-buildpack/tomcat/bin/catalina.sh run

     state     since                  cpu     memory           disk             details
#0   running   2018-09-15T09:06:42Z   60.8%   304.4M of 768M   158.2M of 512M


C:\Users\Naman Gupta\Desktop\02\07\applications>
```

We will explore interface of our application and then bind and see changes on application as well then we see the database
we configured.

![Before binding service](images/PreMongoService.PNG?raw=true)

Now we will create one instance of our application and then bind it with spring music and then restart application.
After that we will see the changes.

```cmd
C:\Users\Naman Gupta\Desktop\02\07\applications>cf restart mongo-service-broker
Restarting app mongo-service-broker in org pcfdev-org / space pcfdev-space as user...

Stopping app...

Waiting for app to start...

name:              mongo-service-broker
requested state:   started

C:\Users\Naman Gupta\Desktop\02\07\applications>cf create-service MongoDB-Naman standard music-db
Creating service instance music-db in org pcfdev-org / space pcfdev-space as user...
OK

C:\Users\Naman Gupta\Desktop\02\07\applications>cf services
Getting services in org pcfdev-org / space pcfdev-space as user...

name       service         plan       bound apps   last operation
music-db   MongoDB-Naman   standard                create succeeded

C:\Users\Naman Gupta\Desktop\02\07\applications>cf bind-service spring-music music-db
Binding service music-db to app spring-music in org pcfdev-org / space pcfdev-space as user...
OK
TIP: Use 'cf restage spring-music' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\07\applications>cf restart spring-music
Restarting app spring-music in org pcfdev-org / space pcfdev-space as user...

Stopping app...

Waiting for app to start...

name:              spring-music
requested state:   started
instances:         1/1
usage:             768M x 1 instances
routes:            spring-music-patient-roan.local.pcfdev.io
last uploaded:     Sat 15 Sep 14:35:26 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE spring-auto-reconfiguration=1.10.0_RELEASE...
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculator-2.0.2_RELEASE
                   -memorySizes=metaspace:64m..,stack:228k.. -memoryWeights=heap:65,metaspace:10,native:15,stack:10 -memoryInitials=heap:100%,metaspace:100%
                   -stackThreads=300 -totMemory=$MEMORY_LIMIT) &&  JAVA_HOME=$PWD/.java-buildpack/open_jdk_jre JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh $CALCULATED_MEMORY
                   -Djavax.net.ssl.trustStore=$PWD/.java-buildpack/container_certificate_trust_store/truststore.jks
                   -Djavax.net.ssl.trustStorePassword=java-buildpack-trust-store-password -Djava.endorsed.dirs=$PWD/.java-buildpack/tomcat/endorsed
                   -Daccess.logging.enabled=false -Dhttp.port=$PORT" exec $PWD/.java-buildpack/tomcat/bin/catalina.sh run

     state     since                  cpu     memory           disk             details
#0   running   2018-09-15T10:15:26Z   58.5%   269.9M of 768M   158.2M of 512M

C:\Users\Naman Gupta\Desktop\02\07\applications>
```

We had created service and also bind with our application let's see the changes.

![After binding service](images/PostMongoService.PNG?raw=true)

Now let's observe these in mongoDB.

1. Service instance(database) maintained in the database "mongodb-service-broker.serviceInstance"

![Service Instance](images/serviceInstanceMongoDB.PNG?raw=true)

2. Application binding maintained in the database "mongodb-service-broker.serviceInstanceBinding" with app-id

![Service Binding Instance](images/serviceInstanceBindingMongoDB.PNG?raw=true)

3. Application data in database configured within service

![Application Data](images/applicationDataMongoDB.PNG?raw=true)