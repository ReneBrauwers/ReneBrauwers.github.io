---
ID: 60
post_title: 'First-Look:Installing &#038; Configuration of BizTalk Server  2010 on Windows 8 Developer Preview'
post_name: >
  first-lookinstalling-configuration-of-biztalk-server-2010-on-windows-8-developer-preview
author: Rene Brauwers
post_date: 2011-09-19 10:52:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/09/19/first-lookinstalling-configuration-of-biztalk-server-2010-on-windows-8-developer-preview/
published: true
tags:
  - BizTalk
  - BizTalk Server 2010
  - First look
  - Installation
  - Issues
  - Windows 8 Developer Preview
categories:
  - BizTalk
---
Last week during the BUILD conference a developer preview of Both Windows 8 and Windows Server 8 was released. Once released I’ve decided to give it a go and install and perform a basic configuration (without BAM / EDI) of BizTalk Server 2010 on Windows 8 (note: not Windows Server 8).

&nbsp;

Below a list of issues I encountered and how to resolve these issues.

&nbsp;

Prerequisites
Obtain the Windows 8 developer preview build
Obtain Microsoft BizTalk Server 2010
Ensure that you have a dedicated server available with SQL Server 2008R2.
Hook up your Windows 8 to machine to your Active Directory (this way you can use your AD BizTalk service accounts etc.)

Environment

&nbsp;

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/09/image2.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/09/image_thumb2.png" width="244" height="116" border="0" /></a>
Encountered Issues while installing
Well to be perfectly honest the only issue I encountered during installation of BizTalk Server 2010 was the fact that I got about 10+ windows update screens which all prompted me to go and install the .Net Framework 3.5.1 Features; well I closed all the windows except for one and let windows update continue.

&nbsp;

&nbsp;

Note: For the BizTalk pre-requisites I simple pointed to the cab file , which I already downloaded previously.

&nbsp;

Encountered Issues while configuring
Configuring BizTalk Server 2010 was a bit more of a challenge although everything up to ‘Configuring the BizTalk Server Runtime’ went off without any problems.

&nbsp;

However once it was time to configure the runtime it gave me a an exception informing me that the server could not communicate with the SSO and that it might have to do with the Distributed Transaction Coordinator; Well this was not the issue, as I had configured it on both servers (on the SQL Server box and on the BizTalk Box).

&nbsp;

So next stop was looking into the windows services and then especially the Enterprise Single Sign On Service; well the service was up and running.  So I stopped and started it and tried to configure the BizTalk Server Runtime once again; however I still got the same error.

&nbsp;

Next stop going back to the windows services and this time

I tried an elevated account on the Enterprise Single Sign On Service; but hey you guessed it; still got the same error.

&nbsp;

So; not giving up I went back to the Enterprise Single Sign On service and put everything back to it’s original state (that is use the dedicated sso service account). Well once I tried to start the service again it suddenly gave me an error indicating that there were some RPC issues… Hmmm, so I went and had a look at that particular service and noticed that it was up and running. Restarting it did not throw any other exceptions and that’s when I noticed that the service ‘RpcLocator Service’ was not running. Bingo! Enabling this service resolved the SSO issue and I was able to further configure BizTalk Server 2010.

&nbsp;

Conclusion
BizTalk 2010 can be installed and configured on the Windows 8 Developer Preview Release; however before installing and configuring ensure that

You’ve configured the Distributed Transaction Coordinator on both the SQL Server Box as well as the Windows 8 Box
Ensure that the ‘RpcLocator Service’ is up and running.

Please note: So far I’ve only installed and configured BizTalk 2010 on Windows 8. I’ve not actually played around with sample applications etc.

&nbsp;

Screenshots:

&nbsp;

BizTalk Administrator ‘Pinned’ to Metro

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/09/image3.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/09/image_thumb3.png" width="244" height="199" border="0" /></a>

&nbsp;

BizTalk Server Configuration

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/09/image4.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/09/image_thumb4.png" width="244" height="199" border="0" /></a>

&nbsp;

BizTalk Administrator Console

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/09/image5.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/09/image_thumb5.png" width="244" height="199" border="0" /></a>

&nbsp;

Cheers

&nbsp;

René