# Application Security Group (ASG)
Application Security Groups, often abbreviated as ASGs, are a feature designed for cloudfoundry operators.
Nevertheless it’s important for developers to learn what they are and how they work, as it might explain 
why a particular application may in some cases be unable to communicate with a backing service, and 
how to remedy the issue. 
 
ASG is like virtual firewalls which control the egress traffic. There are various level of security 
groups which are divided in different groups. 

Security group applied during lifecycle of application like while stagging or running are called 
staging security group and running security group respectively.Stagging and Running security groups 
are apply system wide, it does not matter in which org or space you are running.

Space security group applied the space in which application is running. 

Security groups can be defined using rules file. The file contains the array of rule consist of 
protocol,IP and ports to allow.

## Hands-on
To see which are the security groups running in targeted space, use below command<br/>
**cf security-groups**
```cmd
C:\Users\Naman Gupta\Desktop\02\application>cf security-groups
Getting security groups as user...
OK

     name              organization   space   lifecycle
#0   all_pcfdev        <all>          <all>   running
     all_pcfdev        <all>          <all>   staging
#1   dns               <all>          <all>   running
     dns               <all>          <all>   staging
#2   public_networks   <all>          <all>   running
     public_networks   <all>          <all>   staging

```
To see a security group detail, shows the rules configured in that ASG.
```cmd
C:\Users\Naman Gupta\Desktop\02\application>cf security-group public_networks
Getting info for security group public_networks as user
OK

Name    public_networks
Rules
        [
                {
                        "destination": "0.0.0.0-9.255.255.255",
                        "protocol": "all"
                },
                {
                        "destination": "11.0.0.0-169.253.255.255",
                        "protocol": "all"
                },
                {
                        "destination": "169.255.0.0-172.15.255.255",
                        "protocol": "all"
                },
                {
                        "destination": "172.32.0.0-192.167.255.255",
                        "protocol": "all"
                },
                {
                        "destination": "192.169.0.0-255.255.255.255",
                        "protocol": "all"
                }
        ]

No spaces assigned
```

To create, modify or delete ASGs we need to login with **ADMIN** then we can perform below hands-on.

If we wanna see the staging and running security groups then we need to use below commands<br/>
**cf running-security-groups**<br/>
**cf staging-security-groups**<br/>
```cmd
C:\Users\Naman Gupta\Desktop\02\application>cf running-security-groups
Acquiring running security groups as 'admin'
OK

public_networks
dns
all_pcfdev

C:\Users\Naman Gupta\Desktop\02\application>cf staging-security-groups
Acquiring staging security group as admin
OK

public_networks
dns
all_pcfdev
```

The following commands support managing this list:<br/>

**cf create-security-group <security_group_name> <path_to_json_file>**<br/>
**cf update-security-group <security_group_name> <path_to_json_file>**<br/>
**cf delete-security-group <security_group_name>**<br/>

The create and update commands require that you supply the security group definition as a JSON file that encodes a list 
of egress rules.

### Bind security groups to staging or running
To bind the security group to our lifecycle of application we use below commands<br/>
**cf bind-running-security-group <security_group_name>**<br/> 
**cf unbind-running-security-group <security_group_name>**<br/>
**cf bind-staging-security-group  <security_group_name>**<br/> 
**cf unbind-staging-security-group  <security_group_name>**<br/>

### Bind security groups specific to Org and Space
To bind the security group to our lifecycle of application we use below commands<br/>
**cf bind-security-group <security_group_name> <org_name> <space_name>**<br/> 
**cf unbind-running-security-group <security_group_name> <org_name> <space_name>**<br/>

### More on security groups
https://docs.pivotal.io/pivotalcf/2-2/concepts/asg.html<br/>
https://docs.pivotal.io/pivotalcf/2-2/concepts/security.html#network-traffic

#Log Drain
Cloud Foundry’s loggregator subsystem routes applications' log streams out of the containers from which they emanate, 
and makes them accessible via the cf logs command.

But what if you needed to maintain a month’s worth of logs, and wanted to leverage third party tools or services for 
log analysis? Cloudfoundry provides the capability to drain application logs to some destination, whether it be an 
internal enterprise system or a third-party service.

For the log draining we have not so much for the hands-on since it need a tool like papertrail or splunk to drain log in
third party systems. Below are steps to setup log draining.
1. Install or create an account for the log aggregator system
2. Create a User Provided Service Instance that streams logs to log system.<br/>
**cf create-user-provided-service articulate-log-drain -l syslog://{{syslog_drain_url}}**
3. Bind the articulate-log-drain service to the articulate application.
**cf bind-service <application_name> <log_service_name>**

### More on log drain
https://docs.pivotal.io/pivotalcf/2-2/devguide/services/log-management.html

