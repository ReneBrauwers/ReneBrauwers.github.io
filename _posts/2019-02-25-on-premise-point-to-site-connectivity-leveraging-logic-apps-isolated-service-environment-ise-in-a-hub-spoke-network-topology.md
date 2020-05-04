---
ID: 5477
post_title: >
  On-Premise point-to-site connectivity
  leveraging Logic Apps Integration
  Services Environment (ISE) in a
  Hub-Spoke network topology
post_name: >
  on-premise-point-to-site-connectivity-leveraging-logic-apps-isolated-service-environment-ise-in-a-hub-spoke-network-topology
author: Rene Brauwers
post_date: 2019-02-25 23:44:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2019/02/25/on-premise-point-to-site-connectivity-leveraging-logic-apps-isolated-service-environment-ise-in-a-hub-spoke-network-topology/
published: true
tags: [ ]
categories:
  - Azure
  - Integration Service Environment
  - Logic Apps
  - Networking
  - Security
---
<!-- wp:paragraph {"align":"center"} -->
<p style="text-align:center"><em><strong><a href="https://azure.microsoft.com/en-us/updates/integration-services-environment/">Today Microsoft made the Logic Apps Integration Services Environment (ISE) available to the broader public by announcing its Public Preview!</a></strong></em></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Whilst ISE was in private preview I was able to
battle test it and my initial findings proofed it to be resilient, reliable and
above all performant. This of course is to be expected as the ISE environment
is a completely isolated environment which uses dedicated storage and other
resources and as such the days are gone of having to share an environment with
a noisy neighbour 😊</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Specifically
the ISE environment is a real good fit for all those scenario’s in which one
requires access to secured resources, systems and/or services in an Azure
Virtual Network.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>But
before I start repeating all the pros/cons and other goodness regarding the
why, I can only recommend that you give the following well written documents a
go.</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><a href="https://docs.microsoft.com/en-us/azure/logic-apps/connect-virtual-network-vnet-isolated-environment-overview">Access to Azure Virtual
     Network resources from Azure Logic Apps by using integration service
     environments (ISEs)</a></li><li><a href="https://docs.microsoft.com/en-us/azure/logic-apps/connect-virtual-network-vnet-isolated-environment">Connect to Azure virtual
     networks from Azure Logic Apps by using an integration service environment
     (ISE)</a></li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p><em>Note; if you are merely interested in the
configuration details which allows to connect your ISE to your on-premise
hosted systems using a Hub-Spoke network topology, scroll down…</em></p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>Background</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Whilst
I was battle testing the ISE environment there was one specific scenario I was
curious about if it was supported. This scenario consists of leveraging
multiple subscriptions within the same tenant whilst having one dedicated
'core' subscription containing the core virtual network components such as the
virtual network gateway, this kind of networking architecture is typically
referred to as a  <a href="https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke">Hub
and Spoke VNET architecture</a>. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So
what makes this scenario so unique to be tested out, you might ask? Well in
order to be able to establish a secure connection to your on-premise network we
will have to rely on a <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-about-vpngateways">VPN
Gateway (Azure virtual network gateway)</a> and within our Hub and Spoke
network topology this would mean that our ISE environment would live in a
different Azure Subscription, leveraging its own Virtual Network, which as such
requires additional configuration (<a href="https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-peering-overview">VNET
Peering</a>).  Nothing really fancy you
might reckon, well not really, as previously we ran into issues where we were
not able to connect to an on-premise hosted system leveraging a dedicated Azure
Functions App living outside of the core subscription and which was connected
to the core subscription containing the VNET which had the VPN Gateway. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Anyways
long story short; the above scenario works with ISE and as such I would easily
recommend using the Hub-Spoke networking topology in combination with ISE as
this architecture offers the following benefits (Blatantly copied from <a href="https://docs.microsoft.com/en-us/azure/architecture/reference-architectures/hybrid-networking/hub-spoke">Hub
and Spoke VNET architecture</a>):</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Cost savings by
     centralizing services that can be shared by multiple workloads, such as
     network virtual appliances (NVAs) and DNS servers, in a single location.</li><li>Overcome subscriptions limits
     by peering VNets from different subscriptions to the central hub.</li><li>Separation of concerns
     between central IT (SecOps, InfraOps) and workloads (DevOps).</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>And is a perfect fit
for scenario's such as:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Workloads deployed in
     different environments, such as development, testing, and production, that
     require shared services such as DNS, IDS, NTP, or AD DS. Shared services
     are placed in the hub VNet, while each environment is deployed to a spoke
     to maintain isolation.</li><li>Workloads that do not require
     connectivity to each other, but require access to shared services.</li><li>Enterprises that require
     central control over security aspects, such as a firewall in the hub as a
     DMZ, and segregated management for the workloads in each spoke.</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>So
enough blabla, let's dig into the nitty gritty; let's get to it.  </p>
<!-- /wp:paragraph -->

<!-- wp:heading -->
<h2>How-to: Allowing On-Premise point-to-site connectivity leveraging Logic Apps Integration Services Environment (ISE) in a Hub-Spoke network topology</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p><em>Note; I will be describing the high-level
configuration but will include links to articles hosted on the Microsoft Docs
site, containing the specifics around configuration and other required steps in
more details.</em></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p> The
image below depicts the high-level infrastructure configuration as I have
tested.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":5478} -->
<figure class="wp-block-image"><img src="https://brauwersnl.blob.core.windows.net/images/uploads/2019/02/hl-overview-ise.png" alt="" class="wp-image-5478"/></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>In
the above picture we see a simplified model of the Hub and spoke network
topology in which I tested my ISE environment. The solution relies on two Azure
subscriptions (both leveraging the same tenant). </p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Subscription A</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p> utilises the main HUB VNET (VNET Y) and
contains my shared services, including the VPN gateway which is responsible for
establishing secure tunnels to my work laptop which hosts a sample REST API and
my home server which hosts a FTP server. VNET Y contains 3 subnets configured
as mentioned below</p>
<!-- /wp:paragraph -->

<!-- wp:table -->
<table class="wp-block-table"><tbody><tr><td>
  Subnet
  </td><td>
  Address Range
  </td><td>
  Available
  Addresses
  </td></tr><tr><td>
  DMZ Subnet
  </td><td>
  10.1.0.0/29
  </td><td>
  3
  </td></tr><tr><td>
  Workload Subnet
  </td><td>
  10.1.1.0/24
  </td><td>
  250
  </td></tr><tr><td>
  VPN Gateway Subnet
  </td><td>
  10.1.0.16/28
  </td><td>
  10
  </td></tr></tbody></table>
<!-- /wp:table -->

<!-- wp:paragraph -->
<p>More information on
how to configure a VNET including the VPN Gateway for point-2-site connectivity
can be found by clicking on the following link:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li> <a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal">Configure a Point-to-Site
     connection to a VNet using native Azure certificate authentication: Azure
     portal</a></li></ul>
<!-- /wp:list -->

<!-- wp:heading {"level":3} -->
<h3>Subscription B</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>is
used to host my core integration services and contains a separate VNET (VNET X)
in which I've deployed the ISE. VNET X contains 4 subnets as required when
setting up a ISE and which are configured as mentioned below</p>
<!-- /wp:paragraph -->

<!-- wp:table -->
<table class="wp-block-table"><tbody><tr><td>
  Subnet
  </td><td>
  Address Range
  </td><td>
  Available
  Addresses
  </td></tr><tr><td>
  ISE Subnet 1
  </td><td>
  10.1.3.0/27
  </td><td>
  27
  </td></tr><tr><td>
  ISE Subnet 2
  </td><td>
  10.1.3.32/27
  </td><td>
  27
  </td></tr><tr><td>
  ISE Subnet 3
  </td><td>
  10.1.3.64/27
  </td><td>
  27
  </td></tr><tr><td>
  ISE Subnet 4
  </td><td>
  10.1.3.96/27
  </td><td>
  27
  </td></tr></tbody></table>
<!-- /wp:table -->

<!-- wp:paragraph -->
<p>More
information on how to set up an ISE including the subnet requirements can be
found by clicking on the following link:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><a href="https://docs.microsoft.com/en-us/azure/logic-apps/connect-virtual-network-vnet-isolated-environment#create-your-ise">Connect to Azure virtual
     networks from Azure Logic Apps by using an integration service environment
     (ISE)</a></li></ul>
<!-- /wp:list -->

<!-- wp:heading {"level":3} -->
<h3>VNET Peering</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>In
order for ISE to talk to my 'on-premise' servers it needs to connect to the VPN
gateway, hosted in Subscription A. This is achieved by means of leveraging VNET
peering and ensuring that both peering for VNET Y and VNET X are configured as
depicted in the table below.</p>
<!-- /wp:paragraph -->

<!-- wp:table -->
<table class="wp-block-table"><tbody><tr><td>
   Peering
  </td><td>
  Direction
  </td><td>
  Allow  forwarded traffic
  </td><td>
  Allow gateway
  traffic
  </td><td>
  Use remote
  gateways
  </td></tr><tr><td>
  VNET Y (Hub)
  </td><td>
  Core to ISE
  </td><td>
  False (Unchecked)
  </td><td>
  <strong>True (Checked)</strong>
  </td><td>
  False (Unchecked)
  </td></tr><tr><td>
  VNET X (Spoke)
  </td><td>
  ISE to Core
  </td><td>
  <strong>True (Checked)</strong>
  </td><td>
  False (Unchecked)
  </td><td>
  <strong>True (Checked)</strong>
  </td></tr></tbody></table>
<!-- /wp:table -->

<!-- wp:paragraph -->
<p>More
information on how to configure VNET peering and allow traffic to use the VPN
gateway can be found by clicking on the following link:</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li><a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit">Configure VPN gateway transit
     for virtual network peering</a></li></ul>
<!-- /wp:list -->

<!-- wp:heading -->
<h2> Testing on-prem connectivity</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>At
this point you should have all the information needed to successfully create a
Hub-spoke VNET topology, create a VPN point-2-site VPN, enable VNET peering and
provisioned an ISE; and more importantly you should have done so in the
following order:</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li><a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal#createvnet">Provisioned and configured a VNET in your 'core'
     subscription, containing multiple subnets</a></li><li><a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal">Provisioned and configured a
     point-2-site VPN using the Azure VPN Gateway Service</a></li><li><a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal#createvnet">Provisioned and configured a
     VNET in your 'integration' subscription, containing at least 4 subnets</a></li><li><a href="https://docs.microsoft.com/en-us/azure/logic-apps/connect-virtual-network-vnet-isolated-environment#create-your-ise">Provisioned and configured an
     Integration Service Environment in your 'integration' subscription</a></li><li><a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-peering-gateway-transit">Set up VNET peering on your
     VNET in your 'core' subscription and in your 'integration' subscription.</a></li><li><a href="https://docs.microsoft.com/en-us/azure/vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal#clientconfig">Established point-2-site VPN
     connections from your on-premise servers to your VNET in your 'core'
     subscription.</a></li></ol>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>In
order to validate that your ISE can successfully communicate with your
'on-premise' hosted servers, you would need to create some Logic Apps within
the ISE which would leverage ISE compatible connectors. At the point of writing
this blogpost the following connectors are ISE compatible and are able to
leverage all the goodness contained within your (linked) VNETs</p>
<!-- /wp:paragraph -->

<!-- wp:list -->
<ul><li>Azure Blob Storage, File
     Storage, and Table Storage</li><li>Azure Queues, Azure Service
     Bus, Azure Event Hubs, and IBM MQ</li><li>FTP and SFTP-SSH</li><li>SQL Server, SQL Data
     Warehouse, Azure Cosmos DB</li><li>AS2, X12, and EDIFACT</li></ul>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>Additionally
built-in triggers and actions such as HTTP always run in the same ISE as your
logic app and are as such capable to 
leverage all the goodness contained within your (linked) VNETs as well.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Test: Calling on-premise hosted REST-API</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The
screenshot depicted below details the basic ISE hosted logic app which I
created to test my on-premise connectivity, leveraging the HTTP connector.
Please note; the IP used ;-) </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":5480} -->
<figure class="wp-block-image"><img src="https://brauwersnl.blob.core.windows.net/images/uploads/2019/02/rest-api-designerview-1.png" alt="" class="wp-image-5480"/><figcaption>Designer view</figcaption></figure>
<!-- /wp:image -->

<!-- wp:image {"id":5491,"width":572} -->
<figure class="wp-block-image is-resized"><img src="https://brauwersnl.blob.core.windows.net/images/uploads/2019/02/rest-api-executedrun-view-1.png" alt="" class="wp-image-5491" width="572"/><figcaption>Execution run view</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Test:
Calling on-premise hosted FTP-Server</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The screenshot
depicted below details the basic ISE hosted logic app which I created to test
my on-premise connectivity, levering the FTP (ISE) Connector. Please note; the
IP used  ;-)  for the FTP API connection</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":5482,"width":572} -->
<figure class="wp-block-image is-resized"><img src="https://brauwersnl.blob.core.windows.net/images/uploads/2019/02/ftp-designerview.png" alt="" class="wp-image-5482" width="572"/></figure>
<!-- /wp:image -->

<!-- wp:image {"id":5485,"width":572} -->
<figure class="wp-block-image is-resized"><img src="https://brauwersnl.blob.core.windows.net/images/uploads/2019/02/image.png" alt="" class="wp-image-5485" width="572"/><figcaption>FTP Connection view</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":5484,"width":572} -->
<figure class="wp-block-image is-resized"><img src="https://brauwersnl.blob.core.windows.net/images/uploads/2019/02/ftp-executedrun-view.png" alt="" class="wp-image-5484" width="572"/><figcaption>Execution run view</figcaption></figure>
<!-- /wp:image -->

<!-- wp:heading -->
<h2>Conclusion</h2>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Although a lengthy
blog-post with quite some links to external Microsoft links; it proves pretty
straightforward to configure your Azure Environment such that you can have your
ISE environment connect to your 'on-premise' hosted systems / services. Taking
into account that ISE has full VNET support even complex networking topologies
such as the Hub and spoke architecture are supported.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Cheers</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>René</p>
<!-- /wp:paragraph -->