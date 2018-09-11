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