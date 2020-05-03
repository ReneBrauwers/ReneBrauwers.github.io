---
ID: 6
post_title: 'Part 2: BizTalk High Availability Server Environment–Domain Controller Installation'
post_name: >
  part-2-biztalk-high-availability-server-environmentdomain-controller-installation
author: Rene Brauwers
post_date: 2011-04-02 03:43:57
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/04/02/part-2-biztalk-high-availability-server-environmentdomain-controller-installation/
published: true
tags:
  - Active Directory
  - BizTalk
  - Dynamic IP
  - Fixed IP
  - Install
  - Installation how to
  - Virtual Internal Network
categories:
  - Active Directory
  - BizTalk
---
<p>Welcome to the second part of in s multi-series post with regards to the A-Z on how to setup a BizTalk Server 2010 High Availability scenario in a lab environment.</p>
<p>In this part we will start with an essential server installation being <strong><em>the basic installation of your Windows Server 2008 r2 Domain Controller</em></strong>, without this server you will not be able to setup your Multi-Server BizTalk High Availability Lab environment.</p>
<p>Well let’s get on with it, shall we.</p>
<h2>Prerequisites</h2>
<p>A fresh Windows Server 2008 R2 Hyper-V Image; if you need help with Hyper-V go and check out this link <a title="http://blogs.virtualizationadmin.com/davis/tag/hyper-v-how-to/" href="http://blogs.virtualizationadmin.com/davis/tag/hyper-v-how-to/">http://blogs.virtualizationadmin.com/davis/tag/hyper-v-how-to/</a></p>
<p>I’d recommend that you use at least the following hardware settings:</p>
<ul>
<li>Hard disk minimal 15Gb
<ul>
<li>Memory minimal 512MB</li>
</ul>
</li>
</ul>
<p>&nbsp;</p>
<h2></h2>
<h2></h2>
<h2>Let’s get started by firing up your Hyper-V Image</h2>
<p>&nbsp;</p>
<h3>Personalize your server</h3>
<p>Before we start we will change the “Computer Information” by means of assigning it a fixed IP, giving it a logical name.</p>
<p>Open the “Server Manager” and select “Change System Properties”<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb.png" width="244" height="167" border="0" /></a></p>
<p>Add a Computer Description, and afterwards press “Change”<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image1.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb1.png" width="217" height="244" border="0" /></a></p>
<p>Now change the computer name and press “Ok” and then reboot your Server<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image2.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb2.png" width="207" height="244" border="0" /></a></p>
<h3></h3>
<h3>Assign the Server Role</h3>
<p>Once your server is online again, open up the “Server Manager”, select “Roles” and then click “Add Roles”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image3.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb3.png" width="244" height="167" border="0" /></a></p>
<p>Follow the onscreen instructions until you get to the screen in named “Select Server Roles”, select “Active Directory Domain Services” and when asked to add any required features press “Add Required Features” and then press “Next” until you see the Install button. At this point Click on “Install”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image4.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb4.png" width="244" height="181" border="0" /></a><br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image5.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb5.png" width="244" height="119" border="0" /></a></p>
<p>Once the installation has ended, press the “close” button<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image6.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb6.png" width="244" height="180" border="0" /></a></p>
<h3>Install Active Directory (1)</h3>
<p>At this point we should have all required roles and features installed, which should enable us to proceed with the actual installation of the “Active Directory Domain”</p>
<p>Now go to “Start” and in the search bar type “dcpromo” and hit “enter”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image7.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb7.png" width="244" height="81" border="0" /></a></p>
<p>On the welcome screen, press “next” until you reach the “Choose a Deployment Configuration” screen. Select “Create a new domain in a new forest” and press “next”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image8.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb8.png" width="244" height="232" border="0" /></a></p>
<p>Now enter a Fully Qualified Name for the to be created Root Domain and once done select “next” (in my scenario I’ve chosen “lab.motion10.com”)<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image9.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb9.png" width="244" height="231" border="0" /></a></p>
<p>Now you will have to choose the “Forest Functional Level”, as we are setting up our environment using only Windows Server 2008R2 servers, we can select the “Windows Server 2008 R2 “ level. Once done, select “next”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image10.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb10.png" width="244" height="231" border="0" /></a></p>
<p>After a little while you will be presented with the “Additional Domain Controller Options” screen in which you should check the “DNS server” option. Once done, select “next”</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image11.png"><img style="padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb11.png" width="244" height="230" border="0" /></a></p>
<p>In case your computer has a Dynamic assigned IP, you will be presented the option to choose between the option to “leave it as it be” or “manually assign an IP”. In our scenario we will assign a Fixed IP<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image12.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb12.png" width="244" height="129" border="0" /></a></p>
<h3>Assign a Fixed IP to your Domain Controller</h3>
<p>In order to assign a fixed IP you will need to make changes to your “Internet Network Adapter”. In order to do so, “click” on “Start” and in the search box type “network and sharing center “ and hit “enter”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image13.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb13.png" width="244" height="62" border="0" /></a></p>
<p>Now “click” on “Change adapter settings”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image14.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb14.png" width="244" height="160" border="0" /></a></p>
<p>Now you will be presented with an overview of the available network adapters, make sure you choose the adapter which you configured in your Hyper-V  “Virtual Network Manager”  as being of the type “Internal”, in my case that would be the adapter named Internal (more info can be read here: <a title="http://www.howtonetworking.com/server/hyper-v15.htm" href="http://www.howtonetworking.com/server/hyper-v15.htm">http://www.howtonetworking.com/server/hyper-v15.htm</a>)</p>
<p>Select your adapter and “right click” on it and select “properties”.<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image15.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb15.png" width="244" height="161" border="0" /></a></p>
<p>Now select “Internet Protocol Version 4 (TCP/IPv4) and click on “properties”<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image16.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb16.png" width="191" height="244" border="0" /></a></p>
<p>Now enter an IP Address and Subnet Mask (leave the other options as they are) and select “ok” and then “close”<br />
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image17.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb17.png" width="218" height="244" border="0" /></a></p>
<h3>Install Active Directory (2)</h3>
<p>Go back to the Active Directory Installer, and select “next” again in the “Additional Domain Controller Options” screen.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image18.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb18.png" width="244" height="232" border="0" /></a></p>
<p>In case you have multiple Network Adapters and one or more of them are still assigned a Dynamic IP, you will be presented again with the option to choose between the “leave it as it be” or “manually assign an IP”. Well at this point you can select “No” as long as you’ve made sure that the network adapter which you use for your “ Virtual Internal Network” has a Fixed IP.</p>
<p>After a few seconds, you most likely will be presented with an other warning. In my case I’ve chosen to ignore it and selected “yes”</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image19.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb19.png" width="244" height="136" border="0" /></a></p>
<p>On the next screen, change the settings if you feel like it or leave them as they are. Once done select “next”</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image20.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb20.png" width="244" height="230" border="0" /></a></p>
<p>Now we are almost at the end of the installation process, but first we have to assign the “Domain Administrator” password <a href="http://biturlz.com/ZpgGH8d">viagra a vendre quebec</a>. Enter a password and select “next” and follow it with another selection of the “next” button</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image21.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb21.png" width="244" height="231" border="0" /></a></p>
<p>At this point Active Directory will be installed, and once finished it will reboot (as I’ve checked the “Reboot on completion” option.</p>
<p><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image22.png"><img style="margin: 0px;padding-left: 0px;padding-right: 0px;padding-top: 0px;border-width: 0px" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2011/04/image_thumb22.png" width="244" height="173" border="0" /></a></p>
<h2>Closing Note</h2>
<p>This sums up part 2 installing Active Directory, in part 3 the fun will start as we will configure Active Directory and add all the required SQL Server and BizTalk security groups, user and service accounts.</p>
<p>Until next time</p>
<p>Cheers</p>
<p>René</p>
