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

## Subscription
Whenever a new account is created there will be a subscription tagged to that.

## Region
Datacenter built around the globes at different locations are called region. There are ~60 regions avaialble for Microsoft Azure. Almost every new resource we create should be allocated to Region. And each physical data centers are called **Zones**. There are possiblities that a Region may contain more than one Zone.

## Paired Region
Concept of where there will be pair region for some Regions. This is because in case if one region failed then other will fulfil this. Pairs are set by Azure and we dont have control over it.

## Resource Group
Resource Group is a logical container for resources. It is used to to contain resource in logical boundary. Every Service need to be bound to resource group. Resource Groups are **Free** to create.

we can create Azure resource group with the help of portal or through command prompt as well

Bash command
```bash
az group create -l centralindia -n CLITest-rg
```
Same activity can be done with Powershell instead of CLI and here is the command for that

Powershell command
```powershell
New-AzResourceGroup -Name PSTest-rg -location centralindia
```

Creating resource group

You can use search bar to search resource group, by typing resource name

#### Region
Selecting region is very important with respect to Region.
- Geographical proximity to System-s audience- Choosing users closest to Target audience
- Services availability.
https://azure.microsoft.com/en-us/explore/global-infrastructure/products-by-region/

- Availability Zone - Support of Availability zones
- Pricing - Pricing changes from region to region
#### Resource Group
- A logic container for resources
- Used for grouping resources by a logic boundary
- Free
- Examples
    - Development/Test/Production resources
    - Team A resources/ Team B resources
> Subscription is also a logical container

https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-setup-guide/organize-resources?tabs=AzureManagementGroupsAndHierarchy

>Management groups are used to manage multiple subscriptions

- Its best practice to have an **rg** or **RG** as part of resource group name
- Could be prefix or suffix
    - RG-Project-Dev
    - Finance-Resources-rg

- Almost every resource in Azure is placed in a Resource Group


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
- Reservations - Certain amount of time.

Always check cost before provisioning

To Check any pricing upfront make sure we use [Azure Calculator](https://azure.microsoft.com/en-in/pricing/calculator/)

Cost management + Billing : Will help to form the budget

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

## Azure Compute
- Set of cloud services for hosting and running applications
- Allows uploading your code and then running it
- Offers various levels of control and flexibility

There are 4 types of compute services
- Virtual Machines
- App Services
- AKS
- Azure Function

### Virtual Machines
- A Virtual server running on a physical server
- Allows creating new servers extremly quick
- Based on existing resources of the physical server
- From the user's point of view - a regular server, nothing new
- Called as **Unmanaged service**
- Eg: Azure wont manage anything we have to manage everything
- **VM Density** - Is number of Virtual machines per host

#### Steps for creating Virtual Machines
- Select the location
- Select the image (OS + Pre-Installed Software)
- Select the size
- *Dont forget to check price*

#### Cost of VM's includes following items
- VM
- Disk
- IP
- Storage

To Check any pricing upfront make sure we use [Azure Calculator](https://azure.microsoft.com/en-in/pricing/calculator/)

#### How to reduce cost
- Auto Shutdown - Point to remember is though VM's are shutdown, IP and Network cost will still incur
    - Reduce more than 50% of the cost
- Reserved Instances - For 1-3 years, will have reduced cost. Mainly used for Production machine.
    - Offers great price and can reduce up to 60% of the listed price
    - Can be divided to pay monthly
    - **Once started can never be reverted back**
- Spot Instances - Machine that run on Unused capacity in Azure. Can be evicted any moment when needed by Azure
    - You will receive very short notification for evicting your machine
    - Great for non critical projects

- Disk optimization - Make sure to select the right disk for the machine. Default is premium SSD - the most expensive option. Non IO- intesive machines can do with Standard SSD. Disk type affects SLA
![DiskType](Disk_Type.png)
    - App servers or Cache server can use Non IO- intensive machines.
    - High disk operation can use Premium SSD
- Always select right size for your machine.
    - CPU shouldn't rest. Make sure CPU utilization should be around 70%.
- Select Linux over windows when possible
- Check price in nearby regions

#### Availability of VM
There are multiple SLA's available.
- With Premium SSD the SLA is 99.99%
- With Standard SSD the SLA is 99.5%
- With standard HDD the SLA is 95%

#### Availability concepts in Azure
There are four availability concepts
- Fault Domain
- Update Domain
- Availability Set
- Availability zone

Fault Domain
- Fault Domain is a logical group of physical hardwares that share a common power source and network switch
- we have to make sure that we select servers from more than one Fault Domain

Update Domain
- Update Domain is a logical group of physical hardwards that goes on server upgrades, rebooting etc
- We have to make sure that we select servers from more than one Update Domain
- Update domain is not like Fault domain, but it is just a logical grouping. Doesnt necessarily has to be at the same rack like Fault domain


Availability Set
- A collection of Fault domain and Update domains a virtual machine will be spread across
- Can contain upto 3 Fault domains and up to 20 update domains 
- All domains (Fault and Update) domain in same location
- Deploy identical VM's in to the same Availaiblity set
- If needed - deploy load balancers to route between the VMs
- Availability set is free, you pay only for the additional PM's

Availaiblity Zone
- A building contain autonomus data center. 
- A physically seperate zone
- Each zone functions as a fault & update domain
- Provides protection against a complete zone shutdown - Better SLA
- Deploy identitcal VMs into seperte Availabiltiy zone in the same region
- Ensure they wont shut down simultanteously when the zone is shut down
- If needed - deploy load balancer to route between the VMs



#### Creating Available and Cost effective VM
- Go to search and type Virtual Machine
- Create a new one
-  create Resource Group
- Create VM Name, Region, Availabiltiy Option
- Select Availability option as Availaiblity Zone
- If availability zone is not there create a new one. When creating make sure you have selected Fault Domain as 2 approx and Update Domain as 3 approx for non production environment
- Availability Zones will not be there for all the regions
- If Availability zone is not present for a particular region then selected Availability set
- Select Operating system (For eg: Windows server 2019 Data center) , then Select Vcpus (2) and RAM (8gig) etc...
- Provide admin user name and password
- then instead of creating VM directly go to disk section.
- Remember that by default Premium SSD will be selected. We have to make sure that do we really require that.
- In the Management section  go to AutoShutDown and setup the configuration. This will save lot of money
- Before creating there is an option for ARM template to download


#### ARM Template
- Azure Resource Manager Template
- A Json file describing the resources to be created
- used by Azure in all deployments
- Can be exported modified, uploaded deployed
- Can also be created from scratch
- ARM template is a decalarative way of deploying resources
    - Declarative
        - Describes the end result
        - Allows "What-if" operation
        - Can deploy multiple resources at once
        - Can be integrated in CI/CD processes
        - Can be source controlled
    - Imperative
        - Sends instruction to run
        - Error prone
        - Can't be verified
        - Can't be source controlled
        - Suited for quick and dirty operations

> If i want to create some quick virtual machine then i can use Imperative method whereas if you want to create a complete Azure environment then Declaratice method can be used

ARM Template - When we download will have two files, *parameters.json* and *template.json*

If we want to deploy the files then we can go to bash command from cloud storage location

```bash
az deployment group create --resource-group optimized-vm-rg --template-file template.json --parameters parameters.json
```
The above command is used to deploy any ARM Templates.

#### Virtual Machine scale set
- A group of seperate VMs sharing the same image
- Managed as a group
- Can be scaled out or in manually or according to predefined condition
- Great for handling unpredictable load
- Once setup the machines should notbe modified
    - Change files, install apps etc
- New machines created by the scale set will be based on the original image
- For web apps, a load balancer should be put in front of the scale set
- Scale set is free
- You pay for the VM's deployed in it

- Scaling menu in scaling is the main point where we configure how many instances and when what should kicks in

Before selecting the vm scale set, first we need to register the resource in ```Subscription->ResourceProviders-> Select the resource and click register``` button
For this scale set to work we have to register ```microsoft.insights```

To check the status of the registered item, try following command in thje bash script

```bash
az provider show --namespace microsft.insights -o table
```

#### Azure Instance Metadata Services
- A little knows feature of Azure VMs
- A REST API accessible from the VM
- Providing a lot of info about the machine
- Info includes:
    - SKU, storage, networking, scheduled events
- Accessible only from the VM

- With Scaleset
    - Get notification about upcoming eviction
- Can be polled every ~1 to get enough time to close things up


Step by Step process
- Create Virtual Machine by default
- Login to Virtual Machine through RDP
- Install Postman inside Virtual Machine
- now in Postman -> Headers -> Put key = Metadata and Value = true
- URL: http://169.254.169.254/metadata/instance?api-version=2021-02-01
- verb: HttpGET

> If we have to get the same details for scheduled events rather than instances then this url
http://169.254.169.254/metadata/instance?api-version=2021-02-01 has to be replaced on top of this url http://169.254.169.254/metadata/instance?api-version=2021-02-01

- The actual Link to the page can be found here
[Azure Instance Metadata Service URL](https://learn.microsoft.com/en-us/azure/virtual-machines/instance-metadata-service?tabs=windows)


#### Setting up the Catalog App
First lets go to the app and publish it with following command

```bash
dotnet publish -o publish
```
`-o` mention that output to and `publish` mention the folder to publish the files

Step by Step
- Create a new Virtual Machine with default configuration , choose the machine type wisely. We dot have to go for higher configuration CPU's. 
- Change disk type from Premium SSD to `Standard SSD`
- move to networking tab and create a good name based on resource group name without -rg and give `-vnet`
- And also we need to select the ip configuration to point to Static otherwise the ip keep changing
- Move to Management tab - Enable Auto shutdown
- Then `Review + Create`

>On the newly created Virtual machine in order to deploy the application we need to install IIS first. For that we have to got here and add features using Add Roles and Features in Server Manager.

Steps to do inside new Virtual Machine
- Open Server
- Go to Server Manager
- Click Local Server
- Click IE Enhanced Security Configuration in the main screen
- Click on Off for both the items shown in popup . This will avoid unnecessary popups in IE
- Install IIS through Add Roles and Features from Server Manager Page
- Click on the next,next and select the instance where we are going to install the feature and in the feature section select Web Server (IIS)
- Once it is installed then we are all set.
- Check that by typing localhost in browser
- If required install Google chrome
- Then install .net 6 - Hosting Bundle. This we can see when we go to all .NET 6 releases and select hosting bundle and not .NET Run time/ .NET SDK

Steps to deploy node application in Unix VM
- Create Unix VM
- Open Putty
- Place the Server ip address in that
- It will prompt you for username and password
- Then following commands has to be executed one by one
```bash
sudo apt install git # To install git
sudo apt update # To update everything to latest
sudo apt install nodejs # To install nodejs
sudo git clone https://github.com/memilavi/WeatherAPI.git # To clone the project
cd Weather API $ To see the code directory
ls # To list down all files iniside the Directory
sudo apt install npm # To install node modules
cd node_modules # To get inside the directory
ls # To see files
npm start # To start the application
```
>The above configuration is for node js application

#### Virtual Machines Tips and Tricks
- If your VM needs more disks, in addition to the default one provided by Azure, then go to the Disks page for that, and add whatever disk you need. Don't forget that disks have costs, and check it in the calculator beforehand.

- Want to backup your VM so that it can be restored in a case of failure? Check the Backup page, where you can define the frequency of the backup and the retention period.

- You can define DNS name for the VM, so that it will be accessible not just using its IP. This can be done by clicking the DNS Name: Configure link in the Overview page.


> Azure Architecture diagram icons can be downloaded from this link [Architecture diagram icons](https://learn.microsoft.com/en-us/azure/architecture/icons/)

> VM's can be accessed from internet if someone knows public IP then they can RDP this

> Shutdown the VM's when not in use

#### App Services
- A fully managed web hosting for websites
- Publish your code - and it just runs
- No access to the underlying servers
- Secured and Compliant  - Microsoft is responsible for security and compliances
- Integrates with many source controls and Devops engines
    - Hithub
    - Bitbucket
    - Azure Devops
    - Docker Hub
    - and many more
- Supported Platforms
    - .NET
    - .NET Core
    - Node.JS
    - Java
    - Python
    - PHP
- App service supports containers
- App Types: 
    - Web Apps
    - Web API
    - Web Jobs
- Extremely easy to deploy
    - Develop your app
    - Create Web App (Can be done through the IDE)
    - Publish your code
    - Done
    

**Creating new Azure App Serice**
Creating new azure app service is easy
- Create new App Service
- Provide name field (It should be unique across entire Azure cloud)
- And select the region + other items.
- By default if we are in free tier then we have to stick to that.
- Just Review + Create

**Deploying existing .NET Core application**
- For the project just go to terminal
- write following command
```bash
dotnet publish -o publish
```
- once publish folder is created then right click on the "Deploy to web app"
- once deployed and we are ready

**App Service features**
- In the Development Tools of App Service we have App Service Editor to edit the files and save
- Settings -> Scale Up is to sacle the subscription
- Settings -> To include Auto scaling option to increase capacity up and down. Remember in the free tier we dont have option to Auto scale.
- We can browse the pplication deployed in Default Domain url which is as example
> readit-inventory-vinoth.azurewebsites.net

- By default, App Services can be accessed using http and https. You can make it https only in the TLS/SSL settings in the App Service menu.

- App service can run also batch processes, or continuous jobs, and not only web apps with the request / response  paradigm. This can be done using the WebJobs menu item, where you can upload exe file that will run always, or on scheduled times.

- Want to know the IP address of the App Service? Take a look at the Properties of the page. You can find there the Virtual IP address of the App Service, and also - the Outbound IP addresses. Note the plural - App Service can have more than one outbound IP address.

- Want to know how much storage did you use, and what are the current usage statistics? Go to Quotas for this data.

**Shutting down app service**
With VM's i recommend to use Shut down the VM when not in use
With App Services this is not the case. 
> while you can stop an App Service (using the stop button at the top of the overview page) all it will do is to stop the functionality of the App service but you will still pay for it

Thats an important difference between App Service and a VM.
With VM - you pay when the VM is on
With App Service - The only way to stop paying for it is completely deleting it.

This is another difference betwen VM a nd App Service you should keep in mind.

#### AKS - Azure Kubernetes Services
- Managed Kubernetes on Azure
- Allows deploying containers and managing them using Kubernetes on Azure
- Paying only on the instances (=VMs) used

#### Containers
- Traditional Deployment
    - Code was copied and built on the production server
    - Problems were found on the servers that weren't found in the dev machines
- Thin packaging model
- Packages software, its dependencies, and configuation files
- Can be copied between machines
- Uses the underlying operating system

#### Container vs VM
- VM's run on top of Hypervisor whereas Containers runs on top of Operating System

