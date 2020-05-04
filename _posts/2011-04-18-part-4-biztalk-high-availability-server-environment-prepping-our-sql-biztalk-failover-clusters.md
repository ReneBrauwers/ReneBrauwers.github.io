---
ID: 8
post_title: 'Part 4: BizTalk High Availability Server Environment &#8211; Prepping our SQL &#038; BizTalk Failover Clusters'
post_name: >
  part-4-biztalk-high-availability-server-environment-prepping-our-sql-biztalk-failover-clusters
author: Rene Brauwers
post_date: 2011-04-18 20:32:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/04/18/part-4-biztalk-high-availability-server-environment-prepping-our-sql-biztalk-failover-clusters/
published: true
tags:
  - BizTalk
  - Failover Clustering
  - Features
  - File Server
  - Installation how to
  - iSCSI
  - MSMQ
  - Roles
  - SQL Server
  - Windows Storage Server
categories:
  - BizTalk
---
In our previous posts we’ve set up our Domain controller. This post will focus on prepping our other Servers which will be used and include:
<ul>
	<li>BizTalk Failover Servers</li>
	<li>SQL Server Failover Servers</li>
	<li>File Server</li>
</ul>
This posts will assume that you’ve already pre-installed 5 servers with Windows Server 2008R2, named your servers, assigned Fixed IP’s and hooked them up to your Domain.

Please be aware; this is a long blog-post; read it carefully and I recommend to follow the steps in the order as mentioned. <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/wlEmoticon-smile.png" />

&nbsp;
<h2>Prepping your File Server</h2>
Your File Server will fulfill a crucial part when setting up both your SQL and BizTalk Failover clusters as both of them require available storage which is to be used as:
<ul>
	<li>a witness (used to obtain majority for your clusters)</li>
	<li>a clustered SQL Resource</li>
	<li>a clustered MCDTC Resource</li>
	<li>a clustered MSMQ Resource</li>
</ul>
&nbsp;
<h3></h3>
<h3>Adding the required features</h3>
Once you’ve booted up your File Server and logged on to your domain, open up your Server Manager and ‘Right Click’ on Features and select ‘Add Features’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image61.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb61.png" width="199" height="244" border="0" /></a>

A wizard will start and once you’re on the ‘Select Features’ screen, select the ‘Storage Manager for SAN’s feature and press next

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image62.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb62.png" width="244" height="181" border="0" /></a>

On the ‘Confirmation Screen’ select ‘Install’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image63.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb63.png" width="244" height="181" border="0" /></a>

Verify the installation results and click ‘Close’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image64.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb64.png" width="244" height="181" border="0" /></a>
<h2>Prepping your SQL Servers</h2>
Boot up one of your Servers (which will be used for SQL) and login with the domain admin account, open up your Server Manager and ‘Right Click’ on Features and select ‘Add Features’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image65.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb65.png" width="220" height="244" border="0" /></a>

A wizard will start and once you’re on the ‘Select Features’ screen, select the ‘Failover Clustering’ feature and press ‘next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image66.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb66.png" width="244" height="181" border="0" /></a>

On the ‘Confirmation Screen’ select ‘Install’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image67.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb67.png" width="244" height="180" border="0" /></a>

Verify the installation results and click ‘Close’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image68.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb68.png" width="244" height="181" border="0" /></a>

Repeat the above mentioned steps for your second server which will be used for sql.
<h2>Prepping your BizTalk Servers</h2>
Boot up one of your Servers (which will be used for BizTalk) and login with the domain admin account.
<h3>Adding the required Roles and Role Services</h3>
open up your Server Manager and ‘Right Click’ on Roles and select ‘Add Roles’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image69.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb69.png" width="201" height="219" border="0" /></a>

A wizard will start and once you’re on the ‘Select Server Roles’ screen, select the following Role ‘Application Server’. A message will appear informing you that some additional features are required. Select ‘Add Required Features’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image70.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb70.png" width="244" height="183" border="0" /></a>

Press ‘Next’ until you reach the ‘Select Role Services’ screen in order to add the ‘Application Server’ Role and the required Features

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image71.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb71.png" width="244" height="180" border="0" /></a>

On the ‘Select Role Services’ Screen, select the ‘Web Server (IIS) Support’. A message will appear informing you that some additional features and/or role services are required. Select ‘Add Required Role Service’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image72.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb72.png" width="244" height="179" border="0" /></a>

Once done, ensure to select the the following Role Services as well:
<ul>
	<li>COM+ Network Access</li>
	<li>TCP Port Sharing</li>
	<li>HTTP Activation</li>
	<li>Message Queuing Activation</li>
</ul>
&nbsp;

While selecting the Role Service ‘Message Queuing Activation’ a message will appear informing you that some additional features are required. Select ‘Add Required Features’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image73.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb73.png" width="244" height="181" border="0" /></a>

Continue with selecting the following Role Services:
<ul>
	<li>TCP Activation</li>
	<li>Named Pipes Activation</li>
	<li>Incoming Remote Transactions</li>
	<li>Outgoing Remote Transactions</li>
	<li>WS-Atomic Transactions</li>
</ul>
&nbsp;

You should now have all Role Services selected, press ‘next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image74.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb74.png" width="244" height="180" border="0" /></a>

You will asked to Choose a Server Authentication Certificate for SSL Encryption. Select the option “Choose a certificate for SSL encryption later’ and press ‘next

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image75.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb75.png" width="244" height="180" border="0" /></a>

Now proceed until you reach the ‘Role Services’ Screen and check that all Role Services are checked with <strong><span style="color: #ff0000;">exception off the following</span>:</strong>
<ul>
	<li>WebDav Publishing</li>
	<li>ASP</li>
	<li>CGI</li>
	<li>Server Side Includes</li>
	<li>Custom Logging</li>
	<li>ODBC Logging</li>
	<li>IIS 6 Scripting Tools</li>
	<li>IIS 6 Management Console</li>
	<li>FTP Service</li>
	<li>FTP Extensibility</li>
	<li>IIS Hostable Web Core</li>
</ul>
&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image76.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb76.png" width="244" height="181" border="0" /></a>

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image77.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb77.png" width="244" height="181" border="0" /></a>

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image78.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb78.png" width="244" height="181" border="0" /></a>

Press next, and conform the Installation Selections and then press Install<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image79.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb79.png" width="244" height="181" border="0" /></a>

Check the results and click on Close

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image80.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb80.png" width="244" height="180" border="0" /></a>
<h3>Adding the required Features (MSMQ)</h3>
<em><strong>Please note the following instructions (copied from : <a title="http://technet.microsoft.com/en-us/library/cc730960.aspx" href="http://technet.microsoft.com/en-us/library/cc730960.aspx">http://technet.microsoft.com/en-us/library/cc730960.aspx</a>) </strong></em>
<blockquote>
<h4><span style="font-size: xx-small;">Setting Permissions in Active Directory Domain Services Before Installing the Routing Service or the Directory Service Integration Features of Message Queuing</span></h4>
<span style="font-size: xx-small;">The successful installation of the Routing Service feature on a Windows Server 2008 R2 computer that is not a domain controller, or the Directory Service Integration feature of Message Queuing on a Windows Server 2008 R2 computer that is a domain controller requires that specific permissions are set in Active Directory Domain Services. Follow these steps to grant the appropriate permissions in Active Directory Domain Services before installing these features.</span>

<strong><span style="font-size: xx-small;">To grant permissions for a computer object to the Servers object in Active Directory Domain Services before installing the Routing Service feature on a computer that is not a domain controller </span></strong>
<ol>
	<li><span style="font-size: xx-small;">Click <strong>Start</strong>, point to <strong>Programs</strong>, point to <strong>Administrative Tools</strong>, and then click <strong>Active Directory Sites and Services</strong> to open <strong>Active Directory Sites and Services</strong>.</span></li>
	<li><span style="font-size: xx-small;">Click to expand <strong>Active Directory Sites and Services</strong>, click to expand <strong>Sites</strong>, and then click to expand the site which this computer will be a member of.</span></li>
	<li><span style="font-size: xx-small;">Right-click <strong>Servers</strong> and select <strong>Properties</strong> to display the <strong>Servers Properties</strong> dialog box.</span></li>
	<li><span style="font-size: xx-small;">Click the <strong>Security</strong> tab of the <strong>Servers Properties</strong> dialog box.</span></li>
	<li><span style="font-size: xx-small;">Click the <strong>Add</strong> button to display the <strong>Select Users, Computer, or Groups</strong> dialog box.</span></li>
	<li><span style="font-size: xx-small;">Click the <strong>Object Types</strong> button to display the <strong>Object Types</strong> dialog box, click to enable <strong>Computers</strong>, and then click <strong>OK</strong>.</span></li>
	<li><span style="font-size: xx-small;">Enter the name of the computer for which the Routing Service or Directory Service Integration feature will be installed, click <strong>Check Names</strong>, and then click <strong>OK</strong>.</span></li>
	<li><span style="font-size: xx-small;">Enable the following permissions for this computer object:</span>
<ul>
	<li><span style="font-size: xx-small;">Allow Read</span></li>
	<li><span style="font-size: xx-small;">Allow Write</span></li>
	<li><span style="font-size: xx-small;">Allow Create all child objects</span></li>
</ul>
</li>
	<li><span style="font-size: xx-small;">After enabling these permissions, click <strong>Advanced</strong> to display the <strong>Advanced Security Settings for Servers</strong> dialog box.</span></li>
	<li><span style="font-size: xx-small;">Select the computer object from the list of permission entries, and then click the <strong>Edit</strong> button.</span></li>
	<li><span style="font-size: xx-small;">Select <strong>This</strong><strong>object and all descendant objects</strong> from the <strong>Apply to</strong> drop-down list, and then click <strong>OK</strong>.</span></li>
	<li><span style="font-size: xx-small;">Click <strong>OK</strong> to close the <strong>Advanced Security Settings for Servers</strong> dialog box.</span></li>
	<li><span style="font-size: xx-small;">Click <strong>OK</strong> to close the <strong>Server Properties</strong> dialog box.</span></li>
</ol>
</blockquote>
open up your Server Manager and ‘Right Click’ on Features and select ‘Add Features’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image81.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb81.png" width="191" height="244" border="0" /></a>

A wizard will start and once you’re on the ‘Select Features screen, expand the ‘Message Queuing’ Feature and ensure to select the all options with <strong><span style="color: #ff0000;">exception of</span></strong>:
<ul>
	<li>Routing Service</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image82.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb82.png" width="244" height="181" border="0" /></a>

Select ‘Next’ and confirm the Installation Selections. Once done press ‘Install’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image83.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb83.png" width="244" height="181" border="0" /></a>

Verify the Installation Results and then press ‘Close’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image84.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb84.png" width="244" height="182" border="0" /></a>
<h3>Adding the required Features (Failover Clustering)</h3>
open up your Server Manager and ‘Right Click’ on Features and select ‘Add Features’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image85.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb85.png" width="191" height="244" border="0" /></a>

A wizard will start and once you’re on the ‘Select Features’ screen, select the ‘Failover Clustering’ feature and select ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image86.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb86.png" width="244" height="180" border="0" /></a>

Confirm the option and select ‘Install’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image87.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb87.png" width="244" height="180" border="0" /></a>

Verify the Installation Results and then press ‘Close’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image88.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb88.png" width="244" height="180" border="0" /></a>

Repeat the above mentioned steps for your second server which will be used for BizTalk.

&nbsp;
<h2>Adding Storage to your SQL and BizTalk Servers</h2>
At this point we will have prepped our File Server and all of of our BizTalk and SQL Servers, but we are not quite there yet.

As we are running our servers in a Virtual Environment and we don’t have dedicated storage servers we will need to ‘emulate’ this. In order to do this I’ve decided to use the iSCSI Target software which is part of the Windows Server 2008 Storage Server.

Please note that this is not part of the Windows Server 2008 .iso, you will actually need to download it from MSDN (<a title="http://msdn.microsoft.com/en-us/subscriptions/downloads/default.aspx" href="http://msdn.microsoft.com/en-us/subscriptions/downloads/default.aspx">http://msdn.microsoft.com/en-us/subscriptions/downloads/default.aspx</a>)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image89.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb89.png" width="244" height="109" border="0" /></a>

Once you’ve downloaded the required iso, mount it using your favorite tool. Once you’ve mounted it you will see a self-extracting file named WSS2008R2+ISCSITarget33.exe

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image90.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb90.png" width="244" height="68" border="0" /></a>

Click on it and within the destination folder you will find 2 iso files, one of them called iSCSI_Software_Target_33.iso .
<h3>Adding the required iSCSI software to your Servers</h3>
<h4></h4>
Hook this iSCSI_Software_Target_33 iso file up to your Virtual Machines. There are several ways to do this. Below a description on how I did this for my File Server.

Open up your Hyper-V manager, right click on your File Server and select ‘Settings’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image91.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb91.png" width="244" height="241" border="0" /></a>

Go to your DVD-Drive and within the media Group box select ‘image file’ and browse to the above mentioned iSCSI_Software_Target_33.iso

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image92.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb92.png" width="244" height="230" border="0" /></a>

At this point you’ve mounted the iSCSI_Software_Target_33.iso to your server and now we can go back to our server and access the contents.

From your File Server, browse to your DVD-Drive and open up the Index.htm file

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image93.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb93.png" width="244" height="204" border="0" /></a>
<h4>Install the iSCSI Software Target (x64)</h4>
This step only needs to be done on your File Server, and thus <span style="color: #ff0000;"><strong>can be skipped for the 2 SQL Server Machines and 2 BizTalk Server Machines</strong></span>

&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image94.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb94.png" width="244" height="218" border="0" /></a>

On the welcome screen; press ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image95.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb95.png" width="244" height="189" border="0" /></a>

Accept the End-User License Agreement

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image96.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb96.png" width="244" height="189" border="0" /></a>

Choose a destination folder, and select ‘next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image97.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb97.png" width="244" height="188" border="0" /></a>

Choose if you would like to join the Customer Experience Improvement Program

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image98.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb98.png" width="244" height="189" border="0" /></a>

Choose if you want to use Microsoft Update

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image99.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb99.png" width="244" height="189" border="0" /></a>

Install the iSCSI Software target

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image100.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb100.png" width="244" height="189" border="0" /></a>

Finish the installation

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image101.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb101.png" width="244" height="189" border="0" /></a>
<h4>Install the iSCSI Hardware Providers</h4>
Below mentioned steps, need to be executed on <span style="color: #ff0000;"><strong><span style="text-decoration: underline;"><em>all servers</em></span></strong></span> with exception of your Domain Server.

Open the index.htm file once again and now select the VSS,VDS and HPC Hardware Providers (x64) link

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image102.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb102.png" width="244" height="218" border="0" /></a>

On the welcome screen; press ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image103.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb103.png" width="244" height="190" border="0" /></a>

Accept the End-User License Agreement

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image104.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb104.png" width="244" height="189" border="0" /></a>

Choose ‘Typical installation’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image105.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb105.png" width="244" height="189" border="0" /></a>

Provide a domain user account and enter the according password

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image106.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb106.png" width="244" height="190" border="0" /></a>

Choose if you want to use Microsoft Update

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image107.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb107.png" width="244" height="189" border="0" /></a>

Install the Software Client Software

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image108.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb108.png" width="244" height="189" border="0" /></a>

Finish the installation

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image109.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb109.png" width="244" height="189" border="0" /></a>
<h4>Install the iSCSI Clients for the BizTalk and SQL Servers</h4>
Now log on to one of your SQL Machines, and go to start and type ‘iscsi initiator’ in the search box and hit enter.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image110.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb110.png" width="220" height="244" border="0" /></a>

You will be presented with a message stating that the iSCSI service is not running and you will be presented with the option to automatically start this service. Select ‘Yes’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image111.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb111.png" width="244" height="106" border="0" /></a>

At this point you will be presented with the iSCSI Initiator Properties screen.<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image112.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb112.png" width="172" height="244" border="0" /></a>

Now go to the Configuration Tab, and make a note of the ‘Initiator Name’ (Write it down or even better Copy and Paste it into notepad, as we will use it later on)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image113.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb113.png" width="173" height="244" border="0" /></a>

Repeat the above mentioned steps for the other SQL Machine and both BizTalk Machines, eventually you should have made written down 4 Initiator names which should be something similar to the entries mentioned below
<ul>
	<li>iqn.1991-05.com.microsoft:sql001.lab.motion10.com</li>
	<li>iqn.1991-05.com.microsoft:sql002.lab.motion10.com</li>
	<li>iqn.1991-05.com.microsoft:bts001.lab.motion10.com</li>
	<li>iqn.1991-05.com.microsoft:bts002.lab.motion10.com</li>
</ul>
<h3>Adding Storage to be used</h3>
At this point we will have the required software installed on all of our servers and now it’s time to define some storage which will be made available to our BizTalk and SQL Server Machines.

In order to do this we will have to go back to our File Server; so go ahead and do this.
<h4>Creating your Virtual Disks</h4>
Once logged on. Go to the Server Manager, select Storage , expand Microsoft iSCSI Software Target and select devices and under actions select Create Virtual Disk

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image114.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb114.png" width="244" height="173" border="0" /></a>

The Create Virtual Disk Wizard will now appear. Select ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image115.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb115.png" width="244" height="200" border="0" /></a>

Now enter the path and Filename of the Virtual Disk you want to <strong>create</strong>. We will start of with creating a disk which will be used as our main database storage disk

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image116.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb116.png" width="244" height="201" border="0" /></a>

Add the desired size

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image117.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb117.png" width="244" height="199" border="0" /></a>

Add a description

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image118.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb118.png" width="244" height="200" border="0" /></a>

Skip the “Access” part for now and select ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image119.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb119.png" width="244" height="199" border="0" /></a>

Now complete the Wizard and once done select the ‘Finish’ button

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image120.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb120.png" width="244" height="199" border="0" /></a>

Repeat the above mentioned steps for the following virtual disks

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image121.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb121.png" width="244" height="136" border="0" /></a>

Eventually you should have a total of 8 virtual disks<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image122.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb122.png" width="244" height="79" border="0" /></a>
<h3>Creating your iSCSI Targets</h3>
Now that we have created all of our virtual disks, it is time to create a dedicated BizTalk and SQL iSCSI Target and assign the designated disks to them.

In order to do this go to the Server Manager, select Storage , expand Microsoft iSCSI Software Target, select iSCSI Targets and right click and select ‘Create iSCSI Target’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image123.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb123.png" width="205" height="244" border="0" /></a>

The ‘Create iSCSI Target Wizard’ will pop up, select ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image124.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb124.png" width="244" height="189" border="0" /></a>

Now we will have to enter the iSCSI Target Identification information. We will start with creating an Target for the BizTalk Failover Cluster. For the iSCSI Target Name enter ‘BIZTALK’ and for the description we’ll enter ‘BIZTALK TARGET’. Once done press ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image125.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb125.png" width="244" height="189" border="0" /></a>

At this point in time you will be presented with the iSCSI Initiators Identifiers screen; in this screen we will be entering our previously written down initiator names; in my case the were:
<ul>
	<li>iqn.1991-05.com.microsoft:sql001.lab.motion10.com</li>
	<li>iqn.1991-05.com.microsoft:sql002.lab.motion10.com</li>
	<li>iqn.1991-05.com.microsoft:bts001.lab.motion10.com</li>
	<li>iqn.1991-05.com.microsoft:bts002.lab.motion10.com</li>
</ul>
&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image126.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb126.png" width="244" height="189" border="0" /></a>

Click on Advanced and then select ‘Add’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image127.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb127.png" width="244" height="217" border="0" /></a>

For Machine bts001 use the below mentioned information. Once done press ‘OK’
<ul>
	<li>iqn.1991-05.com.microsoft:bts001.lab.motion10.com</li>
</ul>
&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image128.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb128.png" width="244" height="222" border="0" /></a>

For Machine bts002 use the below mentioned information. Once done press ‘OK’ . Please note you will get a warning, but this warning can be ignored as we want to allow multiple initiators as we are setting up a cluster
<ul>
	<li>iqn.1991-05.com.microsoft:bts002.lab.motion10.com</li>
</ul>
&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image129.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb129.png" width="244" height="215" border="0" /></a>

Eventually you should end up with the following 2 entries:

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image130.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb130.png" width="244" height="214" border="0" /></a>

Select  ‘OK’ and then select ‘NEXT’ and then ‘FINISH’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image131.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb131.png" width="244" height="186" border="0" /></a>

Repeat the above mentioned steps however this time you will be creating a Target for your SQL Machines. Below the information as I have used
<blockquote>TARGET NAME: SQL2008
TARGET DESCRIPTION: SQL SERVER 2008 TARGET
IQN IDENTIFIERS USED:
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image132.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb132.png" width="244" height="64" border="0" /></a></blockquote>
<h3>Assigning the Virtual Disks to the correct iSCSI Targets</h3>
At this point we have defined our two targets, namely BIZTALK and SQL2008 and we’ve created our Virtual Storage Disks. The next step is to actually assign the Virtual Storage to the correct iSCSI target.
<h4>Adding the BizTalk Storage Disks to the BizTalk iSCSI Target</h4>
Open up Server Manager on your File Server. Select Storage , expand Microsoft iSCSI Software Target, select iSCSI Targets, Select the BizTalk target and right click it and select ‘Add Existing Virtual Disk to iSCSI Target’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image133.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb133.png" width="244" height="191" border="0" /></a>

Select all Virtual Disks which indicate that they are to be used by BizTalk and select OK. ( In my case I’ve named the Virtual disks in such a way that I can easily recognize which disks are to be used within the BizTalk Cluster as I used the term BTS)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image134.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb134.png" width="203" height="244" border="0" /></a>

&nbsp;
<h4>Adding the SQL Storage Disks to the SQL2008 iSCSI Target</h4>
Open up Server Manager on your File Server. Select Storage , expand Microsoft iSCSI Software Target, select iSCSI Targets, Select the SQL2008 target and right click it and select ‘Add Existing Virtual Disk to iSCSI Target’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image135.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb135.png" width="185" height="244" border="0" /></a>

Select all Virtual Disks which indicate that they are to be used by SQL Server and select OK. ( In my case I’ve named the Virtual disks in such a way that I can easily recognize which disks are to be used within the SQL Cluster as I used the term SQL)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image136.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb136.png" width="206" height="244" border="0" /></a>
<h3>Hooking up your SQL and BizTalk machines to the intended iSCSI target</h3>
Now that we’ve created our designated Targets and assigned the designated Virtual disks to the target we need to configure the SQL and BizTalk machines such that they have access to these virtual disks.

In order to do this, log on to one of your SQL Machines open up the iSCSI Client. This is done by clicking on ‘Start’ and typing ‘iscsi initiator’ in the search box. Now hit ‘enter’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image137.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb137.png" width="220" height="244" border="0" /></a>

You are now presented with the iSCSI Initiator properties screen. Go to the ‘Discovery’ tab and select the “Discover Portal… button”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image138.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb138.png" width="173" height="244" border="0" /></a>

Now enter the IP Address or the DNS name of your File Server; in my case this would be the EUROPOORT, and select ‘OK’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image139.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb139.png" width="244" height="146" border="0" /></a>

You should now see your portal in the Target Portals Group Box.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image140.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb140.png" width="173" height="244" border="0" /></a>

Now select the “Targets” Tab, and you should see a Discovered Target IQN which is inactive.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image141.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb141.png" width="174" height="244" border="0" /></a>

Select the discovered Target and click on the Connect button

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image142.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb142.png" width="173" height="244" border="0" /></a>

A popup box will appear, just click on “OK”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image143.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb143.png" width="244" height="123" border="0" /></a>

At this point your first SQL Machine is connected to its dedicated SQL2008 Target

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image144.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb144.png" width="173" height="244" border="0" /></a>

Press OK, and repeat the above mentioned steps for the other SQL Machine and the other BizTalk machines.
<h2>Closing Note</h2>
Well it has been a long read (and not to forget quite a long write <img class="wlEmoticon wlEmoticon-winkingsmile" alt="Winking smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/wlEmoticon-winkingsmile.png" />) but this sums up part 4. So far we’ve accomplished the fundamental preparations required in order for us to proceed with the actual installment and configuration of the BizTalk and SQL Cluster.

So what to expect in the near future; well part 5 will cover setting up the SQL Cluster which includes
<ul>
	<li>Clustering the DTC</li>
	<li>Actually Installing and Clustering SQL Server</li>
</ul>
&nbsp;

part 6 will cover setting up the BizTalk cluster which includes
<ul>
	<li>Clustering DTC</li>
	<li>Clustering IIS</li>
	<li>Clustering MSMQ</li>
	<li>Installing and configuring BizTalk</li>
	<li>Clustering the SSO</li>
	<li>Clustering BizTalk</li>
</ul>
&nbsp;

I actually hope to finish up these series this month, but I can’t make a promise as I have a few exams to prepare including 70-595 (BTS 2010) and 70-432 (SQL Server 2008)

Well I hope you enjoyed the posts so far, check back soon and feel free to leave any comments, remarks and/or suggestions with regards to Blog posts you would like to see in the future.

Cheers

René