# Pivotal Cloud Foundry Introduction
Let's start with early days of application development and deployment. Initially in traditional IT to deploy an 
application after development, below steps are required:
1. Provision a VM
2. Configuring Networks
3. Configuring runtime for application on VMs 
4. Connecting to all type of sources (Database, Third party systems)

When cloud computing comes into picture, this scenario changed and we have IaaS (Infrastructure as Service), which ease 
work of the configuring the VM and network, and give freedom developers to just take care Runtime, Application, Data 
and Services to deploy. Still it has a lot of stack to take care by developer to provision the application to customer.

To solve all these problems it comes to Cloud Native Platform, where the developer need to focus on application and 
data, which matters most to business and leave rest of things to IaaS and Pivotal Cloud Foundry(PaaS). Below image 
gives more clarity for the same:

![PCF Evolution](images/CloudNativePlatformEvolution.jpg?raw=true)


Cloud foundry is an open source project which provides the PaaS capability, and PCF provides some additional products
 OSS for the cloud foundry.

## Terminologies
PCF uses some terminologies which need to be understand. I will list some of one liner terminologies here with reference
 to detailed explanation link wherever applicable.
 
**1. Org :** An organization is the unit of tenancy in Pivotal Cloud Foundry. Within an enterprise there are typically 
many lines of business. These map to Orgs within Pivotal Cloud Foundry.

**2. Spaces:** Spaces are used to group applications and services. Spaces are generally created according to a project’s
 delivery environments (e.g. test, qa, staging, production).
 

## HandsOn
To perform the hands on over the PCF we need to setup the PCF runtime or use PCF Webservices.

### Local System Setup
1. To setup the PCF at local system, download PCF Dev and set it up on your machine.
    https://network.pivotal.io/products/pcfdev#/releases/88478
2. To setup your PCF Dev and first application refer below link
    https://pivotal.io/platform/pcf-tutorials/getting-started-with-pivotal-cloud-foundry-dev/introduction

### PCF Webservices
PCF provides it's runtime for one org and one application almost free. Please refer below link and signup
to use PCF runtime. Use below link to register and use runtime

https://run.pivotal.io/

Lesson 1 : Pushing application to PCF<br/>
Lesson 2 : Logging and High availability 