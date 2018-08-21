# Pivotal Cloud Foundry Introduction
Let's start with early days of application development and deployment. Intially in traditional IT to deploy an application after development, below steps are required:
1. Provision a VM
2. Configuring Networks
3. Configuring runtime for application on VMs 
4. Connecting to all type of sources (Database, Third party systems)

When cloud computing comes into picture, this senerio changed and we have IaaS (Infrastructure as Service), which ease work of the configuring the VM and network, and give freedom developers to just take care Runtime, Application, Data and Services to deploy. Still it has a lot of stack to take care by developer to provision the application to customer.

To solve all these problems it comes to Cloud Native Platform, where the developer need to focus on application and data, which matters most to business and leave rest of things to Iaas and Pivotal Cloud Foundry(Paas). Below image gives more clarity for the same:

![PCF Evolution](images/CloudNativePlatformEvolution.jpg?raw=true)
