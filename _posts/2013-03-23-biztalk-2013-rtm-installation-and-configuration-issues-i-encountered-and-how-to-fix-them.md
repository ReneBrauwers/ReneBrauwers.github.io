---
ID: 252
post_title: 'BizTalk 2013 RTM &#8211; Installation and configuration issues I encountered and how to fix them'
post_name: >
  biztalk-2013-rtm-installation-and-configuration-issues-i-encountered-and-how-to-fix-them
author: Rene Brauwers
post_date: 2013-03-23 15:27:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2013/03/23/biztalk-2013-rtm-installation-and-configuration-issues-i-encountered-and-how-to-fix-them/
published: true
tags:
  - BizTalk
  - BizTalk 2013
  - Fix
  - How to
  - Installation
  - Installation how to
  - instructions
  - Issues
  - Resolutions
categories:
  - BizTalk
---
As most of you, I downloaded the final pieces of BizTalk Server 2013 as soon as it was available on MSDN. Once I downloaded it, I decided to setup a clean BizTalk 2013 Development Machine. This post will highlight the issues I encountered during installation and configuration and how I resolved them.

First of a brief highlight of my machine (Single-Server) configuration
* Windows Server 2012
* SQL Server 2013
* One instance for BizTalk MessageBoxDb
* One instance for all other BizTalk databases
* Visual Studio 2012
* BizTalk Server 2013
* Server Memory 8GB
* Disk Space 40Gb

Below the issues I encountered and how I resolved them.
<h2>Issue #1:  Installing BizTalk Server</h2>
Error encountered:   Missing file 'MSVCP100.dll'
<h3>Resolution</h3>
<ol>
	<li><strong></strong>Download and install <a href="http://www.microsoft.com/en-us/download/details.aspx?id=5555">Microsoft Visual C++ 2010 Redistributable Package (both x64 as x86)</a> </li>
</ol>
<h2>Issue #2: BizTalk Configuration – BAM Tools</h2>
Error encountered: Could not install BAM Tools
<h3>Resolution</h3>
<ol>
	<li>install SQL Server 2005 Notification Services
x64 - <a href="http://download.microsoft.com/download/4/4/D/44DBDE61-B385-4FC2-A67D-48053B8F9FAD/SQLServer2005_NS_x64.msi">http://download.microsoft.com/download/4/4/D/44DBDE61-B385-4FC2-A67D-48053B8F9FAD/SQLServer2005_NS_x64.msi</a>
x86 - <a href="http://download.microsoft.com/download/4/4/D/44DBDE61-B385-4FC2-A67D-48053B8F9FAD/SQLServer2005_NS.msi">http://download.microsoft.com/download/4/4/D/44DBDE61-B385-4FC2-A67D-48053B8F9FAD/SQLServer2005_NS.msi</a></li>
	<li>set up database to use <a href="http://www.sqlservercentral.com/blogs/sqlservernotesfromthefield/2012/05/01/how-to-configure-database-mail">database-mail</a> </li>
	<li>configure BAM Alerts</li>
</ol>
<h2>Issue #3: BizTalk Configuration – BAM Portal</h2>
Error encountered: Could not install BAM Portal -&gt; error with regards to "BAM Management Web Service User"
Error thrown: "Attempted to read or write protected memory. This is often an indication that other memory is corrupt."
Actual exception: Log indicated 'Cannot alter the role 'NSSubscriberAdmin', because it does not exist or you do not have permission.'
<h3>Resolution</h3>
<ol>
	<li>Manually Add NSSubscriberAdmin DatabaseRole to BAM Alert Application Database</li>
</ol>
&nbsp;
<h2>END Result</h2>
<a href="https://brauwers-nl.azureedge.net/images/blog/2013/03/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2013/03/image_thumb.png" width="244" height="124" border="0" /></a>

Well this was a quick post, but I hope that it might help you out; if you encounter the same issues

Cheers

René