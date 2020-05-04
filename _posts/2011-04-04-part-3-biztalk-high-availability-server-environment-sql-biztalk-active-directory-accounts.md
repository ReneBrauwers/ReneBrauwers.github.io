---
ID: 7
post_title: 'Part 3: BizTalk High Availability Server Environment &#8211; SQL &#038; BizTalk Active Directory Accounts'
post_name: >
  part-3-biztalk-high-availability-server-environment-sql-biztalk-active-directory-accounts
author: Rene Brauwers
post_date: 2011-04-04 12:33:21
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/04/04/part-3-biztalk-high-availability-server-environment-sql-biztalk-active-directory-accounts/
published: true
tags:
  - Active Directory
  - BizTalk
  - BizTalk Service Accounts
  - BizTalk User Accounts
  - Installation how to
  - SQL Service Accounts
categories:
  - Active Directory
  - BizTalk
---
In our previous part we’ve installed our Domain Controller and not to say the least one of the most vital servers within our to set up Server Environment.

This post will mainly focus on setting up the Active Directory Accounts which will be used throughout the other upcoming parts.

So let’s get started.
<h2>Planning security groups, user accounts and service accounts</h2>
Like every installation and configuration it is essential to have an overview of the things you would like to accomplish before implementing them, well the same rules apply here; therefore below an overview of the required Security groups, user accounts and service accounts.
<h3>Security Groups</h3>
<ul>
<ul>
	<li>BizTalk Application Users</li>
</ul>
	<li>BizTalk Isolated Host Users</li>
	<li>BizTalk Server Administrators</li>
	<li>BizTalk Server B2B Operators</li>
	<li>BizTalk Server Operators</li>
	<li>BizTalk Bam Portal Users</li>
	<li>SSO Administrators</li>
	<li>SSO Affiliate Administrators</li>
	<li>IIS_IUSRS</li>
</ul>
&nbsp;
<h3>Service Accounts</h3>
<ul>
	<li>srvc-bts-trusted<sub>
<em>[Service account used to run BizTalk Isolated host instance (HTTP/SOAP)]</em></sub></li>
	<li>srvc-bts-untrusted
<sub><em>[Service account used to run BizTalk In-Process host instance which access In-Process BizTalk host instance (BTNTSVC)]</em></sub></li>
	<li>srvc-bts-sso<em>
<sub>[Service account used to run Enterprise Single Sign-On Service which accesses the SSO database]</sub></em></li>
	<li>srvc-bts-rule-engine
<sub><em>[Service account used to run Rule Engine Update Service which receives notifications to deployment/undeployment policies from the Rule engine database]</em></sub></li>
	<li>srvc-bts-bam-ns
<sub><em>[Service account used to run BAM Notification Services which accesses the BAM databases]</em></sub></li>
	<li>srvc-bts-bam-ap
<sub><em>[Application pool account for BAMAppPool which hosts BAM Portal Web site]</em></sub></li>
	<li>srvc-sql-agent</li>
	<li>srvc-sql-engine</li>
	<li>srvc-sql-analysis</li>
	<li>srvc-sql-reporting</li>
	<li>srvc-sql-integration</li>
</ul>
&nbsp;
<h3>User Accounts</h3>
<ul>
	<li>usr-bts-install</li>
	<li>usr-bts-bam</li>
	<li>usr-bts-admin</li>
	<li>usr-bts-operator</li>
	<li>usr-bts-b2b-operator</li>
	<li>usr-bts-sso-admin</li>
	<li>usr-bts-sso-affiliate</li>
</ul>
&nbsp;
<h3>Accounts – Security Group mapping</h3>
<h4>BizTalk Application Users</h4>
Contains service accounts for the BizTalk In-Process host instance in the host that the BizTalk Host Group is designated for.
<h5></h5>
<h6>Accounts</h6>
<ul>
	<li>srvc-bts-untrusted</li>
</ul>
<h4>BizTalk Isolated Host Users</h4>
Contains service accounts for the BizTalk Isolated host instance in the host that the Isolated BizTalk Host Group is designated for.
<h6>Accounts</h6>
<ul>
	<li>srvc-bts-trusted</li>
</ul>
&nbsp;
<h4>BizTalk Server Administrators</h4>
Contains users/groups that need to be able to configure and administer BizTalk Server.
<h6>Accounts</h6>
<ul>
	<li>Domain Admin</li>
	<li>usr-bts-admin</li>
</ul>
&nbsp;
<h4>BizTalk Server B2B Operators</h4>
Contains user/groups that will perform all party management operations
<h6>Accounts</h6>
<ul>
	<li>Domain Admin</li>
	<li>usr-bts-b2b-operator</li>
</ul>
&nbsp;
<h4>BizTalk Server Operators</h4>
Contains user/groups that will monitor solutions.
<h6>Accounts</h6>
<ul>
	<li>Domain Admin</li>
	<li>usr-bts-operator</li>
</ul>
&nbsp;
<h4>BizTalk Bam Portal Users</h4>
Everyone group is used for this role by default.
<h6>Accounts</h6>
<ul>
	<li>Domain Users</li>
</ul>
<h4>SSO Administrators</h4>
Contains service accounts for Enterprise Single Sign-On service.

Contains users/groups that need to be able to configure and administer BizTalk Server and SSO service.

Contains accounts used to run BizTalk Configuration Manager when configuring SSO master secret server.
<h6>Accounts</h6>
<ul>
	<li>Domain Admin</li>
	<li>srvc-bts-sso</li>
	<li>usr-bts-sso-admin</li>
</ul>
&nbsp;
<h4>SSO Affiliate Administrators</h4>
Contains account used for BizTalk Server Administrators
<h6>Accounts</h6>
<ul>
	<li>Domain Admin</li>
	<li>usr-bts-sso-affiliate</li>
</ul>
&nbsp;
<h4>IIS_IUSRS</h4>
This built-in group has access to all the necessary file and system resources so that an account, when added to this group, can seamlessly act as an application pool identity.
<h6>Accounts</h6>
<ul>
	<li>srvc-bts-trusted</li>
	<li>srvc-bts-bam</li>
	<li>srvc-bts-bam-ap</li>
</ul>
&nbsp;
<h2>Adding security groups, user accounts and service accounts</h2>
Now that we have a clear overview of all the required security groups, user and service accounts it´s time to actually add them to our Active Directory.

Fire up your Domain Controller Server, and in your Server Manager open up “Roles” –&gt; “Active Directory Users and Computers” and click on your domain

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image23.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb23.png" width="244" height="123" border="0" /></a>
<h3>Setting up BizTalk Organizational Unit</h3>
Add a new Organizational Unit and name called “BizTalk”, do this by “right clicking” on your domain –&gt; “New” –&gt; “Organizational Unit”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image24.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb24.png" width="244" height="190" border="0" /></a>

Enter the name of the new 'Organizational Unit Object”, ensure to check “Protect container from accidental deletion” and press “OK”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image25.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb25.png" width="244" height="207" border="0" /></a>

Select the just created “Organizational Unit BizTalk” and a new group, do this by “right clicking” your “BizTalk Organizational Unit” –&gt; “New” –&gt; Group

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image26.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb26.png" width="184" height="244" border="0" /></a>

Enter the name of the group, ensure the “Group Scope” is “Global” and the “Group Type” is “Security”. Once done press “OK”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image27.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb27.png" width="244" height="207" border="0" /></a>

Now add the following Security Groups, by repeating the 2 previous mentioned steps:
<ul>
	<li>BizTalk Isolated Host Users</li>
	<li>BizTalk Server Administrators</li>
	<li>BizTalk Server B2B Operators</li>
	<li>BizTalk Server Operators</li>
	<li>BizTalk Bam Portal Users</li>
	<li>SSO Administrators</li>
	<li>SSO Affiliate Administrators</li>
</ul>
&nbsp;

You should end up with the following groups within your “BizTalk Organizational Unit”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image28.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb28.png" width="244" height="171" border="0" /></a>

Now select the just created “Organizational Unit BizTalk” and two new “Organizational Units” named:
<ul>
	<li>Service Accounts</li>
	<li>User Accounts</li>
</ul>
&nbsp;

Do this by “right clicking” your “BizTalk Organizational Unit” –&gt; “New” –&gt; “Group” and filling out the required details (ensure to check “Protect container from accidental deletion”). You should end up with the following 2 new “Organization Units” within the “BizTalk" Organizational Unit”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image29.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb29.png" width="244" height="166" border="0" /></a>

Now select the just created “Organizational Unit Service Accounts” and add the following “Users”
<ul>
	<li>srvc-bts-trusted</li>
	<li>srvc-bts-untrusted</li>
	<li>srvc-bts-sso</li>
	<li>srvc-bts-rule-engine</li>
	<li>srvc-bts-bam</li>
	<li>srvc-bts-bam-ns</li>
	<li>srvc-bts-bam-ap</li>
</ul>
&nbsp;

<em><strong>[Repeat the following steps for each new “User” mentioned above]
</strong></em>Do this by “right clicking” your “Service Accounts Organizational Unit” –&gt; “New” –&gt; “User”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image30.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb30.png" width="244" height="160" border="0" /></a>

Fill out the “First Name”, “Full Name”, “User logon name” and press “next”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image31.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb31.png" width="244" height="205" border="0" /></a>

Assign a “Password”, ensure to uncheck “User must change password at next logon” and ensure to check “User cannot change password” and check “Password never expires”. Once done select “Next” and “Finish”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image32.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb32.png" width="244" height="207" border="0" /></a>

Eventually you should end up with the following users within your “Service Accounts Organizational Unit”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image33.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb33.png" width="244" height="183" border="0" /></a>

Now select the “Organizational Unit User Accounts” and add the following “Users”
<ul>
<ul>
	<li>usr-bts-install</li>
	<li>usr-bts-admin</li>
	<li>usr-bts-operator</li>
	<li>usr-bts-b2b-operator</li>
	<li>usr-bts-sso-admin</li>
	<li>usr-bts-sso-affiliate</li>
</ul>
</ul>
&nbsp;

<em><strong>[Repeat the following steps for each new “User” mentioned above]
</strong></em>Do this by “right clicking” your “User Accounts Organizational Unit” –&gt; “New” –&gt; “User”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image34.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb34.png" width="244" height="160" border="0" /></a>

Fill out the “First Name”, “Full Name”, “User logon name” and press “next”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image35.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb35.png" width="244" height="205" border="0" /></a>

Assign a “Password”, ensure to uncheck “User must change password at next logon” and ensure to check “User cannot change password” and check “Password never expires”. Once done select “Next” and “Finish”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image36.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb36.png" width="244" height="205" border="0" /></a>

Eventually you should end up with the following users within your “User Accounts Organizational Unit”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image37.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb37.png" width="244" height="197" border="0" /></a>
<h3>Setting up Sql Server Organizational Unit</h3>
Now it’s time to set up the SQL Server Organizational Unit; this will be done exactly the same way as mentioned in “Setting up BizTalk Server Organizational Unit”. Below I will summarize what to create.

Add new organizational unit “Sql Server”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image38.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb38.png" width="244" height="118" border="0" /></a>

Within the “SQL Server” organizational unit add new organizational unit named “Service Accounts”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image39.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb39.png" width="244" height="172" border="0" /></a>

Add the following user accounts to the Organizational unit “Service Accounts”
<ul>
<ul>
	<li>srvc-sql-agent</li>
	<li>srvc-sql-engine</li>
	<li>srvc-sql-analysis</li>
	<li>srvc-sql-reporting</li>
	<li>srvc-sql-integration</li>
</ul>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image40.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb40.png" width="244" height="184" border="0" /></a>
<h3>Adding users to designated security groups</h3>
Well we are almost there. Next thing on our list is to assign the created users to the correct Security group. For this you will need to open your previously created “BizTalk Organizational Unit”.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image41.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb41.png" width="244" height="177" border="0" /></a>

Further instructions on how to achieve this, are listed below; sorted by Security Group
<h4>Group: BizTalk Application Users</h4>
Right click on the “Biztalk Application Users group” and select properties, select the “members tab” and then press “Add…”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image42.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb42.png" width="219" height="244" border="0" /></a>

Now select “Advanced…”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image43.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb43.png" width="244" height="131" border="0" /></a>

Ensure that your location is set to your domain, and in the “Common Queries” section add the value “srvc-bts” in the “Name starts with” textbox and select “Find Now”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image44.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb44.png" width="217" height="244" border="0" /></a>

Select the following account “srvc-bts-untrusted” and press “OK”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image45.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb45.png" width="244" height="102" border="0" /></a>

Select “OK”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image46.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb46.png" width="244" height="131" border="0" /></a>

Select “OK”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image47.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb47.png" width="218" height="244" border="0" /></a>
<h4>Group: BizTalk Isolated Host Users</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the “srvc-bts-trusted” account.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image48.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb48.png" width="220" height="244" border="0" /></a>
<h4>Group: BizTalk Server Administrators</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Admins” group</li>
	<li>usr-bts-admin “user account”</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image49.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb49.png" width="220" height="244" border="0" /></a>
<h4>Group: BizTalk Server B2B Operators</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Admins” group</li>
	<li>usr-bts-b2b-operator “user account”</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image50.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb50.png" width="219" height="244" border="0" /></a>
<h4>Group: BizTalk Server Operators</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Admins” group</li>
	<li>usr-bts-operator “user account”</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image51.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb51.png" width="220" height="244" border="0" /></a>
<h4>Group: BizTalk Bam Portal Users</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Users” group</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image52.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb52.png" width="220" height="244" border="0" /></a>
<h4>Group: SSO Administrators</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Admins” group</li>
	<li>srvc-bts-sso  “service account”</li>
	<li>usr-bts-sso-admin “user account”</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image53.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb53.png" width="218" height="244" border="0" /></a>
<h4>Group: SSO Affiliate Administrators</h4>
Repeat the steps as mentioned in “Group: BizTalk Application Users”, but this time you will select the following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Admins” group</li>
	<li>usr-bts-sso-affiliate “user account”</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image54.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb54.png" width="219" height="244" border="0" /></a>
<h4>Group: IIS_IUSRS</h4>
Open op the “Builtin Organizational Unit” and double click on the “IIS_IUSRS” group

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image55.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb55.png" width="244" height="162" border="0" /></a>

Select the “Members” tab and press “Add…”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image56.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb56.png" width="220" height="244" border="0" /></a>

Add following accounts (note; leave the common Queries Filter blank, this way you will see all accounts)
<ul>
	<li>“Domain Admins” group</li>
	<li>“BizTalk Isolated Host Users” group</li>
	<li>srvc-bts-bam “service account”</li>
	<li>srvc-bts-bam-ap “service account”</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image57.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2011/04/image_thumb57.png" width="219" height="244" border="0" /></a>
<h3>Closing Note</h3>
This sums up part 3 SQL &amp; BizTalk Active Directory Accounts, in part 4 we will make the necessary preparations for the SQL en BizTalk failover Cluster set ups, which will include:
<ul>
	<li>Installing the required Roles and Features</li>
	<li>Setting up the File Server and assigning storage to the SQL &amp; BizTalk Clusters.</li>
</ul>
&nbsp;

Until next time

Cheers

René