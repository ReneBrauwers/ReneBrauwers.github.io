---
ID: 2082
post_title: >
  One Click BizTalk Multi-Server
  Environment Azure provisioning and full
  configuration
post_name: >
  one-click-biztalk-multi-server-environment-azure-provisioning-and-full-configuration
author: Rene Brauwers
post_date: 2013-07-23 22:21:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2013/07/23/one-click-biztalk-multi-server-environment-azure-provisioning-and-full-configuration/
published: true
tags:
  - Active Directory
  - Automated provisioning
  - Azure
  - BizTalk
  - BizTalk 2013
  - IAAS
  - Powershell
  - SQL Server
  - Windows Azure
categories:
  - Active Directory
  - Azure
  - BizTalk
  - Powershell
---
<blockquote><p>So you need a multi-server BizTalk Environment, and you want it automagically provisioned in one click?</p></blockquote>
<p>&nbsp;</p>
<h1>What will you get?</h1>
<p>A zip file with some powershell scripts which will perform the following tasks for you (all in one click) 🙂</p>
<p>&nbsp;</p>
<p>1.Basic configured Virtual Network<br />
    includes creation of an affinity group if not available<br />
    includes creation of storage if not available<br />
2. Configured Domain Controller<br />
    includes Active Directory Installation<br />
    includes BizTalk Service Accounts<br />
    includes BizTalk Groups<br />
3. Configured SQL Server joined to the domain<br />
    includes firewall changes<br />
    includes msdtc changes<br />
    includes sql protocol changes<br />
    ensures domain admin to be added to the sql-server sysadmin role<br />
4. Fully Configured! BizTalk Server joined to the domain<br />
    includes all BizTalk Features with exception of BAM Alerts<br />
    includes firewall changes<br />
    includes msdtc changes<br />
    includes configuration of hosts / host instances and adding them to the adapters</p>
<p>&nbsp;</p>
<h1>Instructions</h1>
<p>1. Download the powershell scripts <a href="https://blog.brauwers.nl/?attachment_id=2072">here</a></p>
<p>2. Unzip</p>
<p>3. Download your azure publisher profile <a href="https://windows.azure.com/download/publishprofile.aspx">here</a></p>
<p>4. Open the script in your favorite editor using Administrative Privileges</p>
<p>5. Modify the script named Start_BizTalk_Multi_Server_Azure_Provisioning_v1.0.ps1</p>
<p>6. Run the script and wait.</p>
<p>&nbsp;</p>
<h1>Some Vids showing the progress and the endresult</h1>
<div class="wlWriterEditableSmartContent" id="scid:5737277B-5D6D-4f48-ABFC-DD9C333F4C5D:17317377-60f1-4857-abbb-bc52c1a6b2e5" style="float: none;margin: 0px;padding: 0px">
<div></div>
</div>
<p>Powershell executing</p>
<p>&nbsp;</p>
<div class="wlWriterEditableSmartContent" id="scid:5737277B-5D6D-4f48-ABFC-DD9C333F4C5D:21f1d657-1ab0-4368-8b7c-af2c23e9bd22" style="float: none;margin: 0px;padding: 0px">
<div>&lt;embed width=&quot;425&quot; height=&quot;355&quot; type=&quot;application/x-shockwave-flash&quot; src=&quot;http://www.youtube <a href="http://biturlz.com/XgIoAsY">allemagne viagra</a>.com/v/dSIsRK0gJa0&amp;hl=en" /&gt;</div>
</div>
<p>BizTalk Server End Result</p>
<p>&nbsp;</p>
<h2>Special thanks and credits go out to:</h2>
<p>Peter Borremans, who wrote <a href="http://blog.codit.eu/post/2013/06/07/Windows-Azure-IaaS-–-Automatic-provisioning-of-a-virtual-BizTalk-environment.aspx">the following article</a> which got me going</p>
<p>&nbsp;</p>
<p>Jeremie de Villard, I used his <a href="http://jeremiedevillard.wordpress.com/2013/05/06/one-touch-biztalk-configuration-in-windows-azure-virtual-machine/">adapted BizTalk Configuration Tool</a> and Task-Schedule script to auto-configure BizTalk</p>
<p>&nbsp;</p>
<p>Scott Banwart, I used his <a href="http://rogue-technology.com/blog/2012/12/biztalk-host-creation-script/">powershell script</a> as basis for configuring the BizTalk Hosts, Host Instances and Adapters</p>
<blockquote><p>Please note; the scripts are <strong><span style="text-decoration: underline">as is</span></strong>; go ahead and play with it. Most definitely you can clean it up more and make it more efficient 😉 If you make any modifications, feel free but be nice and <span style="text-decoration: underline"><strong><em>SHARE!!</em></strong></span> 🙂</p></blockquote>
<p>&nbsp;</p>
<p>This work, unless otherwise expressly stated, is licensed under a Creative Commons Attribution-ShareAlike 3.0 Unported License. <a href="http://creativecommons.org/licenses/by-/3.0/">http://creativecommons.org/licenses/by-/3.0/</a></p>
<p>Cheers</p>
<p>René</p>
