---
ID: 10
post_title: 'Part 6: BizTalk High Availability Server Environment–BizTalk 2010 Failover Cluster Creation'
post_name: >
  part-6-biztalk-high-availability-server-environmentbiztalk-2010-failover-cluster-creation
author: Rene Brauwers
post_date: 2011-05-14 01:44:04
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/05/14/part-6-biztalk-high-availability-server-environmentbiztalk-2010-failover-cluster-creation/
published: true
tags:
  - BizTalk
  - BizTalk Application Pool
  - BizTalk Clustering
  - DTC Clustering
  - IIS 7.5 Clustering
  - Installation how to
  - iSCSI
  - MSMQ Clustering
  - NO NLB
categories:
  - BizTalk
---
<p>Part 5 covered setting up our SQL-Server Cluster, so now it’s time to do the same but this time for our BizTalk 2010 Server environment.</p>
<p>Once again this post assumes you’ve followed all steps mentioned in the previous posts. All right let’s get on with it <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/wlEmoticon-smile1.png" /></p>
<p>Please note the following with regards to MSMQ: <a title="http://technet.microsoft.com/en-us/library/cc730960.aspx" href="http://technet.microsoft.com/en-us/library/cc730960.aspx">http://technet.microsoft.com/en-us/library/cc730960.aspx</a></p>
<blockquote><p>In case you’ve already installed the MSMQ feature as instructed in part 4, please uninstall the MSMQ feature and then apply the AD modifications as mentioned here: <a title="http://technet.microsoft.com/en-us/library/cc730960.aspx" href="http://technet.microsoft.com/en-us/library/cc730960.aspx">http://technet.microsoft.com/en-us/library/cc730960.aspx</a> . . Once done, add the MSMQ feature again as mentioned in part 4. Sorry for the inconvenience, but this part was unintended left out in the original part 4 (The current version of Part 4 has been updated)</p></blockquote>
<h2>Preparing our BizTalk Cluster</h2>
<p>Before we can actually start with Clustering BizTalk Server, we need to perform all kinds of other things, like:</p>
<ul>
<li>Verifying if all required Roles and Features have been installed</li>
<li>Verifying that we’ve configured the Firewall properly</li>
<li>Configure IIS</li>
<li>Configure the local Microsoft Distributed Transaction Coordinator</li>
<li>Add Storage using iSCSI</li>
</ul>
<p>&nbsp;</p>
<p>Once these steps are done, we will create our BizTalk Cluster Group.</p>
<h3>Verifying the required Server Roles and Features</h3>
<p>Logon to your first server on which BizTalk will be installed, in my case that would be BTS001. Once logged on; open the ‘server manager’ and expand the Roles node and ensure you’ve added the following roles:</p>
<ul>
<li>Application Server</li>
<li>Web Server (IIS)</li>
</ul>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image122.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb122.png" width="199" height="214" border="0" /></a></p>
<p>Now select ‘Application Server’ and in the main pane in the Server Manager scroll down to the Role Services. In total you should see that 13 role services are installed.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image123.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb123.png" width="244" height="204" border="0" /></a></p>
<p>if you’re missing any of the role-services mentioned above, add them by clicking on the ‘Add Role Services’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image124.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb124.png" width="244" height="167" border="0" /></a></p>
<p>Once double-checked, select the ‘Web Server (IIS) role and in the main pane in the Server Manager scroll down to the Role Services. In total you should see that 36 role services are installed</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image125.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb125.png" width="152" height="244" border="0" /></a></p>
<p>if you’re missing any of the role-services mentioned above, add them by clicking on the ‘Add Role Services’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image126.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb126.png" width="244" height="74" border="0" /></a></p>
<p>Once verified, expand the Features node in the Server Manager and select ‘features’. Ensure that you have the features as mentioned below in the image installed.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image127.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb127.png" width="244" height="176" border="0" /></a></p>
<p>if you’re missing any of the features as mentioned above, add them by clicking on the ‘Add Features’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image128.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb128.png" width="244" height="117" border="0" /></a></p>
<h4>Verifying that we’ve configured the Firewall properly</h4>
<p>Open up the windows firewall, by going to Start and in the search box simple type: ‘Windows Firewall with Advanced Security’ followed by hitting ‘enter’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image129.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb129.png" width="214" height="244" border="0" /></a></p>
<p>Within the MMC-Snap in, click on ‘Windows Firewall Properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image130.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb130.png" width="244" height="161" border="0" /></a></p>
<p>A window will appear. Go to the first tab named ‘Domain Profile’ and ensure the firewall state is set to ‘Off’. If not turn it ‘Off’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image131.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb131.png" width="222" height="244" border="0" /></a></p>
<p>Go to the second tab named ‘Private Profile’ and ensure the firewall state is set to ‘Off’. If not turn it ‘Off’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image132.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb132.png" width="220" height="244" border="0" /></a></p>
<h4>Configuring IIS</h4>
<p>Go back to the Server manager, expand Roles –&gt; Web Server (IIS) and select the ‘Internet Information Services (IIS) Manager’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image133.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb133.png" width="244" height="171" border="0" /></a></p>
<p>The Internet Information Services (IIS) Manager will appear in the main pain. Expand the Server Node and select Application Pools.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image134.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb134.png" width="244" height="123" border="0" /></a></p>
<p>In the Actions pane, click on the ‘Set Application Pool Defaults…’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image135.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb135.png" width="244" height="61" border="0" /></a></p>
<p>Change the following settings and once done click on ‘ok’:</p>
<ul>
<li>Net Framework Version: v4.0 (if this version is not available; stop your horses and go fetch the .Net 4.0 Framework <a title="http://www.microsoft.com/downloads/en/confirmation.aspx?displaylang=en&amp;FamilyID=0a391abd-25c1-4fc0-919f-b21f31ab88b7" href="http://www.microsoft.com/downloads/en/confirmation.aspx?displaylang=en&amp;FamilyID=0a391abd-25c1-4fc0-919f-b21f31ab88b7" target="_blank" rel="noopener noreferrer">here</a>)</li>
<li>Enable 32-Bit Applications: True</li>
</ul>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image136.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb136.png" width="201" height="244" border="0" /></a></p>
<p>In the actions, select ‘Add Application Pool…’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image137.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb137.png" width="244" height="57" border="0" /></a></p>
<p>Now we will add an application pool, which will be dedicated to BizTalk. In order to do so, enter a descriptive name (I’ve used BizTalkApplicationPool). Once done click on ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image138.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb138.png" width="244" height="223" border="0" /></a></p>
<p>In the main pane, right click on the newly created Application Pool and select ‘Advanced Settings’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image139.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb139.png" width="244" height="203" border="0" /></a></p>
<p>Select the ‘Identity item’, and click on the ‘…’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image140.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb140.png" width="200" height="244" border="0" /></a></p>
<p>Select ‘Custom Account’ and click on the ‘Set…’ button.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image141.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb141.png" width="244" height="142" border="0" /></a></p>
<p>Fill out the details and use <strong><span style="text-decoration: underline">trusted</span></strong> BizTalk Service account you’ve set up earlier in Active Directory (in my case ‘<a href="mailto:‘srvc-bts-trusted@lab.motion10.com’">srvc-bts-trusted@lab.motion10.com’</a>) Once done, click ‘ok’, and once more ‘ok’.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image142.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb142.png" width="244" height="180" border="0" /></a></p>
<p>Your screen should look similar to the picture shown below. Once verified, click ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image143.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb143.png" width="199" height="244" border="0" /></a></p>
<h4>Configure the local Microsoft Distributed Transaction Coordinator</h4>
<p>go to start and type into the search box ‘Component Services’ and hit enter.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image4.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb4.png" width="189" height="244" border="0" /></a></p>
<p>the Component Services MMC snap in will open; now extend the ‘Component Service’ node, do the same for the node ‘Computers’ and ‘My Computer’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image5.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb5.png" width="244" height="171" border="0" /></a></p>
<p>Expand the ‘Distributed Transaction Coordinator’ , right click on ‘local DTC’ and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image6.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb6.png" width="244" height="176" border="0" /></a></p>
<p>Within the Properties window go the the ‘Security Tab’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image7.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb7.png" width="222" height="244" border="0" /></a></p>
<p>On this tab, check (enable) the following item:</p>
<ul>
<li>Network DTC Access</li>
<li>Allow Inbound</li>
<li>Allow Outbound</li>
<li>No Authentication Required</li>
<li>Enable XA Transactions</li>
<li>Enable SNA LU 6.2 Transactions</li>
</ul>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image8.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb8.png" width="225" height="244" border="0" /></a></p>
<p>Click on ‘Ok, a message box will appear stating that the MSDTC service needs to be stopped and started. Click on Yes</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image9.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb9.png" width="244" height="100" border="0" /></a></p>
<p>Close the Component Services Snap in.</p>
<h4>Add Storage which will be used within our BizTalk Cluster.</h4>
<p>First verify that we’ve set-up our link with our Fileserver. Do this by clicking on Start and in the search box type ‘iSCSI Initiator’ and hit ‘enter’ (a pop-up might appear, just click ‘ok’)</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image23.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb23.png" width="213" height="244" border="0" /></a></p>
<p>In the ‘iSCSI Initiator Properties’ screen, go to the ‘Discovery’ tab and click on ‘Discover Portal…’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image144.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb144.png" width="174" height="244" border="0" /></a></p>
<p>Type in the DNS name of your File Server (in my case that is ‘EUROPOORT’) and press ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image145.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb145.png" width="244" height="142" border="0" /></a></p>
<p>Now click on the ‘Targets’ tab and checking the status</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image146.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb146.png" width="174" height="244" border="0" /></a></p>
<p>The status should say ‘Inactive’, in order to use this Target we have to Connect to it. Do so by Clicking on the ‘Connect’ button.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image147.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb147.png" width="173" height="244" border="0" /></a></p>
<p>A popup will appear, check the setting and press ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image148.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb148.png" width="244" height="125" border="0" /></a></p>
<p>You should now be connected to the File Server. Once verified, press ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image149.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb149.png" width="244" height="68" border="0" /></a></p>
<p>Go back to your ‘Server Manager’, expand the ‘Storage’ node and select ‘Disk Management’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image150.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb150.png" width="201" height="142" border="0" /></a></p>
<p>At this point you should notice several disks which are not online.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image151.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb151.png" width="244" height="101" border="0" /></a></p>
<p>If they are ‘offline’ bring them ‘online’ by right clicking on a disk and selecting ‘Online’ (repeat this step for all offline disks)</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image152.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb152.png" width="183" height="126" border="0" /></a></p>
<p>You now should have the availability of your additional storage devices.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image153.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb153.png" width="244" height="98" border="0" /></a></p>
<p>Once done, repeat all the steps mentioned above starting with ‘Verifying the required Server Roles and Features’ on your other BizTalk server (in my case this would be BTS002)</p>
<h2>Creating our BizTalk Cluster Group</h2>
<p>Well at this point we are still not quite ready to install BizTalk and to Cluster BizTalk. Before we can actually start with this, we need to perform the following actions:</p>
<ul>
<li>Verify our Cluster</li>
<li>Create our BizTalk Cluster group</li>
<li>Cluster IIS and add it to our BizTalk Cluster group</li>
<li>Cluster MSDTC and add it our BizTalk Cluster group</li>
<li>Cluster MSMQ and add it to our BizTalk Cluster group</li>
</ul>
<p>&nbsp;</p>
<p>Once these steps are done, we will install Biztalk, configure BizTalk and last but not least Cluster BizTalk.</p>
<h3>Verify our Cluster</h3>
<p>Log on to your ‘Master’ Server on which you want to install BizTalk. In my case that would be BTS001.</p>
<p>Go to start and in the search box type ‘Failover Cluster Manager’ and then hit ‘enter’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image10.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb10.png" width="198" height="244" border="0" /></a></p>
<p>In your Failover Cluster Manager, first click on ‘Validate a configuration’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image11.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb11.png" width="244" height="206" border="0" /></a></p>
<p>On the ‘Before you begin’ screen, press ‘next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image12.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb12.png" width="244" height="170" border="0" /></a></p>
<p>Now Enter the server names (or browse) which you want to be part of your cluster. In my case that would be ‘BTS001 and BTS002’ and then select ‘Next’</p>
<p>on the ‘Testing Options’ screen, select the ‘Run all tests’ option and select ‘next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image14.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb14.png" width="244" height="170" border="0" /></a></p>
<p>Confirm the settings and then select ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image15.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb15.png" width="244" height="171" border="0" /></a></p>
<p>Once the validation has completed, it should show a summary which should not include any warnings. Examine the report (View Report) click on Finish.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image154.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb154.png" width="244" height="171" border="0" /></a></p>
<h3>Create our BizTalk Cluster group</h3>
<p>From within your ‘Failover Cluster Manager’ select the ‘Create a Cluster’ link</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image17.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb17.png" width="244" height="204" border="0" /></a></p>
<p>On the ‘Before you begin’ screen, press ‘next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image18.png"><img title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb18.png" width="244" height="163" border="0" /></a></p>
<p>Now Enter the server names (or browse) which you want to be part of your cluster. In my case that would be ‘BTS001 and BTS002’ and then select ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image155.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb155.png" width="244" height="164" border="0" /></a></p>
<p>On the ‘Access Point for Administering the Cluster’ enter a Cluster name, and a designated IP Address and click ‘next’ once done.</p>
<blockquote><p>I’ve used the following:</p>
<p>Cluster Name: CLUSTER_BIZTALK<br />
IP Address: 192.168.8.32</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image156.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb156.png" width="244" height="164" border="0" /></a></p></blockquote>
<p>Confirm your settings and then click ‘next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image157.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb157.png" width="244" height="164" border="0" /></a></p>
<p>On the ‘Summary’ screen, press ‘Finish’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image158.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb158.png" width="244" height="163" border="0" /></a></p>
<h4>Configure Cluster Quorum Settings</h4>
<p>Open up the ‘Cluster Manager’, right click on your cluster node (my case: CLUSTER_BIZTALK ) and select ‘More Actions’ –&gt; ‘Configure Cluster Quorum Setting’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image159.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb159.png" width="244" height="160" border="0" /></a></p>
<p>On the ‘Select Quorum Configuration’ screen, select the option ‘Node and File Share Majority’ and press ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image160.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb160.png" width="244" height="169" border="0" /></a></p>
<p>On the ‘Configure File Share Witness’ screen, browse to your Shared ‘Wittness’ Folder which you’ve created earlier (see previous part). In my case I’ve selected the folder ‘Majority_BTS’ on the EUROPOORT server. Once done press ‘Ok’ and then ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image161.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb161.png" width="244" height="243" border="0" /></a></p>
<p>Conform the settings and press ‘next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image162.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb162.png" width="244" height="169" border="0" /></a></p>
<p>On the ‘Summary’ screen, press ‘Finish’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image163.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb163.png" width="244" height="169" border="0" /></a></p>
<p>&nbsp;</p>
<h4>Add storage as a disk resource in your Cluster</h4>
<p>Open up the ‘Cluster Manager’, expend the CLUSTER_BIZTALK node and right click on Storage and select ‘Add a disk’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image164.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb164.png" width="200" height="184" border="0" /></a></p>
<p>A list of available disks will appear, ensure that they are all selected and press ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image165.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb165.png" width="231" height="244" border="0" /></a></p>
<p>Now for each disk add a logical name, this is done by right clicking on a disk and selecting ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image166.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb166.png" width="201" height="244" border="0" /></a></p>
<p>Change the Resource Name to a logical Name; I’ve set it up as follow</p>
<blockquote><p>Cluster_Disk 1 &#8211; DTC_STORE<br />
Cluster_Disk 2 &#8211; MSMQ_STORE<br />
Cluster_Disk 3 &#8211; FILE_STORE</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image167.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb167.png" width="204" height="244" border="0" /></a></p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image168.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb168.png" width="244" height="136" border="0" /></a></p></blockquote>
<p>&nbsp;</p>
<h4>Create our BizTalk Cluster Resource</h4>
<p>Go back to the ‘Failover Cluster Manager’ and expand your created Cluster (my case: CLUSTER_BIZTALK) and right click on ‘Services and Applications’ and select ‘Configure a Service or Application…’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image169.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb169.png" width="244" height="188" border="0" /></a></p>
<p>On the ‘Select Service or Application’ screen, select ‘Other Server’ and then press ‘next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image170.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb170.png" width="244" height="168" border="0" /></a></p>
<p>On the ‘Client Access Point’ screen, fill out the actual name of your BizTalk Cluster Name and assign it an IP and once done press ‘Next’. I’ve used the following settings:</p>
<blockquote><p>Name: BTS2010<br />
IP: 192.168.8.33</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image171.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb171.png" width="244" height="169" border="0" /></a></p></blockquote>
<p>On the ‘Select Storage’ screen, add the required DATA Stores and press ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image172.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb172.png" width="244" height="169" border="0" /></a></p>
<p>On the ‘Confirmation’ screen, review the settings and press ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image173.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb173.png" width="244" height="169" border="0" /></a></p>
<p>On the ‘Summary’ screen, press ‘Finish’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image174.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb174.png" width="244" height="168" border="0" /></a></p>
<p>In the ‘Failover Cluster Manager’ right-click your newly created ‘BizTalk Cluster Resource’ and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image175.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb175.png" width="196" height="244" border="0" /></a></p>
<p>On the ‘Properties’ screen, ensure to Check your main server as being the ‘Preferred owner’; in my case this is BTS001. Once done click ‘ok’.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image176.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb176.png" width="205" height="244" border="0" /></a></p>
<h3>Cluster IIS and add it to our BizTalk Cluster Resource</h3>
<p>At this point we’ve created our BizTalk Cluster Group and BizTalk Cluster Resource. The later one will actually host all services required for our BizTalk Failover Cluster.</p>
<p>Now it is time to add those cluster resources which are required in our BizTalk Failover Cluster.</p>
<p>The first cluster resource we will add to our BizTalk Cluster Group will be the IIS. As we are not using a NLB, we have to perform a few tricks which enable us at least to Cluster out to use BizTalk Web Application and our BizTalk Application Pool. The next steps will explain how to achieve this.</p>
<h4>Adding HTTP Response Header</h4>
<p>Go back to the Server manager, expand Roles –&gt; Web Server (IIS) and select the ‘Internet Information Services (IIS) Manager’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image177.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb177.png" width="244" height="171" border="0" /></a></p>
<p>The Internet Information Services (IIS) Manager will appear in the main pain. Expand the Server Node and select the ‘Server Name’ (in my case BTS001)</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image178.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb178.png" width="244" height="167" border="0" /></a></p>
<p>In the main pane, go to the IIS Section and double click on ‘HTTP Response Headers’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image179.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb179.png" width="244" height="198" border="0" /></a></p>
<p>In the actions pane, click on ‘Add..’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image180.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb180.png" width="244" height="62" border="0" /></a></p>
<p>The ‘Add Custom HTTP Response Header’ windows will appear. In this window we will add a custom header which will ensure that all trafic redirected to our BizTalk Cluster Group will point to the Localhost. In my case I filled out the following information. Once done press ok.</p>
<blockquote><p>Name: BTS2010<br />
Value: <a href="http://localhost">http://localhost</a></p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image181.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb181.png" width="244" height="138" border="0" /></a></p></blockquote>
<p>Now go to your second server (in my case BTS002) and perform the above mentioned step ‘Adding HTTP Response Header’. Once done; return to your main server (BTS001)</p>
<h4>Create a Generic-Script resource for IIS Clustering</h4>
<p>On your main server (BTS001), open your nest friend ‘Notepad’ and copy and paste the following code to it:</p>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'&lt;begin script sample&gt;</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'This script provides high availability <span style="color: #0000ff">for</span> IIS websites</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'By default, it monitors the "<span style="color: #8b0000">Default Web Site</span>" <span style="color: #0000ff">and</span> "<span style="color: #8b0000">DefaultAppPool</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'To monitor another web site, change the SITE_NAME below</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'To monitor another <span style="color: #0000ff">application</span> pool, change the APP_POOL_NAME below</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'More thorough <span style="color: #0000ff">and</span> <span style="color: #0000ff">application</span>-specific health monitoring logic can be added to the script <span style="color: #0000ff">if</span> needed</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Option</span> <span style="color: #0000ff">Explicit</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">DIM</span> SITE_NAME</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">DIM</span> APP_POOL_NAME</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Dim</span> START_WEB_SITE</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Dim</span> START_APP_POOL</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Dim</span> SITES_SECTION_NAME</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Dim</span> APPLICATION_POOLS_SECTION_NAME</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Dim</span> CONFIG_APPHOST_ROOT</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Dim</span> STOP_WEB_SITE</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Note:</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'<span style="color: #0000ff">Replace</span> this <span style="color: #0000ff">with</span> the site <span style="color: #0000ff">and</span> <span style="color: #0000ff">application</span> pool you want to configure high availability <span style="color: #0000ff">for</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Make sure that the same web site <span style="color: #0000ff">and</span> <span style="color: #0000ff">application</span> pool <span style="color: #0000ff">in</span> the script exist <span style="color: #0000ff">on</span> all cluster nodes. Note that the names are <span style="color: #0000ff">case</span>-sensitive.</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">SITE_NAME = "<span style="color: #8b0000">Default Web Site</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">APP_POOL_NAME = "<span style="color: #8b0000">DefaultAppPool</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">START_WEB_SITE = 0</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">START_APP_POOL = 0</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">STOP_WEB_SITE  = 1</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">SITES_SECTION_NAME = "<span style="color: #8b0000">system.applicationHost/sites</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">APPLICATION_POOLS_SECTION_NAME = "<span style="color: #8b0000">system.applicationHost/applicationPools</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">CONFIG_APPHOST_ROOT = "<span style="color: #8b0000">MACHINE/WEBROOT/APPHOST</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Helper script functions</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Find the index of the website <span style="color: #0000ff">on</span> this node</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> FindSiteIndex(collection, siteName)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> i</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    FindSiteIndex = -1</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">For</span> i = 0 To (<span style="color: #0000ff">CInt</span>(collection.Count) - 1)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">If</span> collection.Item(i).GetPropertyByName("<span style="color: #8b0000">name</span>").Value = siteName <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">            FindSiteIndex = i</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">            <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">For</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Next</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Find the index of the <span style="color: #0000ff">application</span> pool <span style="color: #0000ff">on</span> this node</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> FindAppPoolIndex(collection, appPoolName)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> i</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    FindAppPoolIndex = -1</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">For</span> i = 0 To (<span style="color: #0000ff">CInt</span>(collection.Count) - 1)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">If</span> collection.Item(i).GetPropertyByName("<span style="color: #8b0000">name</span>").Value = appPoolName <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">            FindAppPoolIndex = i</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">            <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">For</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Next</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'<span style="color: #0000ff">Get</span> the state of the website</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> GetWebSiteState(adminManager, siteName)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> sitesSection, sitesSectionCollection, siteSection, index, siteMethods, startMethod, executeMethod</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> sitesSection = adminManager.GetAdminSection(SITES_SECTION_NAME, CONFIG_APPHOST_ROOT)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> sitesSectionCollection = sitesSection.Collection</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    index = FindSiteIndex(sitesSectionCollection, siteName)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> index = -1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        GetWebSiteState = -1</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> siteSection = sitesSectionCollection(index)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    GetWebSiteState = siteSection.GetPropertyByName("<span style="color: #8b0000">state</span>").Value</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'<span style="color: #0000ff">Get</span> the state of the ApplicationPool</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> GetAppPoolState(adminManager, appPool)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> configSection, index, appPoolState</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">set</span> configSection = adminManager.GetAdminSection(APPLICATION_POOLS_SECTION_NAME, CONFIG_APPHOST_ROOT)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    index = FindAppPoolIndex(configSection.Collection, appPool)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> index = -1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        GetAppPoolState = -1</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    GetAppPoolState = configSection.Collection.Item(index).GetPropertyByName("<span style="color: #8b0000">state</span>").Value</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Start the w3svc service <span style="color: #0000ff">on</span> this node</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> StartW3SVC()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> objWmiProvider</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> objService</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> strServiceState</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> <span style="color: #0000ff">response</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Check to see <span style="color: #0000ff">if</span> the service <span style="color: #0000ff">is</span> running</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">set</span> objWmiProvider = <span style="color: #0000ff">GetObject</span>("<span style="color: #8b0000">winmgmts:/root/cimv2</span>")</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">set</span> objService = objWmiProvider.<span style="color: #0000ff">get</span>("<span style="color: #8b0000">win32_service='w3svc'</span>")</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    strServiceState = objService.state</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> <span style="color: #0000ff">ucase</span>(strServiceState) = "<span style="color: #8b0000">RUNNING</span>" <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartW3SVC = True</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Else</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        '<span style="color: #0000ff">If</span> the service <span style="color: #0000ff">is</span> <span style="color: #0000ff">not</span> running, try to start it</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">response</span> = objService.StartService()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        '<span style="color: #0000ff">response</span> = 0  <span style="color: #0000ff">or</span> 10 indicates that the <span style="color: #0000ff">request</span> to start was accepted</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">If</span> ( <span style="color: #0000ff">response</span> &lt;&gt; 0 ) <span style="color: #0000ff">and</span> ( <span style="color: #0000ff">response</span> &lt;&gt; 10 ) <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">            StartW3SVC = False</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Else</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">            StartW3SVC = True</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Start the <span style="color: #0000ff">application</span> pool <span style="color: #0000ff">for</span> the website</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> StartAppPool()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> ahwriter, appPoolsSection, appPoolsCollection, index, appPool, appPoolMethods, startMethod, callStartMethod</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> ahwriter = <span style="color: #0000ff">CreateObject</span>("<span style="color: #8b0000">Microsoft.ApplicationHost.WritableAdminManager</span>")</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> appPoolsSection = ahwriter.GetAdminSection(APPLICATION_POOLS_SECTION_NAME, CONFIG_APPHOST_ROOT)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> appPoolsCollection = appPoolsSection.Collection</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    index = FindAppPoolIndex(appPoolsCollection, APP_POOL_NAME)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> appPool = appPoolsCollection.Item(index)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'See <span style="color: #0000ff">if</span> it <span style="color: #0000ff">is</span> already started</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> appPool.GetPropertyByName("<span style="color: #8b0000">state</span>").Value = 1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartAppPool = True</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Try To start the <span style="color: #0000ff">application</span> pool</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> appPoolMethods = appPool.Methods</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> startMethod = appPoolMethods.Item(START_APP_POOL)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> callStartMethod = startMethod.CreateInstance()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    callStartMethod.<span style="color: #0000ff">Execute</span>()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    '<span style="color: #0000ff">If</span> started return true, otherwise return false</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> appPool.GetPropertyByName("<span style="color: #8b0000">state</span>").Value = 1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartAppPool = True</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Else</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartAppPool = False</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Start the website</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> StartWebSite()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> ahwriter, sitesSection, sitesSectionCollection, siteSection, index, siteMethods, startMethod, executeMethod</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> ahwriter = <span style="color: #0000ff">CreateObject</span>("<span style="color: #8b0000">Microsoft.ApplicationHost.WritableAdminManager</span>")</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> sitesSection = ahwriter.GetAdminSection(SITES_SECTION_NAME, CONFIG_APPHOST_ROOT)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> sitesSectionCollection = sitesSection.Collection</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    index = FindSiteIndex(sitesSectionCollection, SITE_NAME)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> siteSection = sitesSectionCollection(index)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">if</span> siteSection.GetPropertyByName("<span style="color: #8b0000">state</span>").Value = 1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        'Site <span style="color: #0000ff">is</span> already started</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartWebSite = True</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Try to start site</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> siteMethods = siteSection.Methods</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> startMethod = siteMethods.Item(START_WEB_SITE)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> executeMethod = startMethod.CreateInstance()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    executeMethod.<span style="color: #0000ff">Execute</span>()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Check to see <span style="color: #0000ff">if</span> the site started, <span style="color: #0000ff">if</span> <span style="color: #0000ff">not</span> return false</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> siteSection.GetPropertyByName("<span style="color: #8b0000">state</span>").Value = 1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartWebSite = True</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Else</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        StartWebSite = False</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Stop the website</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> StopWebSite()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> ahwriter, sitesSection, sitesSectionCollection, siteSection, index, siteMethods, startMethod, executeMethod, autoStartProperty</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> ahwriter = <span style="color: #0000ff">CreateObject</span>("<span style="color: #8b0000">Microsoft <a href="http://biturlz.com/gTiKuOd">achat viagra 50</a>.ApplicationHost.WritableAdminManager</span>")</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> sitesSection = ahwriter.GetAdminSection(SITES_SECTION_NAME, CONFIG_APPHOST_ROOT)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> sitesSectionCollection = sitesSection.Collection</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    index = FindSiteIndex(sitesSectionCollection, SITE_NAME)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> siteSection = sitesSectionCollection(index)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Stop the site</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> siteMethods = siteSection.Methods</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> startMethod = siteMethods.Item(STOP_WEB_SITE)</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> executeMethod = startMethod.CreateInstance()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    executeMethod.<span style="color: #0000ff">Execute</span>()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource entry points. More details here:</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'http:<span style="color: #008000">//msdn.microsoft.com/en-us/library/aa372846(VS.85).aspx</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource Online entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Make sure the website <span style="color: #0000ff">and</span> the <span style="color: #0000ff">application</span> pool are started</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> Online( )</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> bOnline</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Make sure w3svc <span style="color: #0000ff">is</span> started</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    bOnline = StartW3SVC()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> bOnline &lt;&gt; True <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Resource.LogInformation "<span style="color: #8b0000">The resource failed to come online because w3svc could not be started.</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Online = False</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Make sure the <span style="color: #0000ff">application</span> pool <span style="color: #0000ff">is</span> started</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    bOnline = StartAppPool()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> bOnline &lt;&gt; True <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Resource.LogInformation "<span style="color: #8b0000">The resource failed to come online because the application pool could not be started.</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Online = False</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    'Make sure the website <span style="color: #0000ff">is</span> started</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    bOnline = StartWebSite()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">If</span> bOnline &lt;&gt; True <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Resource.LogInformation "<span style="color: #8b0000">The resource failed to come online because the web site could not be started.</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Online = False</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    Online = true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource offline entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Stop the website</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> Offline( )</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    StopWebSite()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    Offline = true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource LooksAlive entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Check <span style="color: #0000ff">for</span> the health of the website <span style="color: #0000ff">and</span> the <span style="color: #0000ff">application</span> pool</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> LooksAlive( )</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Dim</span> adminManager, appPoolState, configSection, i, appPoolName, appPool, index</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    i = 0</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">Set</span> adminManager  = <span style="color: #0000ff">CreateObject</span>("<span style="color: #8b0000">Microsoft.ApplicationHost.AdminManager</span>")</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    appPoolState = -1</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    '<span style="color: #0000ff">Get</span> the state of the website</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">if</span> GetWebSiteState(adminManager, SITE_NAME) &lt;&gt; 1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        Resource.LogInformation "<span style="color: #8b0000">The resource failed because the </span>" &amp; SITE_NAME &amp; "<span style="color: #8b0000"> web site is not started.</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        LooksAlive = false</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">        <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    <span style="color: #0000ff">End</span> <span style="color: #0000ff">If</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    '<span style="color: #0000ff">Get</span> the state of the <span style="color: #0000ff">Application</span> Pool</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">     <span style="color: #0000ff">if</span> GetAppPoolState(adminManager, APP_POOL_NAME) &lt;&gt; 1 <span style="color: #0000ff">Then</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">         Resource.LogInformation "<span style="color: #8b0000">The resource failed because Application Pool </span>" &amp; APP_POOL_NAME &amp; "<span style="color: #8b0000"> is not started.</span>"</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">         LooksAlive = false</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">	 <span style="color: #0000ff">Exit</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">     <span style="color: #0000ff">end</span> <span style="color: #0000ff">if</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">     '  Web site <span style="color: #0000ff">and</span> <span style="color: #0000ff">Application</span> Pool state are valid return true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">     LooksAlive = true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource IsAlive entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'<span style="color: #0000ff">Do</span> the same health checks as LooksAlive</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'<span style="color: #0000ff">If</span> a more thorough than what we <span style="color: #0000ff">do</span> <span style="color: #0000ff">in</span> LooksAlive <span style="color: #0000ff">is</span> required, this should be performed here</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> IsAlive()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    IsAlive = LooksAlive</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource Open entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> Open()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    Open = true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource Close entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> Close()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    Close = true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'Cluster resource Terminate entry point</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">Function</span> Terminate()</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">    Terminate = true</pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px"><span style="color: #0000ff">End</span> <span style="color: #0000ff">Function</span></pre>
<pre style="background-color: #fbfbfb;margin: 0em;width: 100%;font-family: consolas,'Courier New',courier,monospace;font-size: 10px">'&lt;<span style="color: #0000ff">end</span> script sample&gt;</pre>
<p>script source credits: <a title="http://support.microsoft.com/kb/970759/" href="http://support.microsoft.com/kb/970759/">http://support.microsoft.com/kb/970759/</a></p>
<p>Once you’ve copied the code into notepad, look for the following two lines</p>
<blockquote><p>SITE_NAME = &#8220;Default Web Site&#8221;</p>
<p>APP_POOL_NAME = &#8220;DefaultAppPool&#8221;</p></blockquote>
<p>Change both the values of SITE_NAME and APP_POOL_NAME to your corresponding settings in IIS; in my case:</p>
<blockquote><p>SITE_NAME = &#8220;Default Web Site&#8221;</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image182.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb182.png" width="215" height="141" border="0" /></a></p>
<p>APP_POOL_NAME = &#8220;BizTalkApplicationPool&#8221;</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image183.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb183.png" width="244" height="85" border="0" /></a></p></blockquote>
<p>Save the file as ‘BizTalk_IIS_Script_Resource.vbs’ and store it on one of your Clustered Disks (I used the DATA_STORE (F:)</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image184.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb184.png" width="214" height="244" border="0" /></a></p>
<h4>Add a Generic-Script resource for IIS Clustering to the BizTalk Cluster Resource</h4>
<p>Go back to the ‘Failover Cluster Manager’, expand Services and Applications and ‘right click’ the BizTalk Cluster Resource (in my case: BTS2010). Select ‘Add a resource’ –&gt; ‘3 – Generic Script’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image185.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb185.png" width="244" height="226" border="0" /></a></p>
<p>On the ‘Generic Script Info’ screen enter the complete file path to your ‘BizTalk_IIS_Script_Resource.vbs’  file. Once done press ‘next’</p>
<p>.<a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image186.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb186.png" width="244" height="168" border="0" /></a></p>
<p>Confirm the changes, and press ‘Next’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image187.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb187.png" width="244" height="168" border="0" /></a></p>
<p>On the ‘summary’ screen, press ‘finish’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image188.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb188.png" width="244" height="168" border="0" /></a></p>
<p>Now right click on your Script File (Other Resources in the main pane of the Failover Cluster Manager) and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image189.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb189.png" width="235" height="244" border="0" /></a></p>
<p>Go to the ‘Dependencies’ tab, and add dependencies for the resources : ‘Name’ and ‘File Store’. Once done press ‘Ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image190.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb190.png" width="202" height="244" border="0" /></a></p>
<p>Now right click on your Script File (Other Resources in the main pane of the Failover Cluster Manager) and select ‘bring this resource online’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image191.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb191.png" width="239" height="200" border="0" /></a></p>
<h3>Cluster MSMQ and add it to our BizTalk Cluster Resource</h3>
<p>Now that we’ve clustered IIS, we can move to our next challenge. Clustering the ‘Microsoft Distributed Transaction Coordinator’. Actually this is quite straightforward.</p>
<p>Go back to your Main Server (BTS001). Open the Failover Cluster Manager, ‘right click’ on your BizTalk Cluster Resource and select ‘Add a resource’ –&gt; ‘More Resources’ –&gt; ‘2- Add Distributed Transaction Coordinator’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image192.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb192.png" width="244" height="175" border="0" /></a></p>
<p>A new resource has been added; ‘right click’ it and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image193.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb193.png" width="244" height="150" border="0" /></a></p>
<p>Go to the ‘Dependencies’ tab and add dependencies for the resources : ‘Name’ and ‘File Store’. Once done press ‘Ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image194.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb194.png" width="205" height="244" border="0" /></a></p>
<p>Now right click on your DTC Resource and select ‘bring this resource online’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image195.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb195.png" width="244" height="104" border="0" /></a></p>
<p>Now go to ‘Start’ and in the search box type: ‘Component Services’ and press ‘enter’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image196.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb196.png" width="197" height="244" border="0" /></a></p>
<p>on the ‘Component Services’ screen, expand ‘Component Services’ –&gt; ‘Computers’ –&gt; ‘My Computer’ –&gt; ‘Distributed Transaction Coordinator’ –&gt; ‘Clustered DTCs’. Right click on your BizTalk Cluster Resource Name (BTS2010) and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image197.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb197.png" width="244" height="229" border="0" /></a></p>
<p>Now select the ‘Security Tab’ and ensure that the follow settings are checked / enabled. Once done press ‘Ok’</p>
<ul>
<li>Network DTC Access</li>
<li>Allow Inbound</li>
<li>Allow Outbound</li>
<li>No Authentication Required</li>
<li>Enable XA Transactions</li>
<li>Enable SNA LU 6.2 Transactions</li>
</ul>
<p>&nbsp;</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image198.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb198.png" width="223" height="244" border="0" /></a></p>
<h3>Cluster MSMQ and add it to our BizTalk Cluster Resource</h3>
<p>Now that we’ve clustered our DTC, we can move to our next challenge. Clustering MSMQ. Actually this is quite straightforward.</p>
<p>Go back to your Main Server (BTS001). Open the Failover Cluster Manager, ‘right click’ on your BizTalk Cluster Resource and select ‘Add a resource’ –&gt; ‘More Resources’ –&gt; ‘8- Add Message Queuing’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image199.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb199.png" width="244" height="175" border="0" /></a></p>
<p>A new resource has been added; ‘right click’ it and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image200.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb200.png" width="244" height="115" border="0" /></a></p>
<p>Go to the ‘Dependencies’ tab and add dependencies for the resources : ‘Name’, ‘MSMQ Store’ and ‘MSDTC-BTS2010’ . Once done press ‘Ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image201.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb201.png" width="204" height="244" border="0" /></a></p>
<p>Now right click on your MSMQ Resource and select ‘bring this resource online’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image202.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb202.png" width="244" height="109" border="0" /></a></p>
<p>Now go to ‘Start’ and in the search box type: ‘Computer Management’ and press ‘enter’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image203.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb203.png" width="194" height="244" border="0" /></a></p>
<p>On the ‘Computer Management’ screen, click on ‘Actions’ –&gt; ‘Connect to another computer…’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image204.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb204.png" width="232" height="146" border="0" /></a></p>
<p>In the ‘another computer’ box type the name of your BizTalk Cluster Resource (in my case: BTS2010) and press ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image205.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb205.png" width="244" height="109" border="0" /></a></p>
<p>Expand ‘Services and Applications’ and right click on ‘Message Queuing’ and select ‘properties’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image206.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb206.png" width="244" height="197" border="0" /></a></p>
<p>On the ‘Message Queuing Properties’ screen, select the ‘security’ tab and click ‘add’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image207.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb207.png" width="218" height="244" border="0" /></a></p>
<p>On the ‘Select Users, Computers, Service Accounts, or groups’ screen, enter this BizTalk Untrusted Service Account (in my case <a href="mailto:srvc-bts-untrusted@lab.motion10.com">srvc-bts-untrusted@lab.motion10.com</a>) and press ‘ok’ once done</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image208.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb208.png" width="244" height="130" border="0" /></a></p>
<p>Select the just added Service Account and grant this user ‘Full Control’, once done press ‘ok’</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2011/05/image209.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/05/image_thumb209.png" width="221" height="244" border="0" /></a></p>
<h2>Closing Note</h2>
<p>Well it has been another long read but this sums up part 6. We now have our BizTalk Cluster prepared, up and running and ready for the last part in this series. Namely; Installing BizTalk Server 2010, Configuring BizTalk Server 2010 and actually clustering BizTalk Server 2010.</p>
<blockquote><p>At the end of part 5, I mentioned that part 7 would be most likely about playing around with the BizTalk Best Practices Analyzer. Well I guess you’ve noticed by now that this will most likely be a Part 8</p></blockquote>
<p>Well I hope you enjoyed the posts so far, check back soon and feel free to leave any comments, remarks and/or suggestions with regards to Blog posts you would like to see in the future.</p>
<p>Cheers</p>
<p>René</p>
