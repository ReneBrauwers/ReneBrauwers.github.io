---
ID: 9
post_title: 'Part 5: BizTalk High Availability Server Environment &#8211; SQL Server 2008r2 Failover Cluster'
post_name: >
  part-5-biztalk-high-availability-server-environment-sql-server-2008r2-failover-cluster
author: Rene Brauwers
post_date: 2011-05-01 16:33:28
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/05/01/part-5-biztalk-high-availability-server-environment-sql-server-2008r2-failover-cluster/
published: true
tags:
  - BizTalk
  - Cluster Installation
  - File Share Mojority
  - Installation how to
  - iSCSI
  - >
    Microsoft Distributed Transaction
    Coordinator
  - Quorum
  - SQL Server
  - SQL Server 1008 R2
  - Windows Firewall
categories:
  - BizTalk
---
<p>In Part 4 we’ve prepped our SQL &amp; BizTalk Servers so that they can be used as a basis for setting up our actual Failover Clusters.</p>
<p>This post will assume that you’ve followed all steps as mentioned in Part 1 through 4. Well let’s get started with installing and configuring our SQL Server 2008 R2 Cluster <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile.png" /></p>
<h2>Installing SQL Server 2008 R2</h2>
<p>One of the most crucial parts when installing SQL Server in combination with BizTalk, is to ensure that you’ve made the proper firewall configurations and at least configured the local Microsoft Distributed Transaction Coordinator.</p>
<h3>Configuring the Firewall</h3>
<p>In our lab environment I’ve simply turned of the Firewall for the following profiles</p>
<ul>
<li>Domain Profile</li>
<li>Private Profile</li>
</ul>
<p>&nbsp;</p>
<p>In order to do so, startup tour first basic SQL Server instance, log on to the domain, open up the windows firewall, by going to Start and in the search box simple type: ‘Windows Firewall with Advanced Security’ followed by hitting ‘enter’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb.png" width="193" height="244" border="0" /></a></p>
<p>Within the MMC-Snap in, click on ‘Windows Firewall Properties’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image1.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb1.png" width="244" height="161" border="0" /></a></p>
<p>A window will appear. Go to the first tab named ‘Domain Profile’ and set the firewall state to ‘Off’ and click on apply</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image2.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb2.png" width="219" height="244" border="0" /></a></p>
<p>Go to the second tab named ‘Private Profile’ and set the firewall state to ‘Off’ and click on apply and then ok</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image3.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb3.png" width="218" height="244" border="0" /></a></p>
<p>Close your Firewall MMC snap-in.</p>
<h3>Configuring the (local) Microsoft Distributed Transaction Coordinator</h3>
<p>go to start and type into the search box ‘Component Services’ and hit enter.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image4.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb4.png" width="189" height="244" border="0" /></a></p>
<p>the Component Services MMC snap in will open; now extend the ‘Component Service’ node, do the same for the node ‘Computers’ and ‘My Computer’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image5.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb5.png" width="244" height="171" border="0" /></a></p>
<p>Expand the ‘Distributed Transaction Coordinator’ , right click on ‘local DTC’ and select ‘properties’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image6.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb6.png" width="244" height="176" border="0" /></a></p>
<p>Within the Properties window go the the ‘Security Tab’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image7.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb7.png" width="222" height="244" border="0" /></a></p>
<p>On this tab, check (enable) the following item:</p>
<ul>
<li>Network DTC Access</li>
<li>Allow Inbound</li>
<li>Allow Outbound</li>
<li>No Authentication Required</li>
<li>Enable XA Transactions</li>
<li>Enable SNA LU 6.2 Transactions</li>
</ul>
<p>&nbsp;</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image8.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb8.png" width="225" height="244" border="0" /></a></p>
<p>Click on ‘Ok, a message box will apear stating that the MSDTC service needs to be stopped and started. Click on Yes</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image9.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb9.png" width="244" height="100" border="0" /></a></p>
<p>Close the Component Services Snap in. And repeat the above mentioned steps on the following other servers</p>
<ul>
<li>Second SQL Server (SQL002)</li>
<li>First BizTalk Server (BTS001)</li>
<li>Second BizTalk Server (BTS002)</li>
</ul>
<p>&nbsp;</p>
<h3>Creating your SQL Server Cluster</h3>
<p>Before we start with installing SQL Server we will have to actually create our SQL Cluster. In order to do this logon to one of your servers which you want to be part of your SQL Cluster. In my particular case this is SQL001</p>
<p>Go to start and in the search box type ‘Failover Cluster Manager’ and then hit ‘enter’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image10.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb10.png" width="198" height="244" border="0" /></a></p>
<p>In your Failover Cluster Manager, first click on ‘Validate a configuration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image11.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb11.png" width="244" height="206" border="0" /></a></p>
<p>On the ‘Before you begin’ screen, press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image12.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb12.png" width="244" height="170" border="0" /></a></p>
<p>Now Enter the server names (or browse) which you want to be part of your cluster. In my case that would be ‘SQL001 and SQL002’ and then select ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image13.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb13.png" width="244" height="171" border="0" /></a></p>
<p>on the ‘Testing Options’ screen, select the ‘Run all tests’ option and select ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image14.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb14.png" width="244" height="170" border="0" /></a></p>
<p>Confirm the settings and then select ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image15.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb15.png" width="244" height="171" border="0" /></a></p>
<p>Once the validation process has finished you will notice a few warning relating to the storage. Ignore these warnings for now as we will take care of these one we’ve created our Cluster. Once you’ve examend the report (View Report) click on Finish</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image16.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb16.png" width="244" height="170" border="0" /></a></p>
<p>From within your ‘Failover Cluster Manager’ select the ‘Create a Cluster’ link</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image17.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb17.png" width="244" height="204" border="0" /></a></p>
<p>On the ‘Before you begin’ screen, press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image18.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb18.png" width="244" height="163" border="0" /></a></p>
<p>Now Enter the server names (or browse) which you want to be part of your cluster. In my case that would be ‘SQL001 and SQL002’ and then select ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image19.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb19.png" width="244" height="163" border="0" /></a></p>
<p>On the ‘Access Point for Administering the Cluster’ enter a Cluster name, and a designated IP Address and click ‘next’ once done.</p>
<blockquote><p>I’ve used the following:</p>
<p>Cluster Name: CLUSTER_SQL<br />
IP Address: 192.168.8.22</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image20.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb20.png" width="244" height="164" border="0" /></a></p></blockquote>
<p>Confirm your settings and then click ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image21.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb21.png" width="244" height="164" border="0" /></a></p>
<p>On the ‘Summary’ screen, press ‘Finish’ (note the warnings, but no worries as we will address them in a bit)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image22.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb22.png" width="244" height="163" border="0" /></a></p>
<h3>Addressing the Storage issue on your SQL Cluster</h3>
<p>In order to finish prepping our SQL Cluster, we need to address two issues which were mentioned in the previous step. The issue we need to address is:</p>
<ul>
<li>Assigning Storage</li>
</ul>
<h4>Verify your connected with the File Server</h4>
<p>First verify that we’ve set-up our link with our Fileserver. Do this by clicking on Start and in the search box type ‘iSCSI Initiator’ and hit ‘enter’ (note: perform these steps on your main SQL node (in my case SQL001)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image23.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb23.png" width="213" height="244" border="0" /></a></p>
<p>Ensure that you are connected to your ‘Target’, by clicking on the ‘Targets’ tab and checking the status</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image24.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb24.png" width="174" height="244" border="0" /></a></p>
<p>Repeat the above mentioned steps for your other SQL node (in my case SQL002)</p>
<h4>Assigning Storage to your Servers</h4>
<p>Go back to the main SQL Server Node, open the ‘Server Manager’ , expand the ‘Storage’ node and select ‘Disk Management’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image25.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb25.png" width="204" height="210" border="0" /></a></p>
<p>At this point you should notice several disks which are not Initialized.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image26.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb26.png" width="244" height="186" border="0" /></a></p>
<p>Right Click on Disk 1 and select ‘Initialize</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image27.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb27.png" width="156" height="132" border="0" /></a></p>
<p>The ‘Initialize Disk’ screen will appear, and enables you to initialize the other disks as well. Make sure to check all disks, and use the MBR partition option. Once done, click ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image28.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb28.png" width="244" height="182" border="0" /></a></p>
<p>Now right click in the area next to Disk 1 and select ‘Simple Volume’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image29.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb29.png" width="244" height="105" border="0" /></a></p>
<p>The ‘New Simple Volume Wizard’ will pop up; click ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image30.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb30.png" width="244" height="187" border="0" /></a></p>
<p>On the ‘Specify Volume Size’ click ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image31.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb31.png" width="244" height="188" border="0" /></a></p>
<p>On the ‘Assign a drive letter or path’ screen; assign a drive letter and click ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image32.png">&lt;img style=&quot;background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;&quot; title=&quot;image&quot; alt=&quot;image&quot; src=&quot;http://blog.brauwers <a href="http://biturlz.com/C0q72Hh">viagra 100 mg posologie</a>.nl/wp-content/uploads/2011/05/image_thumb32.png" width="244" height="188" border="0" /&gt;</a></p>
<p>On the ‘Format Partition’ screen; leave the Default Values intact with exception of the ‘Volume Label’ for this enter a name (fe; Disk1) and press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image33.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb33.png" width="244" height="188" border="0" /></a></p>
<p>Finish the wizard by clicking ‘Finish’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image34.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb34.png" width="244" height="188" border="0" /></a></p>
<p>Repeat the ‘new Simple Volume’ steps for all other Disks which are ‘Unallocated’</p>
<p>Once done; your disk management screen should look similar like to this</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image35.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb35.png" width="244" height="222" border="0" /></a></p>
<p>Open your iSCSI initiator once again, do this by clicking on Start and in the search box type ‘iSCSI Initiator’ and hit ‘enter’ Once in the iSCSI Initiator properties screen pops up, go to the ‘Volume and Devices Tab’ and click on the ‘auto configure’ button.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image36.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb36.png" width="173" height="244" border="0" /></a></p>
<p>Your Volume List should now be populated with the disks you assigned earlier. Once done click ‘OK’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image37.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb37.png" width="174" height="244" border="0" /></a></p>
<p>At this point, go to your second SQL Server node (in my case SQL002). Open your iSCSI initiator, do this by clicking on Start and in the search box type ‘iSCSI Initiator’ and hit ‘enter’ Once in the iSCSI Initiator properties screen pops up, go to the ‘Targets’ tab and verify that your connected. If not; hit ‘Refresh’ and then Connect to the target.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image38.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb38.png" width="173" height="244" border="0" /></a></p>
<p>Now go to the ‘Volume and Devices Tab’ and verify that the Volume List is populated.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image39.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb39.png" width="174" height="244" border="0" /></a></p>
<p>&nbsp;</p>
<h4>Add the assigned storage as a disk resource in your Cluster</h4>
<p>Go back to the main SQL Server Node (in my case SQL001)  open up the ‘Cluster Manager’, expend the CLUSTER_SQL node and right click on Storage and select ‘Add a disk’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image40.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb40.png" width="244" height="206" border="0" /></a></p>
<p>A list of available disks will appear, ensure that they are all selected and press ‘ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image41.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb41.png" width="228" height="244" border="0" /></a></p>
<h3>Installing SQL Server on your Cluster</h3>
<p><em>The steps mentioned below, need to be executed on all your Servers which will be part of your SQL Server Cluster</em></p>
<p>Make sure you’ve mounted the SQL Server 2008r2 ISO file, this image can be downloaded form MSDN if you have a subscription.</p>
<p>Once you’ve mounted SQL Server 2008R2, open up windows explorer and browse to the mounted Drive (in my case drive D) and double click ‘Setup’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image42.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb42.png" width="244" height="167" border="0" /></a></p>
<p>You will be prompted with a message indicating that the .NET Framework is required and an updated version of the windows installer. Click on ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image43.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb43.png" width="244" height="108" border="0" /></a></p>
<p>After a while the ‘SQL Server Installation Center’ will pop up. Click on the ‘Advanced Link’.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image44.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb44.png" width="244" height="183" border="0" /></a></p>
<p>Select ‘Advanced cluster preparation’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image45.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb45.png" width="244" height="113" border="0" /></a></p>
<p>The window ‘Setup Support Rules’ will appear. Wait till it finished, ensure that there are no warnings and then click on ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image46.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb46.png" width="244" height="184" border="0" /></a></p>
<p>After a wile a window will pop up in which you will be asked for the product key. In case no product key is filled out, enter your product key and then press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image47.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb47.png" width="244" height="184" border="0" /></a></p>
<p>Accept the license terms and press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image48.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb48.png" width="244" height="184" border="0" /></a></p>
<p>Install the setup support files, by clicking on ‘Install’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image49.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb49.png" width="244" height="183" border="0" /></a></p>
<p>The support files will no be installed, and once done check the warnings, if everything went well you should only see one warning; being the Windows Firewall warning. This warning can be ignored and click ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image50.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb50.png" width="244" height="184" border="0" /></a></p>
<blockquote>
<h4>warnings</h4>
</blockquote>
<blockquote><p>In case you get a warning relating to ‘Microsoft .NET Application Security’, verify that your machine has access to the Internet. Fix the issue and rerun the validation process</p></blockquote>
<blockquote><p>In case you get a warning relating to ‘Network binding order’, check out the following links</p>
<p><a title="http://theregime.wordpress.com/2008/03/04/how-to-setview-the-nic-bind-order-in-windows/" href="http://theregime.wordpress.com/2008/03/04/how-to-setview-the-nic-bind-order-in-windows/">http://theregime.wordpress.com/2008/03/04/how-to-setview-the-nic-bind-order-in-windows/</a>.</p>
<p><a title="http://support.microsoft.com/kb/955963" href="http://support.microsoft.com/kb/955963">http://support.microsoft.com/kb/955963</a></p></blockquote>
<blockquote><p>Fix the issue and rerun the validation process (you might need to reboot first and rerun the Cluster Installation Preparation.</p>
<p>Note: if after the binding order changes you still receive the same error; just skip and proceed with the installation as this error is most likely at this point showing up due to the fact that the ‘Failover Feature’ installs a virtual NIC .</p></blockquote>
<p>&nbsp;</p>
<p>You will be presented with the Feature Selection screen, for sake of simplicity we will check all options and thus do a Feature Complete installation <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile.png" /> Once everything has been selected, click on ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image51.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb51.png" width="244" height="182" border="0" /></a></p>
<p>The next screen will be the ‘Instance Configuration’ screen, ensure to check the option ‘Named Instance’ and give it the following name ‘BizTalk2010’. Once done press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image52.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb52.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Disk Space Requirements’ screen, press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image53.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb53.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Cluster Security Policy’ screen, select ‘Use service Sids’ and press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image54.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb54.png" width="244" height="184" border="0" /></a></p>
<p>On the Server Configuration screen; select the ‘Service Accounts’ tab and set the required Accounts and Passwords to the corresponding service</p>
<p>SQL Service Agent: <strong>LABsrvc-sql-agent<br />
</strong>SQL Server Database Engine: <strong>LABsrvc-sql-engine<br />
</strong>SQL Server Analysis Services: <strong>LABsrvc-sql-analysis<br />
</strong>SQL Server Reporting Services: <strong>LABsrvc-sql-reporting</strong></p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image55.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb55.png" width="244" height="184" border="0" /></a></p>
<p>On the Server Configuration screen; select the ‘File Stream’ tab and ensure that the option ‘Enable FILESTREAM’ for Transact-SQL access is <strong>disabled</strong> as we will not use this feature. Press ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image56.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb56.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Reporting Services Configuration’ screen, select ‘Install, but do not configure the report server’ and click ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image57.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb57.png" width="244" height="183" border="0" /></a></p>
<p>On the ‘Error Reporting’ screen, press next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image58.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb58.png" width="244" height="184" border="0" /></a></p>
<p>Ensure that no warning appear on the ‘prepare failover cluster rules’ screen and press next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image59.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb59.png" width="244" height="183" border="0" /></a></p>
<p>Verify the features and select ‘Install’ (Please note: This can take a while)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image60.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb60.png" width="244" height="183" border="0" /></a></p>
<p>Once the installation is complete, press the ‘close’ button and repeat the above mentioned steps for your other sql server.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image61.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb61.png" width="244" height="184" border="0" /></a></p>
<p>&nbsp;</p>
<h3>SQL Server 2008r2 Cluster completion</h3>
<p>Ensure to logon to your SQL Primary Node server, in my case that is the SQL001 server.</p>
<p>Before we start with the ‘Cluster Completion’ installation we will verify the following:</p>
<ul>
<li>SQL Server Configuration</li>
</ul>
<p>&nbsp;</p>
<h4>Verify SQL Server Configuration</h4>
<p>Open up the SQL Server Configuration Manager( Start –&gt; All Programs –&gt; Microsoft SQL Server 2008 R2 –&gt; Configuration Tools)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image62.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb62.png" width="216" height="244" border="0" /></a></p>
<p>Open the SQL Server Network Configuration en select ‘Protocols for BIZTALK2010’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image63.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb63.png" width="244" height="102" border="0" /></a></p>
<p>Ensure that the following items are <strong>enabled</strong></p>
<ul>
<li>Named Pipes</li>
<li>TCP/IP</li>
</ul>
<p>&nbsp;</p>
<p>Ensure that the following items are <strong>disabled</strong></p>
<ul>
<li>Shared Memory</li>
<li>VIA</li>
</ul>
<p>&nbsp;</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image64.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb64.png" width="244" height="98" border="0" /></a></p>
<p>&nbsp;</p>
<h4>Proceed with the Cluster Completion Installation</h4>
<p>Open up the SQL Server Installation Center ( Start –&gt; All Programs –&gt; Microsoft SQL Server 2008 R2 –&gt; Configuration Tools)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image65.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb65.png" width="218" height="244" border="0" /></a></p>
<p>Click on the Advanced link, and select the option ‘Advanced Cluster Completion’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image66.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb66.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Setup Support Rules’ screen, click on ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image67.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb67.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Setup Support Files’ click on install</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image68.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb68.png" width="244" height="183" border="0" /></a></p>
<p>On the ‘Setup Support Rules’ check for any warnings and click on ‘Next’.</p>
<blockquote><p>You might see one warning, this warning relates to the Cluster Validation. You can ignore this warning as it mentions a storage issue, but we’ve tackled this issue earlier</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image69.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb69.png" width="244" height="183" border="0" /></a></p></blockquote>
<p>On the ‘Cluster Node Configuration’. Select the correct SQL Server instance name and assign a SQL Server Network Name and press Next.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image70.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb70.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Cluster Resource Group’ Screen, click Next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image71.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb71.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Cluster Disk Selection’ select the storage intended for your database, and select ‘next’.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image72.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb72.png" width="244" height="184" border="0" /></a></p>
<blockquote><p>In my case I assigned 2Gb for the SQL Data; in order to backtrack which Disk Resource to use; check the sizes of the disks in the Failover Cluster Manager(in my case this would be Disk 2)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image73.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb73.png" width="244" height="151" border="0" /></a></p></blockquote>
<p>&nbsp;</p>
<p>On the ‘Cluster Network Configuration’ Screen, ensure to uncheck ‘DHCP’ and assign a static IP address. In case you have multipe Networks, make sure to only fill out the details for the internal network (in my case I disabled Cluster Network 2)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image74.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb74.png" width="244" height="184" border="0" /></a></p>
<blockquote><p>I’ve used the following:<br />
IP Address: 192.168.8.23</p></blockquote>
<p>On the ‘Server Configuration’ screen, click ‘next’.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image75.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb75.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Database Engine Configuration’ screen,</p>
<ul>
<li>select Mixed Mode and enter a password.</li>
<li>Click on ‘Add Current User’</li>
</ul>
<p>&nbsp;</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image76.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb76.png" width="244" height="184" border="0" /></a></p>
<p>Click on the ‘Data Directories’ tab, verify the settings and press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image77.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb77.png" width="244" height="184" border="0" /></a></p>
<p>On the ‘Analysis Service Configuration’ Screen, click on the ‘Add Current User’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image78.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb78.png" width="244" height="184" border="0" /></a></p>
<p>Click on the ‘Data Directories’ tab, verify the settings and press ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image79.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb79.png" width="244" height="184" border="0" /></a></p>
<p>Click Next on the ‘Complete Failover Cluster Rules’ Screen.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image80.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb80.png" width="244" height="184" border="0" /></a></p>
<p>Check the summary screen and press ‘install’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image81.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb81.png" width="244" height="184" border="0" /></a></p>
<p>Click ‘Close’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image82.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb82.png" width="244" height="184" border="0" /></a></p>
<h3>Finalizing your SQL Server 2008r2 Cluster</h3>
<p>Congratulations we’ve now have a SQL Cluster, however we need to verify a few things and manually add and change some resources. But all of this is explained below.</p>
<h4>Verify the IP settings</h4>
<p>In case you have 2 NICS available to the server, verify you assigned the correct NIC. If not you can skip the following steps.</p>
<p>Open the ‘Failover Cluster Manager’ and select the ‘SQL Server (BizTalk2010)’ node.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image83.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb83.png" width="244" height="205" border="0" /></a></p>
<p>In case you have 2 nics available to the server, verify you assigned the correct NIC, do this by expanding the Name node</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image84.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb84.png" width="244" height="80" border="0" /></a></p>
<p>Right Click on ‘IP Address’ and select properties</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image85.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb85.png" width="244" height="138" border="0" /></a></p>
<p>Verify that the Network settings are correct.</p>
<blockquote><p>In my case I know I need to have 192.168.8.0/24 as 192.168.8.x is used for my internal network and 192.168.1.x is used for my external network (internet access)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image86.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb86.png" width="180" height="244" border="0" /></a></p></blockquote>
<p>&nbsp;</p>
<h4>Add additional Storage to the Cluster instance</h4>
<p>Open the ‘Failover Cluster Manager’ and select the ‘SQL Server (BizTalk2010)’ node.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image87.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb87.png" width="244" height="205" border="0" /></a></p>
<p>Right Click on ‘SQL Server (BizTalk2010’) node, select ‘Add storage’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image88.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb88.png" width="202" height="244" border="0" /></a></p>
<p>Check the available disks and press ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image89.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb89.png" width="228" height="244" border="0" /></a></p>
<p>The Storage has been added.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image90.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb90.png" width="244" height="113" border="0" /></a></p>
<h5>[Optional] Rename the Disk Drives</h5>
<p>For readability I’ve renamed the Disk Drives, in order to rename a disk; richt-click on it and select properties</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image91.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb91.png" width="244" height="128" border="0" /></a></p>
<p>Change the Resource Name, and press ‘ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image92.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb92.png" width="205" height="244" border="0" /></a></p>
<p>Repeat these steps for all disks. Eventually you could have a result similar to this.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image93.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb93.png" width="244" height="114" border="0" /></a></p>
<h4>Add a Clustered Distributed Transaction Coordinator</h4>
<p>Open the ‘Failover Cluster Manager’ and select the ‘SQL Server (BizTalk2010)’ node.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image94.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb94.png" width="244" height="205" border="0" /></a></p>
<p>Right Click on ‘SQL Server (BizTalk2010’) node, select ‘Add a resource’-&gt; ‘More Resources’ –&gt; ‘2 – Add Distributed Transaction Coordinator’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image95.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb95.png" width="244" height="160" border="0" /></a></p>
<p>You will notice that a ‘MSDTC-SQL Server (BIZTALK2010) resource has been added.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image96.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb96.png" width="244" height="223" border="0" /></a></p>
<p>Right Click on this resource, and select ‘properties’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image97.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb97.png" width="244" height="81" border="0" /></a></p>
<p>Go to the ‘Dependencies’ tab, and add the following dependencies:</p>
<ul>
<li>
<ul>
<li>
<ul>
<li>Name</li>
</ul>
</li>
</ul>
</li>
<li>IP Address
<ul>
<li>Storage   (I’ve used the SQL DTC Store)</li>
</ul>
</li>
</ul>
<p>&nbsp;</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image98.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb98.png" width="205" height="244" border="0" /></a></p>
<p>Once done click ‘OK’ and bring the MSDTC resource online, by right-clicking on it and selecting ‘Bring this resource online’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image99.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb99.png" width="244" height="132" border="0" /></a></p>
<p>Go to Start and in the search box type ‘Component Services’ and hit ‘enter’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image100.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb100.png" width="184" height="244" border="0" /></a></p>
<p>The ‘Component Services’ screen will appear, now expand ‘Component Services’ –&gt; ‘Computers’ –&gt; ‘My Computer’ –&gt; ‘Distributed Transaction Coordinator’ –&gt; ‘Clustered DTCs’ right click on ‘SQL 2008’ and select ‘properties.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image101.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb101.png" width="244" height="209" border="0" /></a></p>
<p>On the ‘SQL 2008’ properties screen, select the ‘Security’ Tab</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image102.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb102.png" width="223" height="244" border="0" /></a></p>
<p>Enable the following options:</p>
<ul>
<li>Network DTC Access</li>
<li>Allow Remote Clients</li>
<li>Allow Remote Administration</li>
<li>Allow Inbound</li>
<li>Allow Outbound</li>
<li>No Authentication Required</li>
<li>Enable XA Transactions</li>
<li>Enable SNA LU 6.2 Transactions</li>
</ul>
<p>&nbsp;</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image103.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb103.png" width="222" height="244" border="0" /></a></p>
<p>Once done click ‘Ok’. A message will appear asking to stop/start to DTC service. Click ‘Yes’. Once done Close the Component Services screen and return to your ‘Cluster Manager’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image104.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb104.png" width="244" height="99" border="0" /></a></p>
<h4>Obtaining Quorum on your Cluster</h4>
<p>On the ‘Cluster Manager’ screen, select your main Cluster_SQL node and notice the warning with regards to the Quorum Configuration.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image105.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb105.png" width="244" height="206" border="0" /></a></p>
<p>In order to fix this; right click on ‘Cluster_SQL’ and select ‘More Actions’ –&gt; ‘Configure Cluster Quorum Settings’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image106.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb106.png" width="244" height="174" border="0" /></a></p>
<p>On the ‘Before You Begin’ screen, click ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image107.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb107.png" width="244" height="171" border="0" /></a></p>
<p>On the ‘Select Quorum Configuration’ screen, select ‘Node and File Share Majority’ (You could use Node and Disk Majority, but then you would have to create additional storage on your FileServer and configure your iSCSI target accordingly). Click ‘next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image108.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb108.png" width="244" height="172" border="0" /></a></p>
<p>On the ‘Configure File Share Witness’  browse to an available Shared Folder Path.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image109.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb109.png" width="244" height="171" border="0" /></a></p>
<blockquote><p>If you’ve not created a share at this point. Go to your FileServer, Create a folder and Share this Folder (<a title="http://technet.microsoft.com/en-us/library/cc770880.aspx#BKMK_interface" href="http://technet.microsoft.com/en-us/library/cc770880.aspx#BKMK_interface">http://technet.microsoft.com/en-us/library/cc770880.aspx#BKMK_interface</a>)</p></blockquote>
<p>On the ‘Browse for Shared Folders’ screen, enter your FileServer name; in my case ‘EUROPOORT’ and click on the ‘Show Shared Folders’ button. Select the share you would like to use; in my case the share is called ‘Majority_SQL’ and press ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image110.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb110.png" width="244" height="242" border="0" /></a></p>
<p>Click Next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image111.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb111.png" width="244" height="170" border="0" /></a></p>
<p>Confirm the settings and click on ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image112.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb112.png" width="244" height="170" border="0" /></a></p>
<p>Click Finish</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image113.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb113.png" width="244" height="171" border="0" /></a></p>
<h4>Verifying your Cluster and doing a manual Failover.</h4>
<p>At this point you’ve setup your SQL Server Cluster. Congratulations! No you might be wondering at this point of you need to perform the same actions on your second SQL node (in my case SQL002), well actually this has already auto magically been done for you.</p>
<p>So in order to verify this, go to your second SQL Server Node and open the Failover Cluster Manager <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile.png" /> and expand your ‘Cluster Node’, expand ‘Services and Applications’ and select the ‘SQL Server (BizTalk2010)’ node.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image114.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb114.png" width="244" height="206" border="0" /></a></p>
<p>Notice that the Current Owner is: SQL001 and that all resources are online <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/wlEmoticon-smile.png" /></p>
<p>If you look closely you see, that we haven’t assigned a Preferred Owner yet.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image115.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb115.png" width="244" height="105" border="0" /></a></p>
<p>In order to assign a preferred owner, right click on SQL Server (BizTalk2010) and select Properties</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image116.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb116.png" width="244" height="216" border="0" /></a></p>
<p>On the ‘Properties’ screen, set the Preferred owners to ‘SQL001’ and click ‘Ok’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image117.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb117.png" width="202" height="244" border="0" /></a></p>
<p>Well now we are up&amp;running! However let’s go and test if a failover works. In order to test this, right click on SQL Server (BizTalk2010) and select ‘Move this service or application to another node’ –&gt; ‘1-Move to node SQL002’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image118.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb118.png" width="244" height="192" border="0" /></a></p>
<p>A confirmation message will popup. Select ‘Move SQL Server (BIZTALK2010) to SQL002.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image119.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb119.png" width="244" height="158" border="0" /></a></p>
<p>Observe that changes to your resources</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image120.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb120.png" width="244" height="240" border="0" /></a></p>
<p>Once it is done, you will see that all resources are back online, and that the current owner is SQL002</p>
<p>Voila! Now you’re done!</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image121.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb121.png" width="244" height="242" border="0" /></a></p>
<p>&nbsp;</p>
<h2>Closing Note</h2>
<p>Well it has been another long read but this sums up part 5. We now have our SQL Cluster and we are ready to start on our BizTalk cluster but more on that in part 6.</p>
<blockquote><p><em>You’ve most likely noticed that we are currently not using all disk resources. Well no worries, these resources will be used in a future post. (Most likely part 7 as we will be playing around with the BizTalk Best Practices Analyzer)</em></p></blockquote>
<p>Well I hope you enjoyed the posts so far, check back soon and feel free to leave any comments, remarks and/or suggestions with regards to Blog posts you would like to see in the future.</p>
<p>Cheers</p>
<p>René</p>
