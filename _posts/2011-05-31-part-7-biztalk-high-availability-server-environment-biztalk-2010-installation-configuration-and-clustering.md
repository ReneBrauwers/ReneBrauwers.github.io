---
ID: 11
post_title: 'Part 7 &#8211; BizTalk High Availability Server Environment –BizTalk 2010 Installation, Configuration and Clustering'
post_name: >
  part-7-biztalk-high-availability-server-environment-biztalk-2010-installation-configuration-and-clustering
author: Rene Brauwers
post_date: 2011-05-31 00:02:06
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/05/31/part-7-biztalk-high-availability-server-environment-biztalk-2010-installation-configuration-and-clustering/
published: true
tags:
  - BizTalk
  - BizTalk Clustering
  - BizTalk Configuration
  - BizTalk Installation
  - dtcping
  - Installation how to
  - instructions
  - Joining existing BizTalk Group
  - Microsoft BizTalk Adapter Pack
  - >
    Microsoft SQL Server Data Transformation
    Services (DTS 2008)
  - Microsoft WCF Lob Adapter SDK
  - Single Sign On
categories:
  - BizTalk
---
So, finally we’ve reached the part in which we will actually install, configure and cluster BizTalk Server 2010. As mentioned in my previous posts; I assume you’ve followed all steps mentioned in the previous posts. Okay let’s get started!
<h2></h2>
<h2>Verify that MSDTC is configured</h2>
Before we can go ahead with the actual installation, we need to make sure that we’ve configured our MSDTC correctly. In order to do so we will need the following tool dtcping.exe, which can be downloaded <a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=5e325025-4dcd-4658-a549-1d549ac17644" target="_blank" rel="noopener noreferrer">here</a>. Once you’ve downloaded dtcping, copy it over to all servers involved, in my case that would be
<ul>
	<li>SQL001</li>
	<li>SQL002</li>
	<li>BTS001</li>
	<li>BTS002</li>
</ul>
<blockquote><em>Time Saver Tip: Locally install dtcping.exe (it’s actually an self-extracting archive) and copy only the dtcping.exe application to your servers</em></blockquote>
Next we will start pinging all servers, and we will do this using the following matrix
<table width="459" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td valign="top" width="100">Main Server</td>
<td valign="top" width="130">Partner DTC Server</td>
<td valign="top" width="227"></td>
</tr>
<tr>
<td valign="top" width="100">BTS001</td>
<td valign="top" width="130">SQL001</td>
<td valign="top" width="227">Ensure that BizTalk node one is Active and SQL node one is Active</td>
</tr>
<tr>
<td valign="top" width="100">BTS001</td>
<td valign="top" width="130">SQL002</td>
<td valign="top" width="227">Ensure that BizTalk node one is Active and SQL node two is Active (perform a failover)</td>
</tr>
<tr>
<td valign="top" width="100">BTS002</td>
<td valign="top" width="130">SQL002</td>
<td valign="top" width="227">Ensure that BizTalk node two (perform a failover) is Active and SQL node one is Active</td>
</tr>
<tr>
<td valign="top" width="100">BTS002</td>
<td valign="top" width="130">SQL001</td>
<td valign="top" width="227">Ensure that BizTalk node one is Active and SQL node two is Active (perform a failover)</td>
</tr>
</tbody>
</table>
<h3>BTS001 vs SQL001</h3>
Logon to your main BizTalk node (in my case BTS001) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image210.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb210.png" width="198" height="244" border="0" /></a>

Ensure that your main BizTalk node is currently the owner <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image211.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb211.png" width="244" height="159" border="0" /></a>

Now logon to your main SQL Server Node (in my case SQL001) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image212.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb212.png" width="198" height="244" border="0" /></a>

Ensure that your main BizTalk node is currently the owner <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image213.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb213.png" width="244" height="164" border="0" /></a> Now go back to your main BizTalk Node (BTS001), and browse to the location in which you copied dtcping.exe and start it. The MSDTC Simulation screen will pop up, now enter the SQL Server Cluster Name in the box ‘Remote Server Name’, in my case that would be SQL2008.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image214.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb214.png" width="244" height="150" border="0" /></a>

Now go back to your main SQL Server Node (SQL001), and browse to the location in which you copied dtcping.exe and start it. The MSDTC Simulation screen will pop up, now enter the BTS Server Cluster Name in the box ‘Remote Server Name’, in my case that would be BTS2010. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image215.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb215.png" width="244" height="152" border="0" /></a>

Now go back to your main BizTalk Node (BTS001) and in the MSDTC Simulation windows. Click on the ‘PING’ button. Once you’ve clicked that button; ‘hurry’ over to your main SQL Server Node (SQL001) and in the MSDTC Simulation windows. Click on the ‘PING’ button. If everything goes right, you should see the following results in the MSDTC Simulation Window.

&nbsp;

<strong><em>BTS001</em></strong>

<em>Please refer to following log file for details:
C:UsersAdministrator.LABDownloadsBTS0013924.log
Invoking RPC method on SQL2008
<strong>RPC test is successful</strong>
++++++++++++RPC test completed+++++++++++++++
Please start PING from SQL2008 to complete the test
Please send following LOG to Microsoft for analysis:
Partner LOG: SQL0014040.log
My LOG: BTS0013924.log
++++++++++++Start Reverse Bind Test+++++++++++++
Received Bind call from SQL001
Trying Reverse Bind to SQL001
</em><em><strong>Reverse Binding success: BTS001–&gt;SQL001
</strong>++++++++++++Reverse Bind Test ENDED++++++++++
++++++++++++Start DTC Binding Test +++++++++++++
Trying Bind to SQL001
Received reverse bind call from SQL001
</em><em><strong>Binding success: BTS001–&gt;SQL001
</strong>++++++++++++DTC Binding Test END+++++++++++++</em>

<strong><em>SQL001</em></strong>

<em>++++++++++++Validating Remote Computer Name++++++++++++
Please refer to following log file for details:
C:UsersAdministrator.LABDownloadsSQL0014040.log
Please send following LOG to Microsoft for analysis:
Partner LOG: BTS0013924.log
My LOG: SQL0014040.log
Invoking RPC method on BTS2010
<strong>RPC test is successful</strong>
++++++++++++RPC test completed+++++++++++++++
++++++++++++Start DTC Binding Test +++++++++++++
Trying Bind to BTS001
Received reverse bind call from BTS001
</em><em><strong>Binding success: SQL001–&gt;BTS001
</strong>++++++++++++DTC Binding Test END+++++++++++++
++++++++++++Start Reverse Bind Test+++++++++++++
Received Bind call from BTS001
Trying Reverse Bind to BTS001
<strong>Reverse Binding success: SQL001–&gt;BTS001</strong>
++++++++++++Reverse Bind Test ENDED++++++++++ </em>
<blockquote>Please note; in case of an error.
<ul>
	<li>Ensure that MSDTC is configured correctly! (see previous blog posts)</li>
	<li>Ensure that your Firewall is turned of for your Domain Profile and Private Profile! (see previous blog posts)</li>
</ul>
</blockquote>
Close both DTCPing applications once done.
<h3></h3>
<h3>Perform a manual Failover of the SQL Server</h3>
Logon to your main SQL Server node (in my case SQL001) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ The Failover Cluster Manager windows will appear; in this window ‘right-click’ on your SQL Server Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to Node SQL002’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image216.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb216.png" width="244" height="198" border="0" /></a>

Confirm this action.

&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image217.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb217.png" width="244" height="155" border="0" /></a>

At this point you will failover your SQLCluster to your second node. Before continuing validate this, by checking the ‘Current Owner’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image218.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb218.png" width="244" height="219" border="0" /></a>
<h3>BTS001 vs SQL002</h3>
Now repeat the same steps as mentioned in the previous chapter BTS001 vs SQL001, the only difference now is; that you will start dtcping on your second SQL node (SQL002).

&nbsp;
<h3>Perform a manual Failover of the BizTalk Server</h3>
Logon to your main BizTalk Server node (in my case BTS001) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ The Failover Cluster Manager windows will appear; in this window ‘right-click’ on your BizTalk Server Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to Node BTS002’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image219.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb219.png" width="244" height="200" border="0" /></a>

Confirm this action.

&nbsp;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image220.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb220.png" width="244" height="156" border="0" /></a>

At this point you will failover your BizTalk Cluster to your second node. Before continuing validate this, by checking the ‘Current Owner’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image221.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb221.png" width="244" height="201" border="0" /></a>
<h3></h3>
<h3>BTS002 vs SQL002</h3>
Now repeat the same steps as mentioned in the previous chapter BTS001 vs SQL001, the only difference now is; that you will start dtcping on your second BizTalk node, and your second SQL node (SQL002).

&nbsp;
<h3>Perform a manual Failover of the SQL Server.</h3>
Logon to your main SQL Server node (in my case SQL001) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ The Failover Cluster Manager windows will appear; in this window ‘right-click’ on your SQL Server Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to Node SQL001’. Confirm this change and validate that the SQL001 node is once again the ‘Current Owner’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image222.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb222.png" width="244" height="212" border="0" /></a>
<h3></h3>
<h3>BTS002 vs SQL001</h3>
Now repeat the same steps as mentioned in the previous chapter BTS001 vs SQL001, the only difference now is; that you will start dtcping on your second BizTalk node, and your main SQL node (SQL001).

&nbsp;
<h3>Perform a manual Failover of the BizTalk Server</h3>
Logon to your main BizTalk Server node (in my case BTS001) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ The Failover Cluster Manager windows will appear; in this window ‘right-click’ on your BizTalk Server Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to Node BTS001’. Confirm this change and validate that the BTS001 node is once again the ‘Current Owner’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image223.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb223.png" width="244" height="201" border="0" /></a>
<h3></h3>
<h3>Almost there!</h3>
Congratulations, at this point we have validated our MSDTC configuration and now we are almost ready for the actual BizTalk Installation.

&nbsp;
<h2>Installing BizTalk</h2>
The steps mentioned below, need to be performed on both BizTalk servers, in my case BTS001 and BTS002
<blockquote><em>Time Saver Tip: You can install both servers simultaneously. However note that this can not be done when we are <strong>configuring</strong> BizTalk.</em></blockquote>
<h3>Prerequisites</h3>
Well before we can proceed with the installation we need to install 2 more pre-requisites if we want to install BAM and EDI functionality on our BizTalk servers as well. These prerequisites are:
<ul>
	<li>Microsoft SQL Server Data Transformation Services (DTS 2008) with SP1 or higher</li>
	<li>Microsoft Office Excel 2010</li>
</ul>
&nbsp;
<h4>Installing Microsoft Office Excel 2010</h4>
<blockquote>Make sure that you’ve mounted your BizTalk Server 2010 Installation ISO file, this can be easily done from within the Hyper-V Manager. Simply ‘Right-Click’ on the BizTalk server in question and select ‘Settings’<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image224.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb224.png" width="244" height="201" border="0" /></a> Go to ‘IDE Controller 1’ and mount your media (see screenshot below) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image225.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb225.png" width="244" height="230" border="0" /></a></blockquote>
Once you’ve mounted the Microsoft Office 2010 DVD, run the installer. On the ‘Choose the installation you want screen’ select ‘Customize’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image226.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb226.png" width="244" height="199" border="0" /></a> Now only install Microsoft Excel, you’re screen should look similar to the following screenshot: <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image227.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb227.png" width="244" height="199" border="0" /></a>

Once done select ‘Install Now’ and finally ‘Close’ the installation. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image228.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb228.png" width="244" height="201" border="0" /></a>

and select ‘New Installation or add features to an existing installation’

&nbsp;
<h4>Installing Microsoft SQL Server Data Transformation Services (DTS 2008)</h4>
Well we’re almost done, but we need to install ‘Microsoft SQL Server Data Transformation Services (DTS 2008) with SP1 or higher, if we want to configure and use BAM functionality. So let’s do this now.
<blockquote>Make sure that you’ve mounted your BizTalk Server 2010 Installation ISO file, this can be easily done from within the Hyper-V Manager. Simply ‘Right-Click’ on the BizTalk server in question and select ‘Settings’<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image229.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb229.png" width="244" height="201" border="0" /></a>
Go to ‘IDE Controller 1’ and mount your media (see screenshot below) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image230.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb230.png" width="244" height="228" border="0" /></a></blockquote>
Once you’ve mounted the SQL Server 2008 R2 DVD, run the installer and select ‘New Installation or add features to an existing installation’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image231.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb231.png" width="244" height="184" border="0" /></a>

Follow the onscreen instructions. Once you reach the ‘Setup Role’ screen select ‘SQL Server Feature Installation’. Once done select ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image232.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb232.png" width="244" height="184" border="0" /></a>

On the ‘Feature Selection’ screen ensure to check the following features
<ul>
	<li>Client Tools Connectivity</li>
	<li>Management Tools – Basic</li>
	<li>Management Tools – Complete</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image233.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb233.png" width="244" height="181" border="0" /></a>

Click ‘next’ and follow the onscreen instructions.
<h3></h3>
<h3>BizTalk 2010</h3>
Now logon to one of the servers on which you want to install BizTalk, in my case I will start off with BTS001.
<blockquote>Make sure that you’ve mounted your BizTalk Server 2010 Installation ISO file, this can be easily done from within the Hyper-V Manager. Simply ‘Right-Click’ on the BizTalk server in question and select ‘Settings’<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image234.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb234.png" width="244" height="201" border="0" /></a> Go to ‘IDE Controller 1’ and mount your media (see screenshot below)<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image235.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb235.png" width="244" height="229" border="0" /></a>
(Repeat these steps for your second BizTalk Server)</blockquote>
Start the installation, by running ‘Setup.exe’ from the BizTalk Installation DVD. An installation screen will appear, in this screen select ‘Install Microsoft BizTalk Server 2010’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image236.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb236.png" width="244" height="175" border="0" /></a>

On the Customer Information Screen, enter the required information and press ‘next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image237.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb237.png" width="244" height="198" border="0" /></a>

Accept the License agreement and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image238.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb238.png" width="244" height="198" border="0" /></a>

Choose whether you want to participate in the ‘Customer Experience Improvement Program’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image239.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb239.png" width="244" height="199" border="0" /></a>

On the ‘Component Installation’ screen, check all items you can <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile2.png" /> and press ‘Next’
<blockquote>I was able to check all with exception of; <em>Developer Tools and SDK, MQSeries Agent, Windows Sharepoint Services Adapter</em> as I did not install the pre-requisites for them.</blockquote>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image240.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb240.png" width="244" height="199" border="0" /></a>

On the ‘Redistributable Prerequisites’ screen, select your preferred option. I’ve selected ‘Automatically install the redistributable prerequisites from a CAB file’ which I downloaded previously (click <a href="http://sandroaspbiztalkblog.wordpress.com/2010/10/15/biztalk-2010-prerequisite-redistributable-cab-files/" target="_blank" rel="noopener noreferrer">here</a>) and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image241.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb241.png" width="244" height="197" border="0" /></a>

On the ‘Summary Screen’ press ‘Install’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image242.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb242.png" width="244" height="198" border="0" /></a>

Installation will now start, during the installation you will be prompted for ‘Updates’, choose the preferred option and press ‘next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image243.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb243.png" width="244" height="199" border="0" /></a>

Click finish to Quit the installation and ensure that the option ‘Launch BizTalk Server Configuration’ is not checked. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image244.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb244.png" width="244" height="198" border="0" /></a>

&nbsp;
<h3>Installing Microsoft BizTalk Adapters</h3>
The steps mentioned below, need to be performed on both BizTalk servers, in my case BTS001 and BTS002 Continue the installation, by running ‘Setup.exe’ from the BizTalk Installation DVD. An installation screen will appear, in this screen select ‘Install Microsoft BizTalk Adapters’.<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image245.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb245.png" width="244" height="176" border="0" /></a>

A new window will appear, select ‘Step 1: Install Microsoft WCF Lob Adapter SDK’ and follow the onscreen instructions. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image246.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb246.png" width="244" height="166" border="0" /></a>

When you reach the ‘Choose Setup Type’ screen, select ‘Complete’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image247.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb247.png" width="244" height="190" border="0" /></a>

Click on ‘Install’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image248.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb248.png" width="244" height="191" border="0" /></a>

Installation will now start, during the installation you will be prompted for ‘Updates’, choose the preferred option and press ‘Ok’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image249.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb249.png" width="244" height="91" border="0" /></a>

Click ‘Finish’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image250.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb250.png" width="244" height="189" border="0" /></a>

On the ‘Adapter Installation Window’, note the warning which can be ignored and select ‘Step 2: Install Microsoft BizTalk Adapter Pack’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image251.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb251.png" width="244" height="166" border="0" /></a>

Follow the onscreen instruction and once you reach the ‘Choose Setup Type’ screen, select ‘Complete’ .

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image252.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb252.png" width="244" height="189" border="0" /></a>

Click on ‘Install’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image253.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb253.png" width="244" height="190" border="0" /></a>

Installation will now start, during the installation you will be prompted for ‘Updates’, choose the preferred option and press ‘Ok’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image254.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb254.png" width="244" height="91" border="0" /></a>

Another window will pop up, asking if you want to join ‘The Customer Experience Improvement Program’; select either option and press ‘ok’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image255.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb255.png" width="244" height="151" border="0" /></a>

Click on ‘Finish’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image256.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb256.png" width="244" height="190" border="0" /></a>

On the ‘Adapter Installation Window’, select ‘Step 3: Install Microsoft BizTalk Adapter Pack (x64)’ and follow the onscreen instructions. I will not list them again as they are similar to the previous instructions. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image257.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb257.png" width="244" height="166" border="0" /></a>

On the ‘Adapter Installation Window’, select ‘Step 4: Install Microsoft BizTalk Adapters for Enterprise Applications’ and follow the onscreen instructions. I will not list them again as they are similar to the previous instructions.(when asked for the Setup type; select ‘Complete’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image258.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb258.png" width="244" height="170" border="0" /></a>

During the installation of step 4, you might get a few warnings, indicating that an adapter was already installed. Just press ‘Ok’ and continue. At this point you installed all adapters, so press ‘Exit’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image259.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb259.png" width="244" height="176" border="0" /></a>

Note: We will not install the ‘Microsoft AppFabric Connect’ feature; in a later blog post however I might get back to this <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile2.png" /> Before we can proceed with configuring BizTalk Server, we need to epeat the above mentioned steps on your other BizTalk Server node (my case BTS002)

&nbsp;
<h2>Configuring BizTalk on the Main Node</h2>
Once you’ve installed BizTalk on both servers. Logon to your main BizTalk Node, and click on ‘Start’ –&gt; ‘All Programs’ –&gt; ‘Microsoft BizTalk Server 2010’ –&gt; ‘BizTalk Server Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image260.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb260.png" width="195" height="244" border="0" /></a>

On the ‘Microsoft BizTalk Server 2010 Configuration’ screen, select ‘Custom Configuration’, fill out
<blockquote>
<ul>
	<li>The Database server name (point it to your SQL Cluster!), in my case ‘SQL2008BIZTALK2010’</li>
	<li>Enter the username and password for the BizTalk Untrusted Service, in my case ‘LABsrvc-bts-untrusted’</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image261.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb261.png" width="244" height="224" border="0" /></a></blockquote>
Once done, press Configure.
<h3></h3>
<h3>Enterprise SSO Configuration</h3>
At this point the main configuration screen will appear. Select ‘Enterprise SSO’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image262.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb262.png" width="244" height="202" border="0" /></a>

Select ‘Enable Enterprise Single Sign-On on this computer’ and select ‘Create a new SSO System’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image263.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb263.png" width="244" height="201" border="0" /></a>

Now, add the correct Service Accounts and Windows Accounts; in my case:
<blockquote>
<ul>
	<li>Enterprise Single Sign-On Service Account: LABsrvc-bts-sso</li>
	<li>SSO Administrator(s) group: LABSSO Administrators</li>
	<li>SSO Affiliate Administrator(s)</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image264.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb264.png" width="244" height="201" border="0" /></a> Note the warning: Ignore this warning, as it is most likely a ‘refreshing issue’ as it states that I am using a local SSO Administrator group, which I am not <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile2.png" /></blockquote>
Select ‘Enterprise Single Sign-On Secret Backup’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image265.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb265.png" width="244" height="201" border="0" /></a>

Enter the details and store the backup file on a Secure Place. I’ve used the Clustered Disk named' ‘Data_Store’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image266.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb266.png" width="244" height="128" border="0" /></a>

Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image267.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb267.png" width="244" height="200" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image268.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb268.png" width="244" height="209" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image269.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb269.png" width="244" height="210" border="0" /></a>
<h3></h3>
<h3>Group Configuration</h3>
Select ‘Group’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image270.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb270.png" width="244" height="202" border="0" /></a>

Check ‘Enable BizTalk Server Group on this Computer’ and select ‘Create a new BizTalk Group’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image271.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb271.png" width="244" height="201" border="0" /></a>

Now, add the correct BizTalk Administrative Roles; in my case:
<blockquote>
<ul>
	<li>BizTalk Administrators Group: LABBizTalk Server Administrators</li>
	<li>BizTalk Operators Group: LABBizTalk Server Operators</li>
	<li>BizTalk B2B Operators Group: LABBizTalk Server B2B Operators</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image272.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb272.png" width="244" height="200" border="0" /></a></blockquote>
Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image273.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb273.png" width="244" height="200" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image274.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb274.png" width="244" height="210" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image275.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb275.png" width="244" height="211" border="0" /></a>

&nbsp;
<h3>BizTalk Runtime Configuration</h3>
Select ‘BizTalk Runtime’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image276.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb276.png" width="244" height="200" border="0" /></a>

Check ‘Register the BizTalk Server runtime components’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image277.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb277.png" width="244" height="200" border="0" /></a>

Check ‘Create In-Process Host and Instance’. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image278.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb278.png" width="244" height="200" border="0" /></a>

For Host Name enter: Processing_Host <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image279.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb279.png" width="244" height="200" border="0" /></a>

Check ‘Create Isolated Host and Instance’ and check ‘Trusted’ right below it <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image280.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb280.png" width="244" height="200" border="0" /></a>

For Isolated Host name enter: Isolated_Host <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image281.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb281.png" width="244" height="200" border="0" /></a>

Ensure that both 32-bit only checkboxes are checked<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image282.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb282.png" width="244" height="201" border="0" /></a>

Now, assign the correct Windows Services and Groups ; in my case:
<blockquote>
<ul>
	<li>BizTalk Host Instance Account: LABsrvc-bts-untrusted</li>
	<li>Biztalk Isolated Host Instance Account: LABsrvc-bts-trusted</li>
	<li>Biztalk Host Users Group: LABBizTalk Applications Users</li>
	<li>Biztalk Isolated Host Users Group: LABBizTalk Isolated Host Users</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image283.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb283.png" width="244" height="201" border="0" /></a></blockquote>
Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image284.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb284.png" width="244" height="201" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image285.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb285.png" width="244" height="208" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image286.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb286.png" width="244" height="211" border="0" /></a>

&nbsp;
<h3>Business Rules Engine Configuration</h3>
Select ‘Business Rules Engine’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image287.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb287.png" width="244" height="201" border="0" /></a>

Check ‘Enable Business Rules Engine on this computer’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image288.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb288.png" width="244" height="200" border="0" /></a>

Now, add the correct Windows Service; in my case:
<blockquote>
<ul>
	<li>Rule Engine Update Service : LABsrvc-bts-rule-engine</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image289.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb289.png" width="244" height="200" border="0" /></a></blockquote>
Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image290.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb290.png" width="244" height="201" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image291.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb291.png" width="244" height="211" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image292.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb292.png" width="244" height="210" border="0" /></a>

&nbsp;
<h3>BAM Tools Configuration</h3>
Select ‘Bam Tools’
<h3><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image293.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb293.png" width="244" height="201" border="0" /></a></h3>
Check ‘Enable Business Activity Monitoring tools’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image294.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb294.png" width="244" height="202" border="0" /></a>

Now, add the correct ‘Server Name (SQL Cluster); in my case: SQL2008BIZTALK2010

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image295.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb295.png" width="244" height="200" border="0" /></a>

Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image296.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb296.png" width="244" height="201" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image297.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb297.png" width="244" height="211" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image298.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb298.png" width="244" height="209" border="0" /></a>
<h3></h3>
<h3>BAM Alerts Configuration</h3>
We will skip these for now.
<blockquote>In a future post I might come back to these and configure it. However in case you need it installation is quite straight forward, you only need to make sure to install the pre-requisites</blockquote>
<h3>BAM Portal Configuration</h3>
Select ‘Bam Portal’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image299.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb299.png" width="244" height="200" border="0" /></a>

Check ‘Enable BAM Portal’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image300.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb300.png" width="244" height="200" border="0" /></a>

Now, add the correct Windows Service Accounts ; in my case:
<blockquote>
<ul>
	<li>BAM Management Web Service user: LABsrvc-bts-bam</li>
	<li>BAM Application Pool Account: LABsrvc-bts-bam-ap</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image301.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb301.png" width="244" height="201" border="0" /></a></blockquote>
Next add the correct Windows Group; in my case:
<blockquote>BAM Portal Users: LABDomain Users <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image302.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb302.png" width="244" height="201" border="0" /></a></blockquote>
Finally select the BAM Portal Web site, in my case
<blockquote>Bam Portal Web Site: Default Web Site <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image303.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb303.png" width="244" height="201" border="0" /></a></blockquote>
Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image304.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb304.png" width="244" height="201" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image305.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb305.png" width="244" height="210" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image306.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb306.png" width="244" height="210" border="0" /></a>
<h3></h3>
<h3>BizTalk EDI/AS2 Runtime Configuration</h3>
Select ‘BizTalk EDI/AS2 Runtime’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image307.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb307.png" width="244" height="200" border="0" /></a>

Check all options.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image308.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb308.png" width="244" height="201" border="0" /></a>

Now Click on ‘Apply Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image309.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb309.png" width="244" height="201" border="0" /></a>

Verify the settings on the ‘Summary Screen’ and press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image310.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb310.png" width="244" height="211" border="0" /></a>

Click Finish

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image311.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb311.png" width="244" height="211" border="0" /></a>
<h3></h3>
<h3>Export the configuration</h3>
Now click on Export Configuration

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image312.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb312.png" width="244" height="201" border="0" /></a>

Save it the file (I’ve saved it on a cluster disk) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image313.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb313.png" width="244" height="189" border="0" /></a>
<h2></h2>
<h2>Configuring BizTalk on the Second Node</h2>
Before we start with configuring BizTalk on the Second Node, we will first perform a failover of the BizTalk Cluster such that all resources are available to our second node. Logon to your second BizTalk Node (in my case BTS002) and open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image314.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb314.png" width="198" height="244" border="0" /></a>

Ensure that the main BizTalk node (BTS001) is the current owner <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image315.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb315.png" width="244" height="195" border="0" /></a>

Now fail this cluster over, so that the second node becomes the owner. ‘Right-click’ on your BizTalk Server Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to Node BTS002’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image316.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb316.png" width="244" height="195" border="0" /></a>

Confirm this action

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image317.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb317.png" width="244" height="155" border="0" /></a>

Verify once all resources are back online that current owner is the Second BizTalk Node (BTS002).

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image318.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb318.png" width="244" height="194" border="0" /></a>

Now click click on ‘Start’ –&gt; ‘All Programs’ –&gt; ‘Microsoft BizTalk Server 2010’ –&gt; ‘BizTalk Server Configuration’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image319.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb319.png" width="195" height="244" border="0" /></a>

On the ‘Microsoft BizTalk Server 2010 Configuration’ screen, select ‘Custom Configuration’, fill out
<blockquote>
<ul>
	<li>The Database server name (point it to your SQL Cluster!), in my case ‘SQL2008BIZTALK2010’</li>
	<li>Enter the username and password for the BizTalk Untrusted Service, in my case ‘LABsrvc-bts-untrusted’</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image320.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb320.png" width="244" height="224" border="0" /></a></blockquote>
Once done, press Configure.
<h3></h3>
<h3>Enterprise SSO Configuration</h3>
At this point the main configuration screen will appear. Select ‘Enterprise SSO’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image321.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb321.png" width="244" height="202" border="0" /></a>

Select ‘Enable Enterprise Single Sign-On on this computer’ and select ‘<strong><span style="text-decoration: underline;">Join an existing SSO system’</span></strong>

<strong></strong> <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image322.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb322.png" width="244" height="201" border="0" /></a>

Now, verify that the configurations are correct (should point to the same database as mentioned in the previous chapter) and add the correct Service Accounts; in my case:
<blockquote>
<ul>
	<li>Enterprise Single Sign-On Service Account: LABsrvc-bts-sso</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image323.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb323.png" width="244" height="201" border="0" /></a></blockquote>
Now Click on ‘Apply Configuration’, verify the settings on the ‘Summary Screen’ ,press ‘Next’ followed by ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image324.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb324.png" width="244" height="210" border="0" /></a>
<h3></h3>
<h3>Group Configuration</h3>
Select ‘Group’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image325.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb325.png" width="244" height="202" border="0" /></a>

Check ‘Enable BizTalk Server Group on this Computer’ and select ‘<strong><span style="text-decoration: underline;">Join an existing BizTalk Group</span></strong>’ and verify that the configurations are correct (should point to the same databases as mentioned in the previous chapter) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image326.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb326.png" width="244" height="201" border="0" /></a>

Now Click on ‘Apply Configuration’, verify the settings on the ‘Summary Screen’ ,press ‘Next’ followed by ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image327.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb327.png" width="244" height="211" border="0" /></a>
<h3></h3>
<h3>BizTalk Runtime Configuration</h3>
Select ‘BizTalk Runtime’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image328.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb328.png" width="244" height="200" border="0" /></a>

Check ‘Register the BizTalk Server runtime components’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image329.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb329.png" width="244" height="200" border="0" /></a>

Check ‘Create In-Process Host and Instance’. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image330.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb330.png" width="244" height="202" border="0" /></a>

For Host Name enter: Processing_Host (ignore warning) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image331.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb331.png" width="244" height="202" border="0" /></a>

Check ‘Create Isolated Host and Instance’ and check ‘Trusted’ right below it <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image332.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb332.png" width="244" height="202" border="0" /></a>

For Isolated Host name enter: Isolated_Host (Ignore the warning) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image333.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb333.png" width="244" height="202" border="0" /></a>

Ensure that both 32-bit only checkboxes are checked <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image334.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb334.png" width="244" height="202" border="0" /></a>

Now, assign the correct Windows Services and Groups ; in my case:
<blockquote>
<ul>
	<li>BizTalk Host Instance Account: LABsrvc-bts-untrusted</li>
	<li>Biztalk Isolated Host Instance Account: LABsrvc-bts-trusted</li>
	<li>Biztalk Host Users Group: LABBizTalk Applications Users</li>
	<li>Biztalk Isolated Host Users Group: LABBizTalk Isolated Host Users</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image335.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb335.png" width="244" height="199" border="0" /></a></blockquote>
Now Click on ‘Apply Configuration’, Verify the settings on the ‘Summary Screen’ ,press ‘Next’ and end with ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image336.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb336.png" width="244" height="211" border="0" /></a>
<h3></h3>
<h3>Business Rules Engine Configuration</h3>
Select ‘Business Rules Engine’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image337.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb337.png" width="244" height="201" border="0" /></a>

Check ‘Enable Business Rules Engine on this computer’ and verify that the configurations are correct (should point to the same databases and should use the correct account as mentioned in the previous chapter) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image338.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb338.png" width="244" height="201" border="0" /></a>

Now Click on ‘Apply Configuration’. Verify the settings on the ‘Summary Screen’ ,press ‘Next’ and end with ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image339.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb339.png" width="244" height="210" border="0" /></a>
<h3></h3>
<h3>BAM Tools Configuration</h3>
Select ‘Bam Tools’
<h3><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image340.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb340.png" width="244" height="201" border="0" /></a></h3>
Check ‘Enable Business Activity Monitoring tools’ and verify that the configurations are correct (should point to the same databases as mentioned in the previous chapter)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image341.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb341.png" width="244" height="200" border="0" /></a>

Now Click on ‘Apply Configuration’. Verify the settings on the ‘Summary Screen’ ,press ‘Next’ and end with ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image342.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb342.png" width="244" height="209" border="0" /></a>
<h3></h3>
<h3>BAM Alerts Configuration</h3>
We will skip these for now.
<blockquote>In a future post I might come back to these and configure it. However in case you need it installation is quite straight forward, you only need to make sure to install the pre-requisites</blockquote>
<h3>BAM Portal Configuration</h3>
Select ‘Bam Portal’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image343.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb343.png" width="244" height="200" border="0" /></a>

Check ‘Enable BAM Portal’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image344.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb344.png" width="244" height="201" border="0" /></a>

Now Click on ‘Apply Configuration’. Verify the settings on the ‘Summary Screen’ ,press ‘Next’ and end with ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image345.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb345.png" width="244" height="210" border="0" /></a>
<h3></h3>
<h3>BizTalk EDI/AS2 Runtime Configuration</h3>
Select ‘BizTalk EDI/AS2 Runtime’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image346.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb346.png" width="244" height="200" border="0" /></a>

Check ‘Enable BizTalk EDI/AS2 Runtime on this computer; (other options are greyed out).

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image347.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb347.png" width="244" height="201" border="0" /></a>

Now Click on ‘Apply Configuration’. Verify the settings on the ‘Summary Screen’ ,press ‘Next’ and end with ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image348.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb348.png" width="244" height="211" border="0" /></a>
<h3></h3>
<h3>Export the configuration</h3>
Now click on Export Configuration

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image349.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb349.png" width="244" height="201" border="0" /></a>

Save it the file (I’ve saved it on a cluster disk) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image350.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb350.png" width="244" height="190" border="0" /></a>
<h3></h3>
<h3>Failover Back to your main BizTalk Node</h3>
Open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image351.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb351.png" width="198" height="244" border="0" /></a>

Ensure that the main BizTalk node (BTS002) is the current owner <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image352.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb352.png" width="244" height="194" border="0" /></a>

Now fail this cluster over, so that the second node becomes the owner. ‘Right-click’ on your BizTalk Server Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to Node BTS001’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image353.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb353.png" width="244" height="195" border="0" /></a>

Confirm this action

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image354.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb354.png" width="244" height="158" border="0" /></a>

Verify once all resources are back online that current owner is the Main BizTalk Node (BTS001).

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image355.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb355.png" width="244" height="194" border="0" /></a>
<h2></h2>
<h2>Setting Up BizTalk</h2>
We’ll now that we’ve installed and configured BizTalk it’s time to finish up our BizTalk Cluster.

&nbsp;
<h3>Clustering our Single Sign On Service</h3>
Ensure you’re logged on to the BizTalk Server node which is currently the owner; in my case this is BTS002. Once logged open up one of the best editors in the world <img class="wlEmoticon wlEmoticon-winkingsmile" alt="Winking smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-winkingsmile.png" /> called notepad. In notepad copy and paste the following code

&nbsp;
<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,'Courier New',courier,monospace; font-size: 10px;"><span style="color: #0000ff;">&lt;</span><span style="color: #800000;">sso</span><span style="color: #0000ff;">&gt;</span></pre>
<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,'Courier New',courier,monospace; font-size: 10px;">  <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">globalInfo</span><span style="color: #0000ff;">&gt;</span></pre>
<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,'Courier New',courier,monospace; font-size: 10px;">     <span style="color: #0000ff;">&lt;</span><span style="color: #800000;">secretServer</span><span style="color: #0000ff;">&gt;</span>SSOCLUSTER<span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">secretServer</span><span style="color: #0000ff;">&gt;</span></pre>
<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,'Courier New',courier,monospace; font-size: 10px;">  <span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">globalInfo</span><span style="color: #0000ff;">&gt;</span></pre>
<pre style="background-color: #fbfbfb; margin: 0em; width: 100%; font-family: consolas,'Courier New',courier,monospace; font-size: 10px;"><span style="color: #0000ff;">&lt;/</span><span style="color: #800000;">sso</span><span style="color: #0000ff;">&gt;</span></pre>
change the SSOCLUSTER value in the &lt;secretServer&gt; xml-tag to the name of your BizTalk Server Cluster. In my case this would be ‘BTS2010’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image356.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb356.png" width="244" height="139" border="0" /></a>

Save this file to disk (I saved it to c:tmp) and name it ‘SSO_Secret_Server.xml’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image357.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb357.png" width="244" height="162" border="0" /></a>

Once stored open up a command prompt do this by going to start and typing ‘CMD’ into the search box followed by hitting ‘Enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image358.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb358.png" width="194" height="244" border="0" /></a>

A command prompt box will open; At this command prompt, change to the Enterprise SSO installation folder. In my case I typed "c:Program FilesCommon FilesEnterprise Single Sign-On” <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image359.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb359.png" width="244" height="123" border="0" /></a>

Now enter the following command ‘ssomanage –updatedb &lt;SSO_Secret_Server.xml location&gt; (my case ssomanage –updatedb “c:tmpSSO_Secret_Server.xml”) hit ‘enter’ and once done close the window.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image360.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb360.png" width="244" height="40" border="0" /></a>

Open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image351.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb351.png" width="198" height="244" border="0" /></a>

In the ‘Failover Cluster Manager’ right click on your BizTalk Cluster Service (in my case BTS2010) and select ‘Add a resource’ –&gt; ‘4 – Generic Service’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image361.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb361.png" width="220" height="244" border="0" /></a>

On the ‘Select Service window’, select ‘Enterprise Single Sign On Service’ and press ‘Next’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image362.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb362.png" width="244" height="169" border="0" /></a>

On the ‘Confirmation’ screen press ‘Next’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image363.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb363.png" width="244" height="169" border="0" /></a>

On the ‘Summary’ screen press ‘Finish’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image364.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb364.png" width="244" height="169" border="0" /></a>

Now in the main pane; you will see that the ‘Enterprise Single Sign-On Service’ is added as a resource.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image365.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb365.png" width="244" height="147" border="0" /></a>

Right click on it and select ‘properties’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image366.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb366.png" width="244" height="205" border="0" /></a>

On the ‘General’ Tab; ensure to check ‘Use Network Name for computer name’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image367.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb367.png" width="197" height="244" border="0" /></a>

Confirm this action by click ‘Yes’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image368.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb368.png" width="244" height="131" border="0" /></a>

Click on ‘Ok’ on the ‘Information Screen’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image369.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb369.png" width="244" height="85" border="0" /></a>

Go to the ‘Dependencies’ Tab, add the following depended resources and once done press ‘apply’
<ul>
	<li>name (in my case BTS2010)</li>
	<li>dtc resource (in my case MSDTC-BTS2010)</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image370.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb370.png" width="196" height="244" border="0" /></a>

Now start the SSO resource by right-clicking on it and selecting ‘Bring this resource online’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image371.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb371.png" width="244" height="203" border="0" /></a>

Once this resource is online, right click on the BizTalk Cluster Service and select ‘Move this service or application to another node’ –&gt; ‘1 – Move to node BTS002’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image372.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb372.png" width="244" height="230" border="0" /></a>

Confirm the action

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image373.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb373.png" width="244" height="157" border="0" /></a>

Now log on to the BizTalk node you just failed over to (in my base BTS002). On that server open a command prompt do this by going to start and typing ‘CMD’ into the search box followed by hitting ‘Enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image374.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb374.png" width="194" height="244" border="0" /></a>

A command prompt box will open; At this command prompt, change to the Enterprise SSO installation folder. In my case I typed "c:Program FilesCommon FilesEnterprise Single Sign-On” <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image375.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb375.png" width="244" height="123" border="0" /></a>

Now enter the following command ‘ssoconfig –restoresecret <var>&lt;RestoreFile (Backup of your SSO Secret you saved earlier)</var>&gt; (my case ssoconfig –restoresecret "f:SSO SecretSSO7884.bak”) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image376.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb376.png" width="244" height="123" border="0" /></a>

Hit enter,and enter the Secret Password and hit enter <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image377.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb377.png" width="244" height="123" border="0" /></a>

You should see the following message <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image378.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb378.png" width="244" height="39" border="0" /></a>
<blockquote>In case you receive an adapter, ensure that you followed all steps as mentioned above!</blockquote>
<h3></h3>
<h3>Modify the BizTalk Application Configuration</h3>
In order to optimize the throughput with regards to HTTP based send ports, we need to add a configuration section to both our BTSNTSvc.exe.config and BTSNTSvc64.exe.config files which can be found in the installation directory if BizTalk. (in my case “C:Program Files (x86)Microsoft BizTalk Server 2010”) So open-up windows explorer and browse to your BizTalk installation directory and upon BTSNTSvc.exe.config by right clicking it and selecting ‘open with…’ –&gt; ‘notepad’ (if notepad is not visible, select ‘Choose default program… and select then notepad’) <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image379.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb379.png" width="244" height="100" border="0" /></a>

Now add the following code section just above the &lt;/configuration&gt; closing tag and save the file.

&nbsp;
<pre>&lt;system.net&gt;</pre>
<pre>    &lt;connectionManagement&gt;</pre>
<pre>      &lt;add address="*" maxconnection="25" /&gt;</pre>
<pre>    &lt;/connectionManagement&gt;</pre>
<pre>&lt;/system.net&gt;</pre>
Once done, your file should look something like this: <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image380.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb380.png" width="244" height="95" border="0" /></a>

One of the standard settings which come with a BizTalk Installation is the setting which indicated the maximum connections allowed for HTTP based send adapters. This setting is by default set to Now repeat this step for the BTSNTSvc64.exe.config Once done, log on to your other BizTalk server and repeat the above mentioned steps.

&nbsp;
<h3></h3>
<h3>Adding BizTalk Hosts</h3>
At this point we are ready to finally open up the BizTalk Administrator and finish our BizTalk Cluster <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile3.png" /> So let’s get Started. First of all log on to your active BizTalk Node and open the BizTalk Administrator Console. You can find it by going to ‘Start’ –&gt; ‘All Programs’ –&gt; ‘Microsoft BizTalk Server 2010’ and clicking on ‘BizTalk Server Administrator’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image381.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb381.png" width="195" height="244" border="0" /></a>

In the BizTalk Server Administration Console, expand the ‘BizTalk Server Node’, expand ‘BizTalk Group’, expand ‘Platform Settings’ and select ‘Hosts’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image382.png"><img style="background-image: none; margin: 5px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb382.png" width="244" height="123" border="0" /></a>

In the main pane, you should now see two Hosts (these hosts have been created while we configured BizTalk earlier on). <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image383.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb383.png" width="244" height="138" border="0" /></a>

We now will add a few more hosts, beings
<ul>
	<li>Send_Host</li>
	<li>Receive_Host</li>
	<li>Tracking_Host</li>
	<li>Legacy_Host</li>
	<li>x64_Host</li>
</ul>
Adding a new host is quite simple; just right-click on the main pane and select ‘New’ –&gt; ‘Host’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image384.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb384.png" width="244" height="168" border="0" /></a>

On the Host Properties screen, change/add the following properties. Once done click on ‘Ok
<blockquote>Name: Send_Host Type: In_Process Options: 32-bit only Track Windows group: &lt;add your BizTalk Application Users Group&gt; <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image385.png"><img style="background-image: none; margin: 5px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb385.png" width="244" height="196" border="0" /></a></blockquote>
On the Host Properties screen, change/add the following properties. Once done click on ‘Apply’
<blockquote>Name: Receive_Host Type: In_Process Options: 32-bit only Windows group: &lt;add your BizTalk Application Users Group&gt; <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image386.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb386.png" width="244" height="197" border="0" /></a></blockquote>
On the Host Properties screen, change/add the following properties. Once done click on ‘Ok’
<blockquote>Name: Tracking_Host Type: In_Process Options: 32-bit only and <strong><span style="text-decoration: underline;">Check Allow Host Tracking</span></strong> Windows group: &lt;add your BizTalk Application Users Group&gt;

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image387.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb387.png" width="244" height="196" border="0" /></a></blockquote>
On the Host Properties screen, change/add the following properties. Once done click on ‘Ok’
<blockquote>Name: Legacy_Host Type: In_Process Options: 32-bit only Windows group: &lt;add your BizTalk Application Users Group&gt; <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image388.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb388.png" width="244" height="195" border="0" /></a></blockquote>
On the Host Properties screen, change/add the following properties. Once done click on ‘Ok’
<blockquote>Name: x64_Host Type: In_Process Options: 32-bit only <strong>DO NOT CHECK!</strong> Windows group: &lt;add your BizTalk Application Users Group&gt; <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image389.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb390.png" width="244" height="197" border="0" /></a></blockquote>
Now double click on the ‘Processing Host’ and <strong><span style="text-decoration: underline;">remove</span></strong> the check in front of ‘Allow Host Tracking’ and click ok.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image390.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb391.png" width="244" height="196" border="0" /></a>

A warning pops up indicating that a restart is required of the associated host instances, press ‘Ok’.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image391.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb392.png" width="244" height="52" border="0" /></a>
<h3></h3>
<h3>Adding BizTalk Host-Instances</h3>
<blockquote>First of all log on to your active BizTalk Node and open the BizTalk Administrator Console. You can find it by going to ‘Start’ –&gt; ‘All Programs’ –&gt; ‘Microsoft BizTalk Server 2010’ and clicking on ‘BizTalk Server Administrator’</blockquote>
In the BizTalk Server Administration Console, expand the ‘BizTalk Server Node’, expand ‘BizTalk Group’, expand ‘Platform Settings’ and select ‘Host Instances’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image392.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb393.png" width="244" height="123" border="0" /></a>

You should see, four host-instances; and you should notice that we have 2 exactly the same host instances on both BizTalk Server instances. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image393.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb394.png" width="244" height="138" border="0" /></a>

Adding a new host instance is quite simple; just right-click on the main pane and select ‘New’ –&gt; ‘Host Instance’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image394.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb395.png" width="244" height="139" border="0" /></a>

On the Host Instance Properties screen, select the following:
<blockquote>Host Name: Legacy_Host Server: BTS001   <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image395.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb396.png" width="244" height="197" border="0" /></a></blockquote>
Now click on ‘Configure’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image396.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb397.png" width="244" height="197" border="0" /></a>

On the ‘Logon Credentials’ screens enter the details of the windows service account you want this host instance to execute with (the Untrusted BizTalk Service Account in AD), in my case this will be LABsrcv-bts-untrusted. Once done press ‘OK’ and once more ‘OK’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image397.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb398.png" width="244" height="183" border="0" /></a>

Now repeat the above steps, but this time on the ‘Host Instance Properties’ screen select ‘<strong><span style="text-decoration: underline;">BTS002’ for the server propery</span></strong> <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image398.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb399.png" width="244" height="196" border="0" /></a>
<blockquote>Note: Don’t forgot to hit configure and enter the service credentials <img class="wlEmoticon wlEmoticon-winkingsmile" alt="Winking smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-winkingsmile.png" /></blockquote>
Repeat the above mentioned steps for the remaining other hosts, being:
<ul>
	<li>Tracking_Host</li>
	<li>Send_Host</li>
	<li>Receive_Host</li>
	<li>x64_Host</li>
</ul>
&nbsp;

Once done, your main pane should look similar to the image below <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image399.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb400.png" width="244" height="125" border="0" /></a>
<h3></h3>
<h3>Adding the correct Host Instances to the Adapters</h3>
<blockquote>First of all log on to your active BizTalk Node and open the BizTalk Administrator Console. You can find it by going to ‘Start’ –&gt; ‘All Programs’ –&gt; ‘Microsoft BizTalk Server 2010’ and clicking on ‘BizTalk Server Administrator’</blockquote>
In the BizTalk Server Administration Console, expand the ‘BizTalk Server Node’, expand ‘BizTalk Group’, expand ‘Platform Settings’ and expand the ‘Adapters’ node
<h4></h4>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image400.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb401.png" width="132" height="244" border="0" /></a>

Click on the ‘File’ Adapter, and then right-click in the main-pane and select ‘New’ –&gt; ‘Send Handler’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image401.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb402.png" width="244" height="108" border="0" /></a>

On the ‘File – Adapter Handler Properties’ For Host name select Send_Host and ensure to check ‘Make this the default Handler’. Once done press ‘Ok’<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image402.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb403.png" width="244" height="196" border="0" /></a>

A warning will pop-up, just read it and click “ok”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image403.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb404.png" width="244" height="50" border="0" /></a>

Right-click in the main-pane once more and select ‘New’ –&gt; ‘Receive Handler’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image404.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb405.png" width="244" height="98" border="0" /></a>

On the ‘File – Adapter Handler Properties’ For Host name select ‘Recieve_Host’. Once done press ‘Ok’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image405.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb406.png" width="244" height="197" border="0" /></a>

A warning will pop-up, just read it and click “ok” <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image406.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb407.png" width="244" height="50" border="0" /></a>

In the main-pane right click on ‘Processing_Host’ for the direction ‘Send’ and select ‘Delete’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image407.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb408.png" width="244" height="89" border="0" /></a>

Confirm the ‘Deletion’ by selecting ‘Yes’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image408.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb409.png" width="244" height="135" border="0" /></a>

Note the ‘warning’ and press ‘Ok’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image409.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb410.png" width="244" height="53" border="0" /></a>

In the main-pane right click on ‘Processing_Host’ for the direction ‘Receive’ and select ‘Delete’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image410.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb411.png" width="244" height="75" border="0" /></a>

Confirm the ‘Deletion’ by selecting ‘Yes’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image411.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb412.png" width="244" height="135" border="0" /></a>

Note the ‘warning’ and press ‘Ok’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image412.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb413.png" width="244" height="53" border="0" /></a>

<strong><span style="text-decoration: underline;">Now repeat the above steps for the following adapters, but note the host_names as listed below</span></strong>

&nbsp;
<table width="480" border="0" cellspacing="0" cellpadding="2">
<tbody>
<tr>
<td valign="top" width="103">Adapter</td>
<td valign="top" width="153">Send Host Name</td>
<td valign="top" width="112">Receive Host Name</td>
<td valign="top" width="110">Remarks</td>
</tr>
<tr>
<td valign="top" width="103">FTP</td>
<td valign="top" width="153">Legacy_Host (<em>make default)</em></td>
<td valign="top" width="112">Legacy_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">HTTP</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">No change</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">MQSeries</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">Receive_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">MSMQ</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">Receive_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">POP3</td>
<td valign="top" width="153">Not Applicable</td>
<td valign="top" width="112">Legacy_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">SMTP</td>
<td valign="top" width="153">Legacy_Host (<em>make default)</em></td>
<td valign="top" width="112">Not Applicable</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">SOAP</td>
<td valign="top" width="153">Legacy_Host (<em>make default)</em></td>
<td valign="top" width="112">No Change</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">SQL</td>
<td valign="top" width="153"></td>
<td valign="top" width="112"></td>
<td valign="top" width="110">Can’t delete Receive Processing_Host entries as they are used (see note below)</td>
</tr>
<tr>
<td valign="top" width="103">WCF-BasicHTTP</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">No Change</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">WCF-Custom</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">Receive_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">WCF-CustomIsolated</td>
<td valign="top" width="153">Not Applicable</td>
<td valign="top" width="112">No Change</td>
<td valign="top" width="110"></td>
</tr>
<tr>
<td valign="top" width="103">WCF-NetMsmq</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">Receive_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">WCF-NetNamedPipe</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">Receive_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">WCF-NetTcp</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">Receive_Host</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">WCF-WSHttp</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">No Change</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
<tr>
<td valign="top" width="103">Windows SharePoint Services</td>
<td valign="top" width="153">Send_Host (<em>make default)</em></td>
<td valign="top" width="112">No Change</td>
<td valign="top" width="110">Delete Processing_Host entries</td>
</tr>
</tbody>
</table>
<blockquote>Note: In order to delete the Receive Processing_Host, we first need to change the binding of the receive locations of the ‘Biztalk EDI Application’, in order to do so. Click on ‘Applications’ –&gt; &lt;All Artifacts&gt;’ –&gt; and select ‘Receive Locations’. <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image413.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb414.png" width="244" height="125" border="0" /></a>

<em>Perform the following actions for all receive locations visible in the main pane</em> Double click receive-location and change the Receive handler to: <strong>Receive_Host </strong>and click ‘Ok’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image414.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb415.png" width="244" height="196" border="0" /></a>

Now go back to the SQL Adapter and delete the ‘Processing Host’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image415.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb416.png" width="244" height="144" border="0" /></a></blockquote>
<h3>Clustering BizTalk Server Hosts</h3>
At this point we are almost done with our tasks in the BizTalk Server Administration Console. The one thing which we now need to do is actually cluster the hosts. In the BizTalk Administration Console, click on the Hosts node

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image416.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb417.png" width="244" height="81" border="0" /></a>

Perform the following actions for <strong>all</strong> hosts, with exception of the Isolated_Host Right Click on a host and select ‘Cluster’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image417.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb418.png" width="244" height="84" border="0" /></a>

Select the Cluster Resource Group to use, in my case ‘BTS2010’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image418.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb419.png" width="244" height="190" border="0" /></a>

Select ‘Ok’ Once you’re done; you should see the that the ‘Clustered’ Column<strong> </strong>says yes for all hosts except for the ‘Isolated Host’ as out Isolated Host can not be clustered (but we’ve taken care of this in a previous step when we’ve clustered IIS <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile3.png" /> )

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image419.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb420.png" width="244" height="66" border="0" /></a>

At this point we are almost done, we only need to verify everything in our Cluster Failover Manager and add some dependencies<strong>.</strong>
<h2></h2>
<h2>Verifying our BizTalk Cluster Resources and adding dependencies</h2>
At this point we are almost done, we only need to verify everything in our Cluster Failover Manager and some dependencies to the just clustered hosts Open the ‘Failover Cluster Manager’ do this by going to start and in the search box type ‘Failover Cluster Manager’ then hit ‘enter’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image351.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb351.png" width="198" height="244" border="0" /></a>

In the Failover Cluster Manager, select your BizTalk Cluster Service and notice all the BTSSvc$…. resources added <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile3.png" /> <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image420.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb421.png" width="244" height="138" border="0" /></a>

Well, for all these resources we will add a few dependencies. So let’s get started. ‘Right Click’ on the first BTSSvc$… resource and select ‘properties’ <a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image421.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb422.png" width="244" height="192" border="0" /></a>

On the properties screen select the ‘Dependencies’ Tab and add the following dependencies and once done click ‘Ok’
<ul>
	<li>MSDTC (in my case ‘MSDTC-BTS2010’)</li>
	<li>Enterprise Single Sign-On Service</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image422.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/05/image_thumb423.png" width="196" height="244" border="0" /></a>

Repeat the above mentioned step, for all other BTSSvc$… resources.
<h3></h3>
<h2></h2>
<h2>Closing note</h2>
Congratulations, at this point you've created a complete BizTalk High Availability Environment. Well it took some time and effort, but hey you accomplished it :-) and that at least deserves a congratulation. So what's up next you might be wondering, well most likely I won't be posting a new article pretty soon as I am currently involved in reviewing one of the upcoming BizTalk 2010 books and this takes up more time than I initially expected :-) But hey, isn't that with all the things we 'developers' do ;-) Anyway, if you have any suggestions on future article with regards to BizTalk, Programming in general or anything else; well just drop me a note info@brauwers.nl or try to contact me on Twitter (ReneBrauwers). Well it has been a pleasure and it took me quite some time (avg 10hrs per post) to put all these posts together so I really hope that one of these posts or all have or can help you in anyway. If you have any questions or feedback please feel free to leave a comment and I'll try to address them. So until the next time and don't forget to check back regularly for new posts (or even better just follow me in Twitter as I'll tweet whenever there is a new blogpost :-) Cheers René