# Pushing application to PCF
Once you complete with PCF dev setup, you will see screen below:

![PCF Dev Start](images/cfDevStart.jpg?raw=true)

You can also see the Apps Manager on given URL which looks like below:

![PCF Dev Start](images/appsManager.jpg?raw=true)

We have 1 Org and 1 Apps. Now we start deploying different applications and see how runtime (buildpacks) will be help 
to quickly setup runtime and setup application.

First, we start command prompt and target to API which manages our PCF instance for this case we got API 
https://api.local.pcfdev.io. We also use  --skip-ssl-validation to skip SSL validation, since it is the test environment.
Refer below screen for this:

![PCF Dev Start](images/login.jpg?raw=true)

As in above screen you can see that we targeted the API and then perform login operation using admin/admin which is
provided in local setup of PCF dev. Also we selected the org. Since our org has one space our application will be deployed
in that space only.

Use **"cf target"** command to see which API, Org and space is targeted for pushing current application. 

Let's start pushing some applications:<br/>
To push the application, we use command **cf push <application_name> -m <memory>**. We also pass **--random-route** argument
to generate a URI for application. Let's understand Below output.

1. We are in node directory and pushing the app with 128 MB memory and with instruction to create random route to application
2. It will create the app with name node in our space and start uploading current directory files
3. Then in staging phase it download buildpack after detecting application
and runtime it require to run application.
4. Then it will create droplet. A droplet is the application and it's runtime. After creating droplet it uploads to PCF.
5. It spins up a container for application.
6. After deploying the application it shows instance and it's information

```cmd
C:\Users\Naman Gupta\Documents\PCFPractice\node>cf push node -m 128M --random-route
Pushing app node to org pcfdev-org / space pcfdev-space as admin...
Getting app info...
Creating app with these attributes...
+ name:       node
  path:       C:\Users\Naman Gupta\Documents\PCFPractice\node
+ memory:     128M
  routes:
+   node-tired-panda.local.pcfdev.io

Creating app node...
Mapping routes...
Comparing local files to remote cache...
Packaging files to upload...
Uploading files...
 615 B / 615 B [====================================================================================================================================] 100.00% 1s

Waiting for API to complete processing files...

Staging app and tracing logs...
   Downloaded java_buildpack (244.5M)
   Downloading php_buildpack...
   Downloaded ruby_buildpack (309.4M)
   Downloading staticfile_buildpack...
   Downloaded staticfile_buildpack (5.3M)
   Downloading binary_buildpack...
   Downloaded binary_buildpack (53.7K)
   Downloaded python_buildpack (388.8M)
          Using default npm version: 2.15.11
   -----> Restoring cache
          Skipping cache restore (new runtime signature)
   -----> Building dependencies
          Installing node modules (package.json)
   -----> Caching build
          Clearing previous node cache
          Saving 3 cacheDirectories (default):
          - .npm (nothing to cache)
          - .cache/yarn (nothing to cache)
          - bower_components (nothing to cache)
   -----> Build succeeded!
   Exit status 0
   Staging complete
   Uploading droplet, build artifacts cache...
   Uploading build artifacts cache...
   Uploading droplet...
   Uploaded build artifacts cache (260B)
   Uploaded droplet (9.3M)
   Uploading complete
   Destroying container

Waiting for app to start...

name:              node
requested state:   started
instances:         1/1
usage:             128M x 1 instances
routes:            node-tired-panda.local.pcfdev.io
last uploaded:     Thu 23 Aug 23:40:16 IST 2018
stack:             cflinuxfs2
buildpack:         node.js 1.5.32
start command:     node main.js

     state     since                  cpu    memory         disk           details
#0   running   2018-08-23T18:14:09Z   0.0%   272K of 128M   436K of 512M
```

We can see the app in apps manager.
![PCF Dev Start](images/appsManagerNodeApplication.jpg?raw=true)

