# Cloud Technology

## Characteristics of Cloud computing
- On-Demand Self Service
- Borad Network Access
- Resource Pooling
- Rapid Elasticity
- Measured Service

## Overview

Cloud platform helps in many aspects. Elasticity, consistency, security, cost etc. Cloud works based on OpEX model and not CapEX model

#### CapEX
Capital Expense: Where the complete money is invested upfront by just forcasting. But it doesnt necessarily be fully utilised.
> Traditional IT is CapEx oriented. Buying space, server, AC, Maintanence etc..

- Non optimal
- Not flexible


#### OpEX
Operational Expense: Where the amount is paid based on operation performed and not upfront. Which is much cheaper when it comes to enterprise applications.

- Extremely Flexible
- More optimal

## Different types of Cloud Service

#### Iaas - Infrastructure as a service
The cloud provide underlying platform
- Compute
- Networking
- Storage

>  Eg:  Virtual Machines


#### Paas - Platform as a service
The cloud provide platform for running apps
Including: Compute, networking, storage, runtime, environments etc
> Eg : Web apps. We dont have access to Virtual machine for this
#### Saas - Software as a service
Software running in cloud. We dont have to install anything. We can just use part of the software but we will not be able to access any insfrastructure. 

![Cloud Model](cloud-models.png)
Source: https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/strategy/monitoring-strategy

#### Faas - Function as a service
#### Daas - Database as a service
#### IOTaas - IOT as a service
#### AIaas - AI as a service


## Types of cloud
- Public Cloud
    - Cloud setup in public network
    - Managed by large companies
    - Accessed through internet
- Private Cloud
    - Cloud set up in organization's premises
    - Managed by the Orgranization's IT team
    - Accessible only in the organization's network
    - Openshift
- Hybrid Cloud
    - A cloud setup in an organization premises
    - but also connected to public cloud
    - Workload can be seperated between two clouds
    - i.e Sensitive data in the organization premises, public data in the public cloud
    - Usually managed by the public cloud, but not always.
    Eg: Azure Arc, AWS Outposts 
## Region
Datacenter built around the globes at different locations are called region. There are ~60 regions avaialble for Microsoft Azure. Almost every new resource we create should be allocated to Region. And each physical data centers are called **Zones**. There are possiblities that a Region may contain more than one Zone.

## Paired Region
Concept of where there will be pair region for some Regions. This is because in case if one region failed then other will fulfil this. Pairs are set by Azure and we dont have control over it.

## Resource Group
Resource Group is a logical container for resources. It is used to to contain resource in logical boundary. Every Service need to be bound to resource group. Resource Groups are **Free** to create.

we can create Azure resource group with the help of portal or through command prompt as well


```shell
az group create -l centralindia -n CLITest-rg
```
Same activity can be done with Powershell instead of CLI and here is the command for that

```powershell
New-AzResourceGroup -Name PSTest-rg -location centralindia
```

#### Resource Group vs Subscription
Subscription has associated account and cost center. Subscription is an account level container. Resource Groups are logical container for resources.

#### Naming Convention
Resource group has naming convention. Its better to have **rg-** or **-rg** as prefix or suffix respectively

## Storage Account
Used to store almost anything. Used transparently by any service in Azure. Eg, Database backup, VMs, Diagnostic Data.

## SLA
Service Level Agreement - The uptime% of a cloud service.

![Alt text](SLA.png?raw=true "SLA")

There is a way to calculate SLA's using the tool called [UpTime](https://uptime.is)

## Cost
In cloud everything is Cost.
- Per Resource (For eg,VM)
- Per Consumption (For eg, Function Apps)
- Reservations

Always check cost before provisioning

To Check any pricing upfront make sure we use [Azure Calculator](https://azure.microsoft.com/en-in/pricing/calculator/)


## VSCode Extensions for Azure
- Azure Account
- Azure App Service

## Different ways to reduce VM Cost
- **Auto Shutdown**
- **Reserved Instances**
- **Spot Instances** - Using unused server for VM, which can be dropped anytime by Azure with short notice.

## Availability
There are four types of Availability concpets.
![Availability](Availability.png?raw=true "Availability")

#### Fault Domain
If the server is placed in a rack and if there is any network problem to that rack or any physical damage to that rack, the entire servers inside that rack will go down. So making sure the VM's or server spread across multiple racks will sort out Fault Domain problem.

#### Update Domain
Sometimes there is a posibility that domain may undergo maintanence activity. Due to security update or my be of multiple reason. And all the servers for an application is in same logical domain then when it undergoes maintainence activity then the application will go down. So its better to split it across multiple logical domain, so that if one goes on maintanence other will be there to serve request.

![FaultDomain](FaultDomain.png "Fault Domain")

The best way to avoid Fault domain is deploying across different region in Availability zone. This ensure that not all server will shutdown simultaneously.

- Availability Zone
- Availability Set

## Azure Resource Management (ARM) Template
A JSON file which contains complete details about what configuration we have defined while creating a new resource.
It can be exported, imported, modified and reused for creating the clone of resource based on configuration mentioned in ARM template. It is very important to use while automation.

## Resource Proviers
Resource providers are the list of capabilities available for subscriptions. Once narrow down to subscription we have to find resource providers. In that there will be a list of resources which have already capabilities registered. If there is which is not regisitered then we can select that capability and register it. It will take some time to get registration completed. To check the status we can use batch command. 

```shell
az provider show--namespace microsoft.insight -o table
```
