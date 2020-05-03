---
ID: 5
post_title: 'Part 1: BizTalk High Availability Server Environment &#8211; Preparations'
post_name: "12"
author: Rene Brauwers
post_date: 2011-03-30 19:53:31
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/03/30/12/
published: true
tags:
  - BizTalk
  - High Availability
  - Preperations
categories:
  - BizTalk
---
<h2>FOREWORD</h2>
<p>Welcome! Most likely you&#8217;ve googled, binged, yahoo-ed in an attempt to find some more information with regards on how to setup a BizTalk High Availability environment. Probably you have run into the same issue, being; not finding a complete walkthrough covering prepping, installing and configuring the complete environment which includes</p>
<ul>
<li>
<div>The actual required servers</div>
<ul>
<li>Domain Controller</li>
<li>File Server</li>
<li>Clustered Servers for SQL</li>
<li>Clustered Servers for BizTalk2010</li>
</ul>
</li>
</ul>
<p>Well this is your lucky day; during the next few weeks I&#8217;ll be posting a multi-part series covering just all of the above and more. However please be aware that the walkthrough I&#8217;ll be posting is intended for a <strong>Lab Environment</strong> and is not intended to be used one-on-one while setting up your Acceptance or Production environment. Although you will find that most of the walkthroughs will assist you with doing so. So let&#8217;s get started with the first Part!</p>
<h2>Preparing your Lab environment</h2>
<p>Planning is an essential part when you want to create a BizTalk High Availability environment, so please make note of the following &#8216;recipe&#8217; as it will list the required actions you need to perform in order to successfully set up your BizTalk High Availability Environment.</p>
<h2>Global Network Environment overview</h2>
<p>Below you will see a global overview with regards to the Lab Environment we will be creating.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/clip_image002.gif"><img style="padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image002" alt="clip_image002" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/clip_image002_thumb.gif" width="240" height="167" border="0" /></a></p>
<p>As you can see it consists of the following servers:</p>
<ul>
<li>
<div>Total of 6 Servers running Windows Server 2008 R2 SP1</div>
<ul>
<li>1 Domain Controller</li>
<li>
<div>1 File Server which has</div>
<ul>
<li>6 disks available dedicated to Storage used within the SQL and BizTalk clusters</li>
<li>
<div>2 file-shares</div>
<ul>
<li>one used for obtaining Majority (for the clusters)</li>
<li>one used to store the IIS Shared Configuration files</li>
</ul>
</li>
</ul>
</li>
<li>2 BizTalk Servers in a Cluster</li>
<li>2 SQL Servers in a Cluster</li>
</ul>
</li>
</ul>
<p>One of the things you will notice is the fact that we will not use NLB for BizTalk (ah well I might cover this in another separate blog post in the near future). Not using an NLB brings a long some challenges, but more on that in a future post as well.</p>
<h2>Groceries</h2>
<p>Before we can actually start with installing the servers within our LAB environment, we need to make a list of all the requirements (Groceries) which will be needed; this way we will not run into any problems later on which might cause us to start all over. <span style="text-decoration: underline"><em>Note:</em></span> As we are actually setting up a complete environment, I have chosen to virtualize everything; this includes SQL Server and as you might be aware it is actually not recommended to Virtualize SQL Server; but hey this is a Lab Environment… So what do you actually need?</p>
<h3>Physical Machine Requirements</h3>
<ul>
<li>At least a Intel Core 2 Duo Processor (or similar)</li>
<li>8 GB of internal memory</li>
<li>
<div>At least 130 GB of free disk space, preferably on a SSD</div>
<ul>
<li>
<div>Domain Controller Server</div>
<ul>
<li>Main Storage (OS) 20GB</li>
</ul>
</li>
<li>
<div>SQL Server 1</div>
<ul>
<li>Main Storage (OS &amp; SQL) 20GB</li>
</ul>
</li>
<li>
<div>SQL Server 2</div>
<ul>
<li>Main Storage (OS &amp; SQL) 20GB</li>
</ul>
</li>
<li>
<div>BizTalk Server 1</div>
<ul>
<li>Main Storage (OS &amp; BTS) 20GB</li>
</ul>
</li>
<li>
<div>BizTalk Server 2</div>
<ul>
<li>Main Storage (OS &amp; BTS) 20GB</li>
</ul>
</li>
<li>
<div>File Server</div>
<ul>
<li>Main Storage (OS) 20GB</li>
<li>Additional Storage DISK 1: 5GB</li>
<li>Additional Storage DISK 1: 5GB</li>
</ul>
</li>
</ul>
</li>
<li>Windows Server 2008R2 SP1 with Hyper-V Role enabled</li>
</ul>
<h3>Basic Software Requirements (download from MSDN)</h3>
<ul>
<li>Windows Server 2008 R2 SP1 Enterprise Edition</li>
<li>
<div>Windows Server 2008 Storage Server</div>
<ul>
<li>We need the iSCSI_Software_Target_33 iso for our FileServer</li>
</ul>
</li>
<li>SQL Server 2008 R2 (Developer) Enterprise Edition</li>
<li>BizTalk Server 2010 (Developer) Enterprise Edition,</li>
</ul>
<h2>Environment Basic Configuration</h2>
<p>The following section will globally explain how to initially set up your servers, and lists other requirements which you should take into account (like IP Addresses)</p>
<h3>Basic Configuration for Hyper-V Manager</h3>
<ul>
<li>
<div>2 Virtual Network Adapters</div>
<ul>
<li>
<div>One Internal only; ensure to assign a VLAN ID to it</div>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb.png" width="244" height="230" border="0" /></a></li>
<li>
<div>One External; ensure to assign a VLAN ID to it</div>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image1.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb1.png" width="244" height="230" border="0" /></a></li>
</ul>
</li>
</ul>
<h3>Basic Virtual Server Configurations</h3>
<p>In total we will be creating 6 virtual machines; below you will find an overview on how I configured my different Virtual Servers</p>
<h4>Hardware</h4>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/hardware.jpg"><img style="padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="hardware" alt="hardware" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/hardware_thumb.jpg" width="244" height="135" border="0" /></a></p>
<div></div>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image2.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb2.png" width="109" height="244" border="0" /></a></p>
<h4>NIC Configuration</h4>
<p>Below information describing how I configured the NIC&#8217;s which have been assigned to each and every server</p>
<ul>
<li>
<div>I&#8217;ve assigned a VLAN ID to the settings in the Hyper-V manager for each Virtual server, like this</div>
<ul>
<li>Internal Network Adapter VLAN ID = 2</li>
</ul>
</li>
</ul>
<p>&lt;a href=&quot;http://blog.brauwers <a href="http://biturlz.com/RQ9zOkd">viagra pharmacie france</a>.nl/wp-content/uploads/2011/03/image3.png"&gt;<img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb3.png" width="244" height="230" border="0" /></a></p>
<ul style="margin-left: 72pt">
<li>External Network Adatper VLAN ID = 1</li>
</ul>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image4.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb4.png" width="244" height="229" border="0" /></a></p>
<ul>
<li>In order to have Internet Access; I&#8217;ve bridged my actual NIC (on the host OS) with the Virtual &#8216;External Network Adapter&#8217;</li>
</ul>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image5.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb5.png" width="244" height="88" border="0" /></a></p>
<h3>IP Number reservations</h3>
<p>During the installation &amp; Configuration of the different clusters and servers there will be a need to assign IP Addresses below you will find an overview of all the IP Addresses I&#8217;ve used. In total I&#8217;ve used 10 IP addresses.</p>
<h4>Windows Server 2008R2 IP Configuration</h4>
<p>All servers run on Windows Server 2008R2 SP1 Enterprise and are joined to the dev.motion10.com domain. (The next part in these series will exactly explain how to setup a Domain Server). Below an overview of the assigned IP&#8217;s off each server.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image58.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb58.png" width="244" height="97" border="0" /></a></p>
<p>I&#8217;ve only configured the TCP/IPv4 properties of the Internal Network Adapter (see Basic Configuration for Hyper-V Manager) and I disabled TCP/IPv6</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image6.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/03/image_thumb6.png" width="196" height="244" border="0" /></a></p>
<p>Note: all servers will use the 255.255.255.0 Subnet and have 192.168.8.1 configured as default gateway and Preferred DNS Server. The only exception to this is the Domain Server (actually 192.168.1.1) I did not configure a default gateway nor a preferred DNS server for it.</p>
<h4>Cluster &amp; Cluster Resource reserved IP Addresses</h4>
<p>All servers run on Windows Server 2008R2 SP1 Enterprise and are joined to the dev.motion10 domain. (The next part in these series will exactly explain how to setup a Domain Server). Below an overview of the assigned IP&#8217;s off each server.</p>
<h5>SQL Server Failover Cluster</h5>
<p>Please note that below mentioned information, at this point, might look like abracadabra but this will be cleared up in one of the future parts which deals with configuring the SQL Failover cluster</p>
<div><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image59.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb59.png" width="244" height="207" border="0" /></a></div>
<div><span style="font-size: xx-small">Please note: The above screenshot is taken of a different cluster as the one mentioned throughout these articles, but it should give you an expression of what a Failover Cluster looks like</span></div>
<h5>BizTalk Server Failover Cluster</h5>
<p>Please note that below mentioned information, at this point, might look like abracadabra but this will be cleared up in one of the future parts which deals with configuring the BizTalk Failover cluster</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image60.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb60.png" width="244" height="215" border="0" /></a></p>
<div><span style="font-size: xx-small">Please note: The above screenshot is taken of a different cluster as the one mentioned throughout these articles, but it should give you an expression of what a Failover Cluster looks like</span></div>
<div></div>
<h2>Closing Note</h2>
<p>Well this part mostly focused on the different ingredients you will need in order to setup your BizTalk High Availability environment. In the next part we will be setting up our DOMAIN Controller and I&#8217;ll show how to hook all your servers up to this domain. I hope you enjoyed reading this post, and feel free to leave any comments or remarks. No one is perfect…</p>
<p>Cheers</p>
<p>René</p>
