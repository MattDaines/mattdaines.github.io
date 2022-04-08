+++
author = "Matt Daines"
title = "Using Private Link Services for Connectivity Between Azure Tenants"
date = "2022-02-23"
description = "Using Private Link Services to enable private connectivity between a service in two or more Azure tenants."
tags = [
    "Azure Networking",
    "Azure Platform"
]
categories = [
    "Azure",
    "Azure Architecture",
    "Azure Networking"
]
series = ["azure-private-connections"]
aliases = ["azure-private-connections"]
+++

Hey there! In this post I'm going to talk about the design and implement of Private Link Services to enable private communication between a client in one Azure tenant to a service in one another tenant.
<!--more-->

### What is a Private Link?

First, the fundamentals. Private Link enables you to create Private **Endpoints** for PaaS services (Key Vault, Storage Accounts, etc) within your Azure virtual networks. This means services that are deployed or integrated within your Azure virtual network can communicate with that PaaS service over an endpoint that cannot be reached from the Internet. Using private endpoints to reduce the attack surface is quite common to contribute to a Defence in Depth strategy.

Private Link Services are another feature of Private Link that enables _you_ to provide a service and allow others to _subscribe_ to it over a private endpoint in their own network. Connections to a private endpoint never leave Microsoft's backbone over the Internet, even when the private endpoint is in a different Azure region to where the private link service is hosted.

One example of private link services is a SaaS provider that enables you to _subscribe_ to their private link service and enabling you to create a private endpoint within your Azure virtual network. A second example is when you deploy Azure Red Hat OpenShift (a Kubernetes managed cluster) it also creates a private link service and an endpoint that the cluster nodes use for reporting and logging back to Red Hat.

### When would I use a Private Link Service?

Software-as-a-Service is so incredibly broad that this private link service shaped shoe does not fit all. But there are certainly some providers where private link services could be beneficial. For example, if your service requires an agent to be installed on virtual machines to send data or telemetry back to your service also in Azure. If this connection is over a public endpoint that could be a deal breaker for regulated customers.

Some Azure customers opt for a multi-tenant deployment, for one reason or another. Private link services can enable systems/services in one of your tenants to consume an application in another potentially mitigating additional licence, compute, storage, and operational costs. If the customer has opted for multi-tenant, then that likely rules out virtual network peering (which, to my recent surprise, is possible between tenants!)

And for my third use case maybe your virtual networks are deployed as _islands_ (virtual networks that have no centralised connectivity) within a single tenant. Quite often these island virtual networks have overlapping address space which is problematic when a service in one virtual network needs to consume from a service in another virtual network as peering cannot be used. Private link services NATs the connection into the provider’s virtual network. So, from the providers point of view the connection _appears_ to come from their own network and not the subscriber’s network.

As the provider you can see the subscriber’s IP if you configure the TCP V2 proxy however your application will need to support the headers. If your application cannot parse the headers, then the connection will not be allowed. More information on [Microsoft Docs](https://docs.microsoft.com/en-gb/azure/private-link/private-link-service-overview#getting-connection-information-using-tcp-proxy-v2).

### Setting Up a Private Link Service

Right, now the hands on section! But first lets paint the picture of our scenario. The non-tech part is really a pick your own story:

- A single organisation uses multiple tenants to separate their pre-production and production environments. However, the Production environment needs to occasionally pull data from the pre-production system.
- An organisation has recently acquired another company and needs to quickly integrate systems between services in two tenants.
- Developers within an organisation deploy their applications to virtual network pods/islands, all configured with `10.0.0.0/16` networks but need to send health probes to another service with overlapping address space.

Fundamentally, what we're going to setup is a `Provider` virtual machine that's running IIS and a `Subscriber` virtual machine in another virtual network in another tenant. After all is configured the `subscriber` virtual machine will be able to see the IIS site without using public domain names or public IP addressing.

If you're familiar with the process of the steps detailed below feel free to skip over the details. I've summarised what you need at the top of each section and then go into detail for each step to support any beginners that come across the article.

**Setting Up The Infrastructure**

1) [Setup the Provider Virtual Network](#create-the-provider-virtual-network)
2) [Create a Windows Azure Virtual Machine](#create-the-provider-virtual-machine)
3) [Create the Subscriber Infrastructure](#create-the-subscriber-infrastructure)
4) [Install IIS and open HTTP](#configure-the-provider-virtual-machine)
5) [Create an internal Azure load balancer](#create-and-configure-an-internal-load-balancer-for-the-provider-service)
6) [Create a Private Link Service](#create-your-provider-private-link-service)

**Request The Private Link Service**

7) [Requesting Access to a Private Link Service](#requesting-access-to-a-private-link-service)
8) [Provider Approval](#provider-approval)

**Testing the Connection**

9) [Loading IIS from the Subscriber tenant/network](#testing-the-connection)

#### Create the Provider Virtual Network

Quick Setup Steps:

- Create a virtual network with two subnets named:
    - `AzureBastionSubnet`
    - `Provider`
- Deploy a `basic` sku Azure Bastion to the virtual network.
- The address space you use doesn't matter. It can overlap with your subscriber network when we get to provisioning that.

[Next Step - Create the Provider VM](#create-the-provider-virtual-machine) OR follow the detailed steps below:

![Create a new virtual network.](imgs/01-CreateVirtualNetwork/01-Create.png)

![Create a new resource group for the provider resources (PrivateLinkService is a more fitting name, I made a typo!)](imgs/01-CreateVirtualNetwork/02-NewRG.png)

![My virtual network basics tab](imgs/01-CreateVirtualNetwork/03-Create-Basics.png)

![Edit the default subnet and rename it to "provider", or similar. The address space doesn't matter as long as there's room for one more subnet.](imgs/01-CreateVirtualNetwork/04-Create-IPAddress-ProviderSnet.png)

![Add a new subnet named "AzureBastionSubnet". Address space again doesn't matter however AzureBastionSubnet requires at least a /26 address.](imgs/01-CreateVirtualNetwork/05-Create-IPAddress-BastionSnet.png)

![If you're following along you should now have a virtual network using /16 address space and two /24 subnets.](imgs/01-CreateVirtualNetwork/06-Create-IPAddresses.png)

![I've left everything as disabled. I'll deploy Azure Bastion after the virtual network has been created.](imgs/01-CreateVirtualNetwork/07-Create-Security.png)

![(Optional) Add any tags you require or want. I've added a tag to help me filter this demo's resources.](imgs/01-CreateVirtualNetwork/08-Create-Tags.png)

![My overview page for my new virtual network. If everything looks correct select create!](imgs/01-CreateVirtualNetwork/09-Create-Review.png)

![Once the virtual network has been created you'll need to create a new Azure Bastion service.](imgs/02-CreateBastion/01-Create.png)

![Deploy Bastion into the provider resource group into the network you created in the step before. To save on costs I've chosen a basic sku of Azure Bastion.](imgs/02-CreateBastion/02-Create-BastionProperties.png)

![(Optional) Added my Demo tag to Bastion's resources](imgs/02-CreateBastion/03-Create-BastionTags.png)

![At the time of writing there are no advanced options supported for the basic sku of Azure Bastion so we can skip over this tab.](imgs/02-CreateBastion/04-Create-BastionAdvanced.png)

![The Bastion that I'm creating. Azure Bastion took just under 6 minutes to deploy however I'm sure historically it used to be longer. You can move onto the next step while you wait.](imgs/02-CreateBastion/05-Create-BastionReview.png)

#### Create the Provider Virtual Machine

Quick Setup Steps:

- Create a Windows Server virtual machine capable of installing the IIS role.
- The virtual machine should not have a public IP address.

[Next Step - Create the Subscriber Infrastructure](#create-the-subscriber-infrastructure) OR follow the detailed steps below:

![Go to Virtual Machines and create an Azure Virtual Machine.](imgs/03-CreateProviderVM/01-Create.png)

![Name your virtual machine, I named mine provider-vm. For the image I selected "Windows Server 2022 Datacenter: Azure Edition - Gen 2", not sure why Portal isn't displaying that! As this is a demo we have no need to configure availability redundancy.](imgs/03-CreateProviderVM/02-Create-ProviderVM-Basics01.png)

![Enter the credentials you'll use to connect to the virtual machine. You'll need these later! I've also selected no public inbound ports.](imgs/03-CreateProviderVM/02-Create-ProviderVM-Basics02.png)

![Change your OS disk type to standard SSD, just to save on costs! No need to create additional data disks.](imgs/03-CreateProviderVM/03-Create-ProviderVM-Disks.png)

![Create the virtual machine in the provider network you created in the previous step. Remove the public IP address, we're only using private IP addressing in this demo. I've selected no network security group as I prefer associating them to subnets. Though, I haven't applied a network security group to either network interface or my subnet in this demo as we're only using private IP addressing. (This despite my inner self telling me I should just add the NSG into the guide 🙃)](imgs/03-CreateProviderVM/04-Create-ProviderVM-Networking.png)

![I disabled boot diagnostics. If you prefer to enable boot diagnostics, go ahead. It won't affect the demo.](imgs/03-CreateProviderVM/05-Create-ProviderVM-Management01.png)

![Even though the virtual machine won't be around for long I've selected manual updates for patch orchestration. This setting doesn't affect the demo so any option is a good choice.](imgs/03-CreateProviderVM/05-Create-ProviderVM-Management02.png)

![We can skip over the Advanced tab as there is nothing we need to do there. I did however apply my tags!](imgs/03-CreateProviderVM/06-Create-ProviderVM-Tags.png)

![My VM create review page 1/3.](imgs/03-CreateProviderVM/07-Create-ProviderVM-Review01.png)

![My VM create review page 2/3.](imgs/03-CreateProviderVM/07-Create-ProviderVM-Review02.png)

![My VM create review page 3/3.](imgs/03-CreateProviderVM/07-Create-ProviderVM-Review03.png)

#### Create the Subscriber Infrastructure

Quick Setup Steps:

- Create another virtual network in either a different tenant or the same tenant as your provider with two subnets named:
    - `AzureBastionSubnet`
    - `Subscriber`
- Provision Azure Bastion to this new virtual network
- Deploy a virtual machine capable of launching a browser (OS agnostic)

[Next Step - Configure the Provider Virtual Machine](#configure-the-provider-virtual-machine) OR follow the detailed steps below:

To setup the provider infrastructure you need to repeat [Create the Provider Virtual Network](#create-the-provider-virtual-network) and [Create the Provider Virtual Machine](#create-the-provider-virtual-machine) but swap provider for subscriber when setting names.

This subscriber/client infrastructure can be in the same tenant or a different tenant to the provider and can have overlapping address space with the provider's virtual network. I'll be using overlapping address space with my demo. If you're doing the same double check that the address space is set as `10.0.0.0/16`. If you're deploying to the same subscription as your provider Azure might detect this space is already used and default to `10.1.0.0/16`.

![My deployed subscriber resources in a second Azure subscription.](imgs/04-CreateSubscriberResources/01-DeployedResources.png)

#### Configure the Provider Virtual Machine

Quick Setup Steps:

- Using Bastion, connect to the `provider-vm`
- Install the IIS role with default options
- Open port 80

[Next Step - Create and configure an internal load balancer for the Provider Service](#create-and-configure-an-internal-load-balancer-for-the-provider-service) OR follow the detailed steps below:

![In Server Manager select Manage -> Add Roles and Features.](imgs/05-InstallIIS/01-InstallRolesAndFeatures.png)

![Click next through to the Server roles section and tick Web Server (IIS) and add the optional management features when prompted.](imgs/05-InstallIIS/02-EnableIIS.png)

![Click next through to completion and install the role and features.](imgs/05-InstallIIS/03-Confirmation.png)

![Open Windows Defender Firewall with Advanced Security.](imgs/05-InstallIIS/04-OpenFirewall.png)

![Create a new Inbound Rule.](imgs/05-InstallIIS/05-NewRules.png)

![Select Port as the rule type.](imgs/05-InstallIIS/06-NewRules-Type.png)

![Select TCP and input port 80. We open port 80 because this is the default port for HTTP communications.](imgs/05-InstallIIS/07-NewRules-ProtoPorts.png)

![Select Allow the Connection.](imgs/05-InstallIIS/08-NewRules-Action.png)

![For the demo, enable on all profiles.](imgs/05-InstallIIS/09-NewRules-Profiles.png)

![Add a name and description to your rule. When you're ready, finish create the rule.](imgs/05-InstallIIS/10-NewRules-NameDesc.png)

![To test IIS is running on the server open server manager and go to the local server blade. In the overview section disable IE Enhanced Security Configuration.](imgs/05-InstallIIS/11-DisableIEEnhancedSecConfig01.png)

![Disable IE Enhanced Security Configuration.](imgs/05-InstallIIS/11-DisableIEEnhancedSecConfig02.png)

![Open the Edge/IE (depending on your version of Windows Server) and go to http://localhost. The default page should load.](imgs/05-InstallIIS/12-OpenLocalhost.png)

#### Create and Configure an Internal Load Balancer for the Provider Service

Quick Setup Steps:

- Create a `Standard`, `Internal`, `Regional` Azure load balancer in your provider network
- Add a single frontend in your Provider subnet
- Add the provider virtual machine into the backend pool
- Create a load balancer rule on port 80 with a HTTP:80 health probe

[Next Step - Create your Provider Private Link Service](#create-your-provider-private-link-service) OR follow the detailed steps below:

![In your provider subscription create a load balancer.](imgs/06-ILB/01-Create.png)

![You'll need a Standard, Internal, Regional Azure load balancer.](imgs/06-ILB/02-Create-Basics.png)

![Create a frontend for your load balancer specifying the provider subnet.](imgs/06-ILB/03-Create-Frontend.png)

![Name your backend pool. We'll be adding virtual a network interface card (NIC) to our backend pool. And the demo only uses IPv4.](imgs/06-ILB/04-Create-Backend01.png)

![Add your provider virtual machine into the backend pool.](imgs/06-ILB/04-Create-Backend02.png)

![This is what I have and the end of configuring my backend pool.](imgs/06-ILB/04-Create-Backend04.png)

![Create a load balancing rule balancing on port 80. Floating IP, off screenshot below TCP reset, is set to disabled.](imgs/06-ILB/05-Create-LBR01.png)

![While configuring your load balancer rule create a health probe on port 80 with the HTTP protocol.](imgs/06-ILB/05-Create-LBR02.png)

![No outbound rules need to be configured for this demo.](imgs/06-ILB/06-Create-OBR.png)

![Add any tags that you need or want.](imgs/06-ILB/07-Create-Tags.png)

![My load balancer review page.](imgs/06-ILB/08-Create-Review.png)

#### Create your Provider Private Link Service

Quick Setup Steps:

- In the provider tenant create a `Private Link Service` from the [Private Link Center](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/overview).
- The name of the private link scope forms part of an alias that is shared to enable others to connect to the service. The naming convention looks like: `<name>.<guid>.<region>.azure.privatelinkservice`
- Select the internal load balancer you created earlier.
- For the NAT subnet select the Provider virtual network and subnet. This is the subnet that network interfaces are deployed to for the private link service's outbound connections. At the time of writing, there can be a maximum of 8 NAT IPs.
- Ensure Enable TCP Proxy V2 is set to **No**.
- Private IP Address settings is where you configure how many NAT interfaces to deploy. For the demo, I've left just one interface.
- For this demo we're testing (or at least simulating) a multi-tenant connection. So I've chosen the `Restricted by subscription` option for access.
- Using `Anyone with your alias` means you cannot setup auto-approvals. All approvals need to be approved when using anyone. `Restricted by subscription` allows you to configure auto-approvals from known and expected subscriptions.
- Once deployed, copy the alias into your clipboard. You'll need it in the next step!

[Next Step - Requesting Access to a Private Link Service](#requesting-access-to-a-private-link-service) OR follow the detailed steps below:

![Go to the Private Link Center in the Azure Portal and create a Private Link Service.](imgs/07-PLS/01-Create.png)

![Enter the name for your private link service. The name that's entered forms part of an alias that's shared to enable others to connect to your private link service.](imgs/07-PLS/02-Create-Basics.png)

![Select the load balancer we created earlier and use the Provider subnet to host network interfaces for the services network interfaces. These network interfaces are created to NAT the incoming connections. Up to 8 can be created per private link service and this scaled with outbound flows. The number of interfaces is configured under Private IP Address Settings. For the demo we won't enabled V2 proxy. The V2 proxy allows the provider to see the private IP address of the connecting clients though the application needs to support a specific configuration that we're not covering in this document.](imgs/07-PLS/03-Create-Outbound.png)

![The descriptions on the Portal provide details about the three options available and what they mean. Restrict by Subscription and Anyone with your Alias allows for connections from other tenants. I've selected Restricted by subscription and added my second (subscriber) subscription ID.](imgs/07-PLS/04-Create-AccessSecurity.png)

![Add any tags that you need or want on your Private Link Service.](imgs/07-PLS/05-Create-Tags.png)

![My Private Link Service review page.](imgs/07-PLS/06-Create-Review.png)

#### Requesting Access to a Private Link Service

Quick Setup Steps:

- In the subscriber tenant create a `Private Endpoint` specifying the resource group, name of the private endpoint, and the region to deploy the endpoint to (it should be the same as your subscriber virtual network... Which does not have to be in the same region as your provider!)
- For Connection Method, paste the alias you copied from the Private Link Service in the step before.
- Enter a Request Message, if you'd like.
- Select the virtual network and subnet the private endpoint should be deployed to and select create.
- Once deployed, grab the private IP address of the private endpoint. We'll need it for testing soon!

[Next Step - Provider Approval](#provider-approval) OR follow the detailed steps below:

![Once deployed, go to your Private Link Scope and copy the alias.](imgs/08-Request/01-FindAlias.png)

![In the context of the subscriber (swap tenants, if that's what you're testing). Go to the Private Link Center and create a new private endpoint.](imgs/08-Request/02-CreatePE.png)

![Create the private endpoint in your subscriber virtual network.](imgs/08-Request/03-CreatePE-Basic.png)

![We need to connect via an alias so select option #2. Enter the Private Link Service Alias and a request message, if you'd like. This message goes to the provider if connections are not auto-approved.](imgs/08-Request/04-CreatePE-Resource.png)

![Select the subnet you want to surface the private endpoint. I'm creating this in the subscriber subnet.](imgs/08-Request/05-CreatePE-Network.png)

![My Private Endpoint review page.](imgs/08-Request/06-CreatePE-Review.png)

#### Provider Approval

Quick Setup Steps:

- Back in the provider tenant go to [Pending Connection](https://portal.azure.com/#blade/Microsoft_Azure_Network/PrivateLinkCenterBlade/pendingconnections) in the Private Link Center.
- Approve/Reject the request from the private endpoint created in the step before.
- If you approved that should be reflected in the Private Endpoints blade.

[Next Step - Testing the Connection](#testing-the-connection) OR follow the detailed steps below:

![In the provider subscription we have a pending connection. Check the box and approve the request.](imgs/09-Approval/01-PendingApproval.png)

![Maybe as the provider we should amend the description.](imgs/09-Approval/02-Approval01.png)

![There we go, that's a more fitting description! This is reflected back in the subscribers private endpoint Request/Response as well.](imgs/09-Approval/02-Approval02.png)

![Now we wait an "Azure minute"...](imgs/09-Approval/02-Approval03.png)

![Bingo!](imgs/09-Approval/02-Approval04.png)

#### Testing the Connection

Quick Setup Steps:

- If you don't have it already, find the IP address of the private endpoint in the subscriber network.
- Using Bastion, connect onto your subscriber virtual machine. If you're logging onto a Windows Server box, turn off `IE Enhanced Security Configuration`.
- Launch the browser and navigate to `http://<private endpoint IP>`
- Hopefully, you're looking at ISS default page utilising a Private Link Service connection!

![Find the private IP address of the private endpoint in the subscriber network. My private endpoint address is 10.0.0.5.](imgs/10-Testing/01-FindPrivateIPAddress.png)

![On your subscriber virtual machine open your browser and go to http://10.0.0.5. The subscriber-vm will reach the IIS service running on the provider-vm through the private endpoint created from a Private Link Service!](imgs/10-Testing/02-IISThroughPLS.png)

### Wrapping Up

Questions? Feel free to use Disqus at the bottom of the post, or my socials on [Twitter](https://twitter.com/MatthewDaines1) or [LinkedIn](https://www.linkedin.com/in/matthew-daines/).

If something didn't work how it was supposed to, do let me know and I'll go through and try to replicate and fix the guide.

Hope you found this useful, _catch ya_!

### Trivia and Afterthoughts

- Azure Private Link went Globally Available on the 4th November 2019 ([GA Annoucement](https://azure.microsoft.com/en-us/updates/azure-private-link-is-now-available-in-all-regions/))
- In the screenshots my resource group names are `PrivateLinkScope-Demo-*`. Notice the mistake? It's Private Link Service, not Private Link Scope! I noticed near the end capturing all of the screenshots and there are far too many screenshots to repair. I've had [Azure Monitor Private Link Scopes](https://docs.microsoft.com/en-gb/azure/azure-monitor/logs/private-link-configure) on my mind and made the typo 🤦🏻‍♂️
- I want to investigate what else you can serve behind a Private Link Scope. Can you query DNS? Join a domain? I want to discover it's limitations.
