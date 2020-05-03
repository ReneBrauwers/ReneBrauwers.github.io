---
ID: 162
post_title: 'Okay one more: Adding Network Load balancing to our High Availability Environment'
post_name: >
  okay-one-more-adding-network-load-balancing-to-our-high-availability-environment
author: Rene Brauwers
post_date: 2011-06-22 00:49:51
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/06/22/okay-one-more-adding-network-load-balancing-to-our-high-availability-environment/
published: true
tags:
  - BizTalk 2010
  - Configuration
  - DNS
  - IIS
  - Installation how to
  - Multi Server Installation
  - Network Load Balancing
  - R2
  - Single Sign On
  - Windows Server 2008
categories:
  - Network Load Balancing
---
<p>In the previous parts we set up our BizTalk High Availability environment; using an Active &lt;-&gt; Passive scenario. Well in this post I’ll describe how to extend our High Availability environment with NLB functionality. In order to so so we will need to add one additional server which will function as Network Load Balancer.</p>
<p>Our end result will be a mix between active &lt;-&gt; active and active &lt;&#8211;&gt; passive. Huh? You might think, well okay let me try to explain what I mean 🙂</p>
<p>In our current server environment this would not be possible as we clustered our IIS’s on the BizTalk Servers and added a custom response header to each ‘BizTalk’ Web Site, which would redirected all IIS traffic to our BizTalk Cluster’s <strong><span style="text-decoration: underline"><em>active node</em></span></strong> (see this <a title="Part 6 Biztalk High Availability Server Environment - biztalk 2010 failover cluster creation" href="http://blog.brauwers.nl/2011/05/14/part-6-biztalk-high-availability-server-environmentbiztalk-2010-failover-cluster-creation/" target="_blank" rel="noopener noreferrer">post</a>) thus ensuring that whenever a party sends a message to our ‘webserver’ the actual IIS installed on our <strong><span style="text-decoration: underline"><em>active</em></span></strong> BizTalk Server node would be hit and process the request. However we want to accomplish that we have two dedicated IIS servers which can be utilized such that both can receive requests and send them for processing to BizTalk. So this is where NLB functionality can save the day as a NLB will act as an entry-point for Network Traffic (in our case traffic intended for IIS) and the NLB will then decide to which server to route this traffic (this can be 2 or more servers)</p>
<h3></h3>
<p>So how do we start utilizing this NLB functionality within our server environment; well read on…</p>
<p>&nbsp;</p>
<h2>Server Preparation</h2>
<p>In order to utilize the power of a NLB we will need two additional BizTalk Servers, which in contrast to our initial two BizTalk Servers will not be installed in a Failover Cluster but they will be installed in a NLB Cluster. You might wonder; is it not possible to simply add the NLB functionality to our existing BizTalk Servers, well no; if you try to do so you will receive the following error once you try to setup your NLB Cluster “ Processing update n from &#8220;NLB Manager on XXX&#8221; Cannot proceed because Microsoft Cluster Service is installed”</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb.png" width="244" height="203" border="0" /></a></p>
<p>So this only leaves us with installing two additional BizTalk servers with a NLB cluster and as a nice side effect we will have in addition the availability of a ‘backup’ BizTalk IIS</p>
<blockquote><p>In case our NLB Cluster fails we could always let our original BizTalk Cluster take over the IIS processing; we could do this by simple adding a round-robin DNS entry which would forward requests send to the NLB to the BizTalk Cluster or inform the customers that they should send webrequests to a different address.</p></blockquote>
<h2>Adding the NLB to our environment</h2>
<p>At this point we’ve concluded that we will need two additional BizTalk servers which need to be installed and configured and will solemnly be used for receiving and processing messages received by IIS.</p>
<blockquote><p>We will not configure these servers to receive or send messages other than through out Isolated Host (read IIS), although theoretically this can be easily done; but you might run into problems if you use adapters with a polling mechanism (risk of picking up duplicate messages for processing)</p></blockquote>
<p>Before we start we will need to perform the following actions.</p>
<ul>
<li>Manually Add a DNS entry for our NLB CLuster</li>
<li>Install BizTalk Server 2010 on our new servers</li>
<li>Configure BizTalk Server 2010 on our new servers (they will join our existing BizTalk Group)</li>
<li>Add the NLB Feature on our dedicated Windows Server 2008</li>
</ul>
<p>&nbsp;</p>
<h3></h3>
<h3>Adding a DNS entry for our NLB</h3>
<p>In order for a client to access our BizTalk webservices we need to create an actual endpoint to connect to, and in order to do this we need to add a DNS entry. So let’s get started.</p>
<p>Logon to your Domain Server (in my case SCHIPHOL) and click on ‘Start’ –&gt; ‘Administrative Tools’ and select ‘DNS’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image1.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb1.png" width="214" height="244" border="0" /></a></p>
<p>The DNS Manager window will appear. Now expand your DNS Server node –&gt; Expand ‘Forward Lookup Zones’ and select your Domain Name (my case lab.motion10.com)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image2.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb2.png" width="244" height="145" border="0" /></a></p>
<p>In the main pane ‘right-click’ and select ‘New Host (A or AAAA)’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image3.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb3.png" width="244" height="209" border="0" /></a></p>
<p>Fill out the following details, consisting of Name and an available IP Address; I’ve used the following values</p>
<blockquote><p>Name: BTSWEB<br />
IP Address: 192.168.8.100</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image4.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb4.png" width="241" height="244" border="0" /></a></p></blockquote>
<p>Once done press ‘Add Host’. You should receive a confirmation that the DNS entry was successfully created.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image5.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb5.png" width="244" height="91" border="0" /></a></p>
<h3></h3>
<h3>Install and configure BizTalk Server 2010 on our ‘new’ servers</h3>
<p>At this point I assume you will have prepped two new servers which runs under Windows Server 2008 R2. For future reference purposes I’ve named my servers BTS003 en BTS004</p>
<p><em><strong><span style="text-decoration: underline">Important:</span></strong> Before you proceed; </em></p>
<blockquote><p><em>ensure that your hyper-v configuration with regards to your internal network adapter has the option ‘Allow spoofing of MAC Address enabled.<br />
</em><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image6.png"><br />
<img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb6.png" width="244" height="229" border="0" /></a></p>
<p><em>ensure the required roles and features are installed (Application Server, Web Server, MSMQ) If in doubt see <a href="http://blog.brauwers.nl/2011/04/18/part-4-biztalk-high-availability-server-environment-prepping-our-sql-biztalk-failover-clusters/" target="_blank" rel="noopener noreferrer">part 4</a> section ‘Prepping your BizTalk Servers’</em></p>
<p><em>ensure that you’ve configured your local DTC. If in doubt see <a href="http://blog.brauwers.nl/2011/05/14/part-6-biztalk-high-availability-server-environmentbiztalk-2010-failover-cluster-creation/" target="_blank" rel="noopener noreferrer">part 6</a> section ‘Configure the local Microsoft Distributed Transaction Coordinator’</em></p></blockquote>
<h4>Install BizTalk Server 2010</h4>
<p>Once your servers are prepped you’re ready to install BizTalk Server 2010; if you need a walkthrough with regards to the installation, please read <a href="http://blog.brauwers.nl/2011/05/31/part-7-biztalk-high-availability-server-environment-biztalk-2010-installation-configuration-and-clustering/" target="_blank" rel="noopener noreferrer">part 7</a> section ‘Configuring BizTalk on the Second Node’</p>
<h4></h4>
<h4>Configure BizTalk Server 2010</h4>
<p>At this point you’ve installed BizTalk Server and now it’s time to configure BizTalk such that we can use it. (for detailed instructions with regards to configuring BizTalk, please read <a href="http://blog.brauwers.nl/2011/05/31/part-7-biztalk-high-availability-server-environment-biztalk-2010-installation-configuration-and-clustering/" target="_blank" rel="noopener noreferrer">part 7</a> section ‘Configuring BizTalk Server’</p>
<blockquote><p>Note: The following steps need to be performed on both servers</p></blockquote>
<p>&nbsp;</p>
<p>So logon and  start the BizTalk Server Configuration Tool and on the main configuration screen fill out the default details. Once done press ‘Configure’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image7.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb7.png" width="244" height="225" border="0" /></a></p>
<p>Configure Enterprise SSO, once done press ‘Apply Configuration&#8217;</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image8.png"><img style="padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb8.png" width="244" height="201" border="0" /></a></p>
<p>Before you continue with configuring, perform the following steps:</p>
<blockquote><p>go to start –&gt; All Programs –&gt; Microsoft Enterprise Single Sign-On and select ‘SSO Administration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image9.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb9.png" width="195" height="244" border="0" /></a></p>
<p>Once the ENTSSO window pops up, extend the main node and subsequently expand the servers node. Right click on System and select ‘Properties’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image10.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb10.png" width="244" height="170" border="0" /></a></p>
<p>Now connect to your master SSO Server (in my case BTS2010) and select Apply</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image11.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb11.png" width="244" height="193" border="0" /></a></p></blockquote>
<p>Configure ‘Group’. Once done press ‘Apply Configuration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image12.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb12.png" width="244" height="202" border="0" /></a></p>
<p>Configure ‘BizTalk Runtime’, but only select the option ‘Create Isolated Host and Instances’. Once done press ‘apply configuration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image13.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb13.png" width="244" height="201" border="0" /></a></p>
<p>Skip the configure ‘Business Rule Engine’ and proceed with the configure ‘Bam Tools’. Once done press ‘Apply Configuration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image14.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb14.png" width="244" height="200" border="0" /></a></p>
<p>Configure the ‘BAM Portal’ and once done press ‘Apply Configuration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image15.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb15.png" width="244" height="201" border="0" /></a></p>
<p>Configure ‘BizTalk EDI/AS2 Runtime’ once done press ‘Apply Configuration’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image16.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb16.png" width="244" height="201" border="0" /></a></p>
<p>Once you’ve finished configuring the BizTalk Servers, open up the BizTalk Administrator console and go to ‘PlatForm Settings –&gt; Host Instances’ and you should notice that all host instances in the BizTalk group are visible and as you can see they all either run on BTS001 or BTS002 with exception of the Isolated_Host they are active on all ‘servers’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image17.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb17.png" width="244" height="176" border="0" /></a></p>
<h4>Adding Dedicated Host and Host Instances</h4>
<p>In order to finish up our BizTalk configuration we need to manually add two dedicated Hosts and Host Instances for Sending Back response messages and one for tracking.</p>
<blockquote><p>Note: These Hosts will be made available to all our BizTalk Servers, however they will not be clustered and they will remain <strong><em>inactive </em></strong>on BTS001 and BTS002. Reason for this; is the fact that in case the NLB is not available and our default Failover cluster takes over the IIS responsibility we will not need to reconfigure any ports.  and will not be clustered.</p></blockquote>
<p>We will create two Hosts and Host Instances on both BizTalk Servers (in my case BTS003 and BTS004), these will be called:</p>
<ul>
<li>SendResponse_Host</li>
<li>Tracking_Host2</li>
</ul>
<p>&nbsp;</p>
<blockquote><p>For a detailed instruction how to add Hosts and Host Instances see <a href="http://blog.brauwers.nl/2011/05/31/part-7-biztalk-high-availability-server-environment-biztalk-2010-installation-configuration-and-clustering/" target="_blank" rel="noopener noreferrer">Part 7</a>  .</p></blockquote>
<p>Add the SendResponse_Host</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image18.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb18.png" width="244" height="197" border="0" /></a></p>
<p>Add the SendResponse_Host Instance for BTS003</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image19.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb19.png" width="244" height="196" border="0" /></a></p>
<p>Repeat the above mentioned step for the Host Instances</p>
<ul>
<li>BTS001</li>
<li>BTS002</li>
<li>BTS004</li>
</ul>
<p>&nbsp;</p>
<p>Add the Tracking_Host2</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image20.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb20.png" width="244" height="198" border="0" /></a></p>
<p>Add the Tracking_Host2 Host Instance for BTS003</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image21.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb21.png" width="244" height="196" border="0" /></a></p>
<p>Repeat the above mentioned step for Host Instance BTS004 only. Once done done the Host Instance overview in the BizTalk Administrator should look similar to the following picture:</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image22.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb22.png" width="244" height="187" border="0" /></a></p>
<h4>configuring the appropriate adapters</h4>
<blockquote><p>Note: The following actions need to be performed on all BizTalk Servers. More information on how to configure the adapters see <a href="http://blog.brauwers.nl/2011/05/31/part-7-biztalk-high-availability-server-environment-biztalk-2010-installation-configuration-and-clustering/" target="_blank" rel="noopener noreferrer">Part 7</a></p></blockquote>
<p>Select the HTTP Adapter and add new Send Handler (SendResponse_Host) and make it default</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image23.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb23.png" width="244" height="196" border="0" /></a></p>
<p>Remove the Send Handler: Send_Host from the HTTP Adapter.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image24.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb24.png" width="244" height="61" border="0" /></a></p>
<p>Repeat all these steps for the following adapters:</p>
<ul>
<li>SOAP (note remove the Legacy_Host handler)</li>
<li>WCF-BasicHttp</li>
<li>WCF-WSHttp</li>
</ul>
<p>&nbsp;</p>
<p>Repeat the above mentioned steps on the other BizTalk Servers.</p>
<p>&nbsp;</p>
<h3>Adding and configuring your NLB</h3>
<p>Logon to BTS003 and open up the Server Manager. ‘Right Click’ on Features and select ‘Add Features’.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image25.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb25.png" width="244" height="164" border="0" /></a></p>
<p>Select ‘network load balancing’ and press ‘Next’, followed by ‘Install’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image26.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb26.png" width="244" height="181" border="0" /></a></p>
<p>Once the installation has completed, verify the result and repeat the above steps for server BTS004.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image27.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb27.png" width="244" height="183" border="0" /></a></p>
<h3></h3>
<h3>configure the NLB</h3>
<p>Logon to your NLB Server and go to start and in the search box type ‘Network Load Balancing Manager’ followed by hitting ‘enter’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image28.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb28.png" width="194" height="244" border="0" /></a></p>
<p>Your ‘Network Load Balancing Manager’ screen should now pop up.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image29.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb29.png" width="244" height="197" border="0" /></a></p>
<p>Right Click the main node which says ‘Network Load Balancing Clusters’ and select ‘New Cluster’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image30.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb30.png" width="242" height="118" border="0" /></a></p>
<p>Now for the Host enter the first BizTalk Server DNS name; in my case BTS003 and press ‘Connect’.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image31.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb31.png" width="244" height="229" border="0" /></a></p>
<p>Select the correct IP Address (I’ve chose the Internal Interface, as this is the IP used within my Domain and has a fixed IP address assigned) and press next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image32.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb32.png" width="244" height="229" border="0" /></a></p>
<p>Now we need to select the IP address of our selected BizTalk Server (BTS003). In my case this is 192.168.8.40 and press next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image33.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb33.png" width="244" height="229" border="0" /></a></p>
<p>At this point we need to assign an IP address which will be used by our NLB cluster. Do this by clicking on the ‘Add’ button.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image34.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb34.png" width="244" height="229" border="0" /></a></p>
<p>Let’s use our DNS entry details we created earlier. In my case this is</p>
<p>IPv4 Address: 192.168.8.100<br />
Subnet Mask: 255.255.255.0</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image35.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb35.png" width="244" height="202" border="0" /></a></p>
<p>Once done press ‘OK’. Now ensure that the entry added in the previous step is selected and press ‘Next’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image36.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb36.png" width="244" height="230" border="0" /></a></p>
<p>On the next screen only fill out the Full Internet Name; in my case this would be the DNS name created earlier; thus BTSWEB. Once done press Next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image37.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb37.png" width="244" height="229" border="0" /></a></p>
<p>On the Port Rules screen, leave all settings as they are and press Finish.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image38.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb38.png" width="244" height="231" border="0" /></a></p>
<p>Once done, you’ll end up at the main screen. Right Click on the just created NLB Cluster ‘BTSWEB’ and select ‘Add Host to Cluster’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image39.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb39.png" width="244" height="137" border="0" /></a></p>
<p>Now for the Host enter the second BizTalk Server DNS name; in my case BTS004 and press ‘Connect’.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image40.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb40.png" width="244" height="229" border="0" /></a></p>
<p>Select the correct IP Address (I’ve chose the Internal Interface, as this is the IP used within my Domain and has a fixed IP address assigned) and press next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image41.png">&lt;img style=&quot;background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;&quot; title=&quot;image&quot; alt=&quot;image&quot; src=&quot;https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb41 <a href="http://biturlz.com/TJtQ6TN">viagra pfizer 50 mg</a>.png" width="244" height="229" border="0" /&gt;</a></p>
<p>Now we need to select the IP address of our selected BizTalk Server (BTS004). In my case this is 192.168.8.41 and press next</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image42.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb42.png" width="244" height="229" border="0" /></a></p>
<p>On the Port Rules screen, leave all settings as they are and press Finish.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image43.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb43.png" width="244" height="229" border="0" /></a></p>
<p>Congratulations we’ve just finished configuring our NLB</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image44.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb44.png" width="244" height="138" border="0" /></a></p>
<h2>configuring IIS</h2>
<p>In order to finish our BizTalk configuration we need to configure our IIS on both our servers, The changes we need to implement are:</p>
<ul>
<li>Adding an application pool for BizTalk</li>
<li>Adding an application pool for the BAMPortal</li>
<li>Adding the BamPortal Website</li>
<li>Bumping up the max connections.</li>
</ul>
<p>&nbsp;</p>
<h3>Adding application pools</h3>
<p>&nbsp;</p>
<blockquote><p>Please note: the following actions need to be performed on both BizTalk servers (in my case BTS003 and BTS004)</p></blockquote>
<p>Open up Internet Information Services Manager and select the ‘application pools’ node</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image45.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb45.png" width="244" height="136" border="0" /></a></p>
<p>In the action pane, select ‘Add Application Pool Defaults’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image46.png"><img style="padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb46.png" width="244" height="64" border="0" /></a></p>
<p>ensure that you’ve changed the default application pool settings as depicted below</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image47.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb47.png" width="199" height="244" border="0" /></a></p>
<p>Now let’s add the application pool for out BAM Portal Site; Open up Internet Information Services Manager and right click on ‘Application Pools’ and select ‘Add Application Pool’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image48.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb48.png" width="244" height="137" border="0" /></a></p>
<p>A new window will pop up; in this windows enter the following information as depicted in the screenshot below. Once done click on ‘OK’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0014.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image001[4]" alt="clip_image001[4]" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0014_thumb.png" width="244" height="224" border="0" /></a></p>
<p>Now click on ‘Application Pools’ and ‘right-click’ on the newly added application pool ‘BAMAppPool’ and select ‘Advanced Setting…’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image49.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb49.png" width="244" height="159" border="0" /></a></p>
<p>In the ‘Process Model’ section, select ‘ApplicationPoolIdentity’ and then click on the ‘…’ button</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0016.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image001[6]" alt="clip_image001[6]" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0016_thumb.png" width="199" height="244" border="0" /></a></p>
<p>In the windows which pop’s up; select the option ‘Custom Account’ and click on the set button</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image002.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image002" alt="clip_image002" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image002_thumb.png" width="244" height="142" border="0" /></a></p>
<p>On the ‘Set Credentials’ screen enter the BizTalk BAM service account and enter its password. (In my case the service account is: LABsrvc-bts-bam-ap). Once done press ok</p>
<p>Now repeat the above steps, but this time use the following details</p>
<blockquote><p>Application Pool Name:BizTalkApplicationPool<br />
Application Pool Identity: LABsrvc-bts-trusted</p>
<p>The end result should look similar to the depicted picture below</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image50.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/image_thumb50.png" width="244" height="102" border="0" /></a></p></blockquote>
<p>our current websites to be able to communicate with BizTalk we need to add an Application Pool running with the srvc-bts-trusted service account and last but not least we .</p>
<p>&nbsp;</p>
<h3>Adding the BAM Portal Application</h3>
<blockquote><p>Please note: the following actions need to be performed on both BizTalk servers (in my case BTS003 and BTS004)</p></blockquote>
<p>At this point you should be back on the main Internet Information Services Manager screen; ‘Right Click’ on the ‘Default Web Site’ and select ‘Add Application’</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0018.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image001[8]" alt="clip_image001[8]" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0018_thumb.png" width="180" height="244" border="0" /></a></p>
<p>The add application window pops up, ensure that the following information is filled out:</p>
<p>Alias: BAM<br />
Application Pool: BAMAppPool<br />
Physical Path: C:Program Files (x86)Microsoft BizTalk Server 2010BAMPortal</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0024.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image002[4]" alt="clip_image002[4]" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image0024_thumb.png" width="244" height="171" border="0" /></a></p>
<p>Once done; press ‘Test Settings’, and verify that it was successful.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image003.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="clip_image003" alt="clip_image003" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/clip_image003_thumb.png" width="244" height="168" border="0" /></a></p>
<p>Once done press ‘Close’ and press ‘OK’</p>
<h3></h3>
<h3>Bumping up the max connections.</h3>
<p>In order to optimize the throughput with regards to HTTP based send ports, we need to add a configuration section to both our BTSNTSvc.exe.config and BTSNTSvc64.exe.config files which can be found in the installation directory if BizTalk. (in my case “C:Program Files (x86)Microsoft BizTalk Server 2010”)</p>
<p>So open-up windows explorer and browse to your BizTalk installation directory and upon BTSNTSvc.exe.config by right clicking it and selecting ‘open with…’ –&gt; ‘notepad’ (if notepad is not visible, select ‘Choose default program… and select then notepad’)</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image379.png"><img title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb379.png" width="244" height="100" border="0" /></a></p>
<p>Now add the following code section just above the &lt;/configuration&gt; closing tag and save the file.</p>
<p>&nbsp;</p>
<pre class="csharpcode"><span class="kwrd">&lt;</span><span class="html">system.net</span><span class="kwrd">&gt;</span>
  <span class="kwrd">&lt;</span><span class="html">connectionManagement</span><span class="kwrd">&gt;</span>
     <span class="kwrd">&lt;</span><span class="html">add</span> <span class="attr">address</span><span class="kwrd">="*"</span> <span class="attr">maxconnection</span><span class="kwrd">="25"</span> <span class="kwrd">/&gt;</span>
  <span class="kwrd">&lt;/</span><span class="html">connectionManagement</span><span class="kwrd">&gt;</span>
<span class="kwrd">&lt;/</span><span class="html">system.net</span><span class="kwrd">&gt;</span></pre>
<p>&nbsp;</p>
<p>Once done, your file should look something like this:</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image380.png"><img title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/05/image_thumb380.png" width="244" height="95" border="0" /></a></p>
<p>One of the standard settings which come with a BizTalk Installation is the setting which indicated the maximum connections allowed for HTTP based send adapters. This setting is by default set to</p>
<p>Now repeat this step for the BTSNTSvc64.exe.config</p>
<p>Once done, log on to your other BizTalk server and repeat the above mentioned steps.</p>
<p>&nbsp;</p>
<h2>Closing Note</h2>
<p>Some of you mentioned in the poll that they would love to see a post about adding a NLB to the Server Environment and well the Customer aka Reader is King, so I hope you enjoyed this post. Please note that; this post might change in the near future which might be due to comments and tips I receive from you <img class="wlEmoticon wlEmoticon-smile" alt="Smile" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/06/wlEmoticon-smile.png" /></p>
<p>&nbsp;</p>
<h3>Surprise Challenge (well I have to keep myself busy)</h3>
<p>What would happen if one of the IIS servers in our NLB is malfunctioning and thus no longer able to process and requests and or responses?</p>
<blockquote><p>Well our NLB <strong><em><span style="text-decoration: underline">can’t </span></em></strong>detect that one of our IIS’s is malfunctioning and therefore it could still decide to route ‘web’ requests to the malfunctioning IIS and this would mean that BizTalk would not receive those requests and the user or calling application would be presented with an error.</p></blockquote>
<p>So what ways are there to prevent this, besides adding a Round Robin DNS entry such that our Clustered IIS (on BTS001 and BTS002) would take over this job?</p>
<blockquote><p>Well the ‘nicest’ way would be disabling the ‘failing’ NLB node automagically, and guess what? My next blog post will address this by coding a Windows Service which detects this and disables the ‘malfunctioning’ NLB node</p></blockquote>
<p>So there you have it, I already disclosed my next blog post.  Well once more thanks for reading, and if you have any remarks and or suggestions; please feel free to contact me!</p>
<p>&nbsp;</p>
<p>Cheers</p>
<p>&nbsp;</p>
<p>René</p>
