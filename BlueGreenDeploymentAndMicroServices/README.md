# Blue Green Deployment
We will cover this topic in two parts and than Hands-On. We will cover below two to understand and
start with Blue Green Deployments.
1. Blue Green Routing
2. Implications on App & Data Model Design

## 1. Blue Green Routing
Till we have deployed the application and scale it. Now the Day 2 problem starts i.e. consider we need to release
new feature in our application withing our SLA which is about zero downtime. Blue-green deployments are a technique for 
deploying updates with zero downtime. 

Cloudfoundry allows us to manage everything about their application: from deployment to the management of routes to an 
application. Till we deployed the articulate application, consider we added a feature in it, and now we wanna deploy
it. We will perform roughly below steps to do that it no down time
1. We will deploy our application with new version with a temporary URL
2. We will do testing over it so we can gain confidence over it
3. Then we map or bind our production URL with the instance of newer version application
4. From Step 3 some of traffic start diverting to newer version application and some of the still over the old version
5. Then we start scaling our newer application so we can get equal traffic on our application.
6. By Step 4 & 5, we get confidence that our newer version application behaving properly.
7. We start scale down our application and maintain with newer version application.
8. When we feel that we can remove our application and remain with newer one only, we can cut down our application by unbinding URL with application.

## 2. Implications on App & Data Model Design
Implication on App or Data Model design points to things need to be take care while developing new feature or fixing bug.
1. No destructive changes at field level or database level. (Never remove a field which is not in use, keep it there in class.
Same as for database never drop a column keep it,even if it is no more in use)
2. Whenever introduce a new field in database keep it nullable or provide default value if not provided.
3. If using serialization in application always provide **serialversionuid** in class, so new version and old version serialize
and deserialization can be in proper manner.
4. Here we will talk about one of the point from 12-Factor app which is admin process. Run admin/management tasks as one-off
processes e.g. migrating data process. 
5. Do make changes idempotent i.e. if we are are copy data to new field, don't delete the column or data in old column.


## Hands-On
For hands-on we are treating same application with different name articulate. Let's see how we can perform deployment in
zero down time.
1. Let's observe our current environment
```cmd
F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>cf apps
Getting apps in org pcfdev-org / space pcfdev-space as admin...
OK

name           requested state   instances   memory   disk   urls
attendee-app   started           1/1         768M     512M   attendee-app-talkative-panda.local.pcfdev.io
articulate     started           1/1         768M     512M   articulate-rested-quokka.local.pcfdev.io

F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>
```

we have one application instance of the articulate application and running on 768M. To simulate the blue green deployment
we will reduce the memory size to 512M, so demo environment can run without hiccups and scale articulate application to 
2 instances. Let's see the environment after this.
```cmd
F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>cf apps
Getting apps in org pcfdev-org / space pcfdev-space as admin...
OK

name           requested state   instances   memory   disk   urls
attendee-app   started           1/1         512M     512M   attendee-app-talkative-panda.local.pcfdev.io
articulate     started           2/2         512M     512M   articulate-rested-quokka.local.pcfdev.io

F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>
```
Below screen show the traffic generation and load before deploying our app version.
![Pre Blue Green Deployment](images/preBlueGreenScreen.png?raw=true)

2. To get some of the information about routes we use
**cf routes** command which we will use in deploying our newer version application.
```cmd
F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>cf routes
Getting routes for org pcfdev-org / space pcfdev-space as admin ...

space          host                           domain            port   path   type   apps           service
pcfdev-space   attendee-app-talkative-panda   local.pcfdev.io                        attendee-app
pcfdev-space   articulate-rested-quokka       local.pcfdev.io                        articulate

F:\Workspaces\JavaWrkps\PivotalCloudFoundryPractice>
```
Record the subdomain (host) for the articulate application.This is our production route. You will use this in the next step.

3. Now we deploy application with version 2 on different URL. To deploy application with URL and attach to attendee 
service. We use below command to deploy with URL.<br/>
**cf push <application_name> -p <artifact_path_like_jar_or_war> -m <provided_memory> -n {{articulate_hostname_temp}} --no-start**<br/>
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf push articulate-v2 -p ./articulate-0.2.jar -m 512M -n
 articulate-rested-quokka-temp --no-start
Pushing app articulate-v2 to org pcfdev-org / space pcfdev-space as admin...
Getting app info...
Creating app with these attributes...
+ name:       articulate-v2
  path:       C:\Users\Naman Gupta\Desktop\02\01\demos\articulate\articulate-0.2.jar
+ memory:     512M
  routes:
+   articulate-rested-quokka-temp.local.pcfdev.io

Creating app articulate-v2...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 661.81 KiB / 661.81 KiB [=====================================================================] 100.00% 1s

Waiting for API to complete processing files...

name:              articulate-v2
requested state:   stopped
instances:         0/1
usage:             512M x 1 instances
routes:            articulate-rested-quokka-temp.local.pcfdev.io
last uploaded:     Thu 06 Sep 12:55:40 IST 2018
stack:             cflinuxfs2
buildpack:
start command:

There are no running instances of this app.


C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf bind-service articulate-v2 attendee-service
Binding service attendee-service to app articulate-v2 in org pcfdev-org / space pcfdev-space as admin...
OK
TIP: Use 'cf restage articulate-v2' to ensure your env variable changes take effect

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf start articulate-v2
Starting app articulate-v2 in org pcfdev-org / space pcfdev-space as admin...

Staging app and tracing logs...
   Downloading dotnet-core_buildpack...
   Downloading java_buildpack...
   Downloading ruby_buildpack...
   Downloading nodejs_buildpack...
   Downloaded dotnet-core_buildpack
   Downloading go_buildpack...
   Downloading python_buildpack...
   Downloaded go_buildpack
   Downloading php_buildpack...
   Downloaded java_buildpack
   Downloading staticfile_buildpack...
   Downloaded ruby_buildpack
   Downloading binary_buildpack...
   Downloaded nodejs_buildpack
   Downloaded php_buildpack
   Downloaded staticfile_buildpack
   Downloaded python_buildpack
   Downloaded binary_buildpack
   Creating container
   Successfully created container
   Downloading app package...
   Downloaded app package (32.8M)
   Staging...
   -----> Java Buildpack Version: v3.13 (offline) | https://github.com/cloudfoundry/java-buildpack.git#03b49
3f
   -----> Downloading Open Jdk JRE 1.8.0_121 from https://java-buildpack.cloudfoundry.org/openjdk/trusty/x86
_64/openjdk-1.8.0_121.tar.gz (found in cache)
          Expanding Open Jdk JRE to .java-buildpack/open_jdk_jre (2.1s)
   -----> Downloading Open JDK Like Memory Calculator 2.0.2_RELEASE from https://java-buildpack.cloudfoundry
.org/memory-calculator/trusty/x86_64/memory-calculator-2.0.2_RELEASE.tar.gz (found in cache)
          Memory Settings: -Xss349K -Xmx681574K -Xms681574K -XX:MetaspaceSize=104857K -XX:MaxMetaspaceSize=1
04857K
   -----> Downloading Container Certificate Trust Store 2.0.0_RELEASE from https://java-buildpack.cloudfound
ry.org/container-certificate-trust-store/container-certificate-trust-store-2.0.0_RELEASE.jar (found in cache
)
          Adding certificates to .java-buildpack/container_certificate_trust_store/truststore.jks (1.1s)
   -----> Downloading Spring Auto Reconfiguration 1.10.0_RELEASE from https://java-buildpack.cloudfoundry.or
g/auto-reconfiguration/auto-reconfiguration-1.10.0_RELEASE.jar (found in cache)
   Exit status 0
   Staging complete
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploading droplet...
   Uploaded build artifacts cache (109B)
   Uploaded droplet (78.3M)
   Uploading complete
   Destroying container
   Successfully destroyed container

Waiting for app to start...

name:              articulate-v2
requested state:   started
instances:         1/1
usage:             512M x 1 instances
routes:            articulate-rested-quokka-temp.local.pcfdev.io
last uploaded:     Thu 06 Sep 12:55:40 IST 2018
stack:             cflinuxfs2
buildpack:         container-certificate-trust-store=2.0.0_RELEASE
                   java-buildpack=v3.13-offline-https://github.com/cloudfoundry/java-buildpack.git#03b493f
                   java-main open-jdk-like-jre=1.8.0_121 open-jdk-like-memory-calculator=2.0.2_RELEASE
                   spring-auto-reconfiguration=1.10...
start command:     CALCULATED_MEMORY=$($PWD/.java-buildpack/open_jdk_jre/bin/java-buildpack-memory-calculato
r-2.0.2_RELEASE
                   -memorySizes=metaspace:64m..,stack:228k..
                   -memoryWeights=heap:65,metaspace:10,native:15,stack:10
                   -memoryInitials=heap:100%,metaspace:100% -stackThreads=300 -totMemory=$MEMORY_LIMIT) &&
                   JAVA_OPTS="-Djava.io.tmpdir=$TMPDIR
                   -XX:OnOutOfMemoryError=$PWD/.java-buildpack/open_jdk_jre/bin/killjava.sh
                   $CALCULATED_MEMORY
                   -Djavax.net.ssl.trustStore=$PWD/.java-buildpack/container_certificate_trust_store/trustst
ore.jks
                   -Djavax.net.ssl.trustStorePassword=java-buildpack-trust-store-password" &&
                   SERVER_PORT=$PORT eval exec $PWD/.java-buildpack/open_jdk_jre/bin/java $JAVA_OPTS -cp
                   $PWD/. org.springframework.boot.loader.JarLauncher

     state     since                  cpu     memory           disk             details
#0   running   2018-09-06T07:29:08Z   73.5%   281.5M of 512M   159.8M of 512M

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
![Version 2 application screen](images/v2AppScreen.png?raw=true)

At this point in the deployment process, you could do further testing of the version you are about to release before 
exposing customers to it.

4. Let’s assume we are ready to start directing production traffic to version 2. We need to map our production route 
to articulate-v2. We will use below command to directing our traffic to application.<br/>
**cf map-route <application_name> <domain_name> --hostname <articulate_hostname>**
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf map-route articulate-v2 local.pcfdev.io --hostname articulate-rested-quokka
Creating route articulate-rested-quokka.local.pcfdev.io for org pcfdev-org / space pcfdev-space as admin...
OK
Route articulate-rested-quokka.local.pcfdev.io already exists
Adding route articulate-rested-quokka.local.pcfdev.io to app articulate-v2 in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf apps
Getting apps in org pcfdev-org / space pcfdev-space as admin...
OK

name            requested state   instances   memory   disk   urls
attendee-app    started           1/1         512M     512M   attendee-app-talkative-panda.local.pcfdev.io
articulate      started           2/2         512M     512M   articulate-rested-quokka.local.pcfdev.io
articulate-v2   started           1/1         512M     512M   articulate-rested-quokka.local.pcfdev.io, articulate-rested-quokka-temp.local.pcfdev.io

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
As seen example articulate old version has 2 instances and articulate version v2 has 1 instance. Also V2 is mapped to 
both URL temp and production URL. Now we start generating load again and as we can see from below screen load is distributed
equally 33% around.
![Single instance load distribution](images/OneInstanceDeployNewApplication.png?raw=true)

5. We scale our version 2 application and see the load distribution.
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf scale articulate-v2 -i 2
Scaling app articulate-v2 in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
![Two instance load distribution](images/TwoInstanceDeployNewApplication.png?raw=true)

6. Scale down the old application to instance 1.
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf scale articulate -i 1
Scaling app articulate in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
![Two instance new and old one instance load distribution](images/TwoInstanceDeployNewApplicationOneOldApplication.png?raw=true)

7. Now we have confidence that our application is working properly and we can remove old application. To do that Remove 
the production route from the articulate application. To do that we use below command<br/>
**cf unmap-route <application_name> <domain_name> --hostname <articulate_hostname>**
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf unmap-route articulate local.pcfdev.io --hostname articulate-rested-quokka
Removing route articulate-rested-quokka.local.pcfdev.io from app articulate in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf apps
Getting apps in org pcfdev-org / space pcfdev-space as admin...
OK

name            requested state   instances   memory   disk   urls
attendee-app    started           1/1         512M     512M   attendee-app-talkative-panda.local.pcfdev.io
articulate      started           1/1         512M     512M
articulate-v2   started           2/2         512M     512M   articulate-rested-quokka.local.pcfdev.io, articulate-rested-quokka-temp.local.pcfdev.io

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
As we can see the old application has no route to access. We need to remove the Temp URLs from the version 2 application. 
```cmd
C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf unmap-route articulate-v2 local.pcfdev.io --hostname articulate-rested-quokka-temp
Removing route articulate-rested-quokka-temp.local.pcfdev.io from app articulate-v2 in org pcfdev-org / space pcfdev-space as admin...
OK

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>cf apps
Getting apps in org pcfdev-org / space pcfdev-space as admin...
OK

name            requested state   instances   memory   disk   urls
attendee-app    started           1/1         512M     512M   attendee-app-talkative-panda.local.pcfdev.io
articulate      started           1/1         512M     512M
articulate-v2   started           2/2         512M     512M   articulate-rested-quokka.local.pcfdev.io

C:\Users\Naman Gupta\Desktop\02\01\demos\articulate>
```
Now the application is completely deployed using blue green deployment and in zero downtime.
![Completely Deployed](images/CompletelyDeployed.png?raw=true)


## For more reading on Blue Green deployment.
https://docs.pivotal.io/pivotalcf/2-2/devguide/deploy-apps/blue-green.html 

