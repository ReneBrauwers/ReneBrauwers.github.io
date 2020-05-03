---
ID: 161
post_title: 'How to: Microsoft CRM 2011 Integration example (Part 2 &#8211; Get data out of CRM 2011)'
post_name: >
  how-to-microsoft-crm-2011-integration-example-part-2-get-data-out-of-crm-2011
author: Rene Brauwers
post_date: 2012-01-17 18:30:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2012/01/17/how-to-microsoft-crm-2011-integration-example-part-2-get-data-out-of-crm-2011/
published: true
tags:
  - BizTalk
  - 'C#'
  - CRM
  - CRM2011
  - Custom Activity
  - How to
  - WCF
categories:
  - BizTalk
  - 'C#'
---
First things first, at this point in time I assume
<ul>
	<li>you’ve read the <a href="http://blog.brauwers.nl/2012/01/13/how-to-microsoft-crm-2011-integration-example-part-1introduction/" target="_blank" rel="noopener noreferrer">previous post</a></li>
	<li>downloaded and installed the <a href="http://www.microsoft.com/download/en/details.aspx?id=24004" target="_blank" rel="noopener noreferrer">CRM2011 SDK</a></li>
	<li>have a working CRM2011 environment to your proposal.</li>
	<li>you have an account for CRM2011 with sufficient rights (I’d recommend System Administrator)</li>
	<li>have visual studio 2010 installed.</li>
	<li>downloaded and extract my visual studio <a href="http://blog.brauwers.nl/?attachment_id=1292" target="_blank" rel="noopener noreferrer">example solution</a></li>
</ul>
&nbsp;

So you’ve met all the requirements mentioned above? Good; let’s get started.

&nbsp;
<blockquote>Note: all code should be used for Demo/Test purposes only! I did not intent it to be Production Grade. So, if you decide to use it, don’t use it for Production Purposes!</blockquote>
&nbsp;
<h2>Building your Custom Workflow Activity for CRM2011</h2>
Once you’ve downloaded and extracted my visual studio <a href="http://blog.brauwers.nl/?attachment_id=1292" target="_blank" rel="noopener noreferrer">example solution</a>, it is time to open it and fix some issues.

&nbsp;
<h3> </h3>
<h3>Ensure your references are correct</h3>
Go to the Crm2011Entities Project and extend the references folder and remove the following two references

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb.png" width="244" height="212" border="0" /></a>

&nbsp;

Once done, we are going to re-add these references; so right click on the References folder of the Crm2011Entities Project and click ‘Add Reference’

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image1.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb1.png" width="244" height="130" border="0" /></a>

&nbsp;

Now click on the ‘browse’ button, in the add reference dialog window

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image2.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb2.png" width="244" height="141" border="0" /></a>

&nbsp;

Now browse to your Windows CRM 2011 SDK BIN folder (in my case: B:InstallMicrosoft CRMSDK CRM 2011bin) and select the following two assemblies:
<ul>
	<li>microsoft.xrm.sdk</li>
	<li>microsoft.xrm.sdk.workflow</li>
</ul>
&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image3.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb3.png" width="244" height="163" border="0" /></a>

&nbsp;

Now repeat the above mentioned steps for the other project

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image4.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb4.png" width="244" height="143" border="0" /></a>

&nbsp;

&nbsp;
<h3>Generate a strongly typed class of all your existing CRM entities.</h3>
Open op the “Crm2011Entities Project”, and notice that it does not contain any files except a readme.txt file.

&nbsp;
<blockquote>
<h4>Q&amp;A session</h4>
<strong><span style="text-decoration: underline;">Me</span></strong>: Well let’s add a file to this project, shall we?

<strong><span style="text-decoration: underline;">You</span></strong>: Hmmm, what file you ask?

<strong><span style="text-decoration: underline;">Me</span></strong>: Well this project will hold a class file which contains all the definitions of your CRM 2011 Entities.

<strong><span style="text-decoration: underline;">You</span></strong>: O no, do I need to create this myself?

<strong><span style="text-decoration: underline;">Me</span></strong>: Well lucky you, there is no need for this.

<strong><span style="text-decoration: underline;">You</span></strong>: So how do I create this file then?

<strong><span style="text-decoration: underline;">Me</span></strong>: Well just follow the steps mentioned below</blockquote>
&nbsp;

So let’s fix this, and open up a command prompt with administrator privileges.

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image5.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb5.png" width="183" height="244" border="0" /></a>

&nbsp;

Now navigate to your CRM 2011 SDK Folder (in my case this would be: B:InstallMicrosoft CRMSDK CRM 2011bin)

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image6.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb6.png" width="244" height="45" border="0" /></a>

&nbsp;
<blockquote>Note: Before you proceed, ensure that you know the url of the  the CRM2011 OrganizationService. Just test it, by simply browsing to this address, and if everything goes right you should see the following page:

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image7.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb7.png" width="244" height="145" border="0" /></a></blockquote>
&nbsp;

&nbsp;

now type in the following command (and replace the values between &lt;….&gt; with your values (see readme.txt)):

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image8.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb8.png" width="244" height="34" border="0" /></a>

&nbsp;

Once completed, you should be presented with the following output:

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image9.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb9.png" width="244" height="52" border="0" /></a>

&nbsp;

The actual file should be written to the location you set and in my case this is: c:Temp

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image10.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb10.png" width="244" height="111" border="0" /></a>

&nbsp;

Once the actual class has been generated, open Visual Studio and right click on the CRM2011Entities project and select  ‘Add Existing Item’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image11.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb11.png" width="244" height="237" border="0" /></a>

Browse to the directory in which the generated class was saved, and select the generated class.

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image12.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb12.png" width="244" height="165" border="0" /></a>

&nbsp;

At this point you should be able to compile the complete solution, so go ahead and do so.
<blockquote>Note: The <a href="http://blog.brauwers.nl/?attachment_id=1292" target="_blank" rel="noopener noreferrer">source code</a>; includes comments which should be self-explanatory</blockquote>
&nbsp;

&nbsp;
<h3>Making the custom workflow activity available in CRM2011.</h3>
So you’ve successfully compiled the solution, so what’s next? Well now it’s time to import this custom created activity in CRM2011.

&nbsp;

In order to do this we will use this nifty application which comes with the CRM2011 SDK. This application is called ‘pluginregistration’ and can be found in the subdirectory tools/pluginregistration of the CRM2011 SDK (in my case the location is

B:InstallMicrosoft CRMSDK CRM 2011toolspluginregistration)

&nbsp;
<blockquote>Note: As you will notice, only the source code of the pluginregistration is available; so you need to compile it; in order to use it.</blockquote>
In the pluginregistration folder, browse to the bin folder and either open the debug or release folder and double click the application PluginRegistration.exe

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image13.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb13.png" width="244" height="130" border="0" /></a>

&nbsp;

You will be presented with the following GUI:

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image14.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb14.png" width="244" height="172" border="0" /></a>

&nbsp;

Now click on “Create New Connection”

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image15.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb15.png" width="244" height="28" border="0" /></a>

&nbsp;

Fill out the connection information, consisting of:
<ul>
	<li>Label:   <em>Friendly name of connection</em>
<ul>
	<li>In my case I named it: CRM2011</li>
</ul>
</li>
	<li>Discovery Url:   <em>Base url of CRM</em>
<ul>
	<li>I used: <a href="http://crm-demo/">http://crm-demo/</a></li>
</ul>
</li>
	<li>User Name: <em>Domain Account with sufficient rights in CRM 2011</em>
<ul>
<ul>
<ul>
<ul>
	<li>In my case I used: LABAdministrator</li>
</ul>
</ul>
</ul>
</ul>
</li>
</ul>
&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image16.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb16.png" width="244" height="132" border="0" /></a>

&nbsp;

Once everything is filled in, press Connect and wait until the discovery is finished. Once finished double click on the organization name (in my case: Motion10 Lab Environent ) and wait for the objects to be loaded.

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image17.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb17.png" width="121" height="244" border="0" /></a>

&nbsp;

Once the objects have been loaded; you should see a screen similar to the one depicted here below:

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image18.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb18.png" width="244" height="171" border="0" /></a>

&nbsp;

Now let’s add our ‘Custom Activity or plugin’. Do this by selecting the ‘Register’ tab and clicking on ‘Register new Assembly’

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image19.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb19.png" width="244" height="126" border="0" /></a>

&nbsp;

The ‘Register New Plugin’ screen will popup and click on the ‘Browse (…)’ button.

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image20.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb20.png" width="217" height="244" border="0" /></a>

&nbsp;

Now browse to the bin folder of the example project “SendCrmEntityToEndPoint“ (the one you compiled earlier) and select the SendCrmEntityToEndPoint.dll file and click on ‘Open’

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image21.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb21.png" width="244" height="150" border="0" /></a>

&nbsp;

Once done, select the option “None“ at step 3 and select the option “Database“ at step 4 and press the button ‘Register Selected Plugins’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image22.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb22.png" width="204" height="244" border="0" /></a>

&nbsp;

Once done you should receive feedback that the plugin was successfully registered.

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image23.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb23.png" width="218" height="244" border="0" /></a>

&nbsp;
<h3>Creating a workflow in CRM2011 which uses the custom activity.</h3>
Now that we have registered our ‘plugin’, it is time to put it to action. In order to do so; we will logon to CRM2011 and create a custom workflow.

&nbsp;

Once you’ve logged on to CRM2011, click on ‘Settings’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image24.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb24.png" width="244" height="147" border="0" /></a>

&nbsp;

Now find the ‘Process Center’ section and click on ‘Processes’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image25.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb25.png" width="244" height="147" border="0" /></a>

&nbsp;

In the main window, click on ‘New’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image26.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb26.png" width="244" height="125" border="0" /></a>

&nbsp;

A dialog window will pop up; fill in the following details and once done press OK:
<ul>
	<li>Process Name: <em>Logical name for this workflow</em>
<ul>
<ul>
<ul>
<ul>
<ul>
	<li><em>I named it: OnAccountProspectStatusExport</em></li>
</ul>
</ul>
</ul>
</ul>
</ul>
</li>
	<li>Entity:<em>  Entity which could trigger this workflow</em>
<ul>
<ul>
<ul>
<ul>
<ul>
	<li><em>I used the Account Entity</em></li>
</ul>
</ul>
</ul>
</ul>
</ul>
</li>
	<li>Category: <em>Select WorkFlow</em></li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image27.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb27.png" width="244" height="206" border="0" /></a>

&nbsp;

A new window will pop up; which is used to define the actual workflow. Use the following settings:
<ul>
	<li>Activate as: <em>Process</em></li>
	<li>Scope: <em>Organization</em></li>
	<li>Start When:
<ul>
<ul>
<ul>
<ul>
<ul>
<ul>
<ul>
	<li>check <em>Record is created</em></li>
	<li><em>check Record fields change and <em>select the field RelationShipType</em></em></li>
</ul>
</ul>
</ul>
</ul>
</ul>
</ul>
</ul>
</li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image28.png"><img style="background-image: none; margin: 5px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb28.png" width="244" height="210" border="0" /></a>
<ul>
	<li>Now add the following step: Check Condition</li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image29.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb29.png" width="158" height="244" border="0" /></a>

&nbsp;
<ul>
	<li>Set the condition to be
<ul>
<ul>
<ul>
<ul>
<ul>
	<li>Select “Account”</li>
	<li>Select Field “RelationshipType”</li>
	<li>Select “Equals”</li>
	<li>Select “Prospect”</li>
</ul>
</ul>
</ul>
</ul>
</ul>
</li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image30.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb30.png" width="244" height="39" border="0" /></a>

&nbsp;
<ul>
	<li>Now add our custom activity the following step: SendCrmEntityToEndPoint</li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image31.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb31.png" width="130" height="244" border="0" /></a>
<ul>
	<li>Configure this activity like this:
<ul>
<ul>
<ul>
<ul>
<ul>
<ul>
	<li>Export to disk:  <em>True</em></li>
	<li>EndPoint location: &lt;Path where entity needs to be written&gt;
<ul>
<ul>
<ul>
<ul>
	<li>In my case I used: c:temp (note this will be written on the c drive on the CRM server!)</li>
</ul>
</ul>
</ul>
</ul>
</li>
</ul>
</ul>
</ul>
</ul>
</ul>
</ul>
</li>
</ul>
&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image32.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb32.png" width="244" height="190" border="0" /></a>

&nbsp;
<ul>
	<li>Now once again add our custom activity the following step: SendCrmEntityToEndPoint</li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image33.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb33.png" width="130" height="244" border="0" /></a>

&nbsp;
<ul>
	<li>Configure this activity like this:</li>
</ul>
<ul>
	<li>Export to disk: <em>False</em></li>
	<li>EndPoint location: Url path to your BizTalk webservice
<ul>
<ul>
<ul>
<ul>
	<li>In my case I used: the endpoint which points to my generated BizTalk WebService (which we will cover in our next blogpost)</li>
</ul>
</ul>
</ul>
</ul>
</li>
</ul>
<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image34.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb34.png" width="244" height="190" border="0" /></a>

&nbsp;

Well at this point your workflow should look similar to this:

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image35.png"><img style="background-image: none; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb35.png" width="244" height="185" border="0" /></a>

&nbsp;

Now click on the ‘Activate’ button

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image36.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb36.png" width="244" height="185" border="0" /></a>

&nbsp;

Confirm the ‘Activation’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image37.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb37.png" width="244" height="179" border="0" /></a>

&nbsp;

Save and close the new workflow

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image38.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb38.png" width="244" height="185" border="0" /></a>

&nbsp;
<h3>Test if everything works</h3>
So now it is time to see if everything works; in order to do so we will create a new Account and if everything went ok; we should see
<ul>
	<li>An Account.xml file somewhere on disk</li>
	<li>An Routing Error in BizTalk (as we send a document which was not recognized by BizTalk)</li>
</ul>
&nbsp;

In CRM2011 click on the ‘Work Place’ button

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image39.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb39.png" width="81" height="244" border="0" /></a>

&nbsp;

Subsequently click on ‘Accounts’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image40.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb40.png" width="147" height="244" border="0" /></a>

And finally add a new ‘Account’, by clicking on ‘NEW’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image41.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb41.png" width="244" height="82" border="0" /></a>

&nbsp;

A new window will pop-up; fill in some basic details

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image42.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb42.png" width="244" height="170" border="0" /></a>

&nbsp;

and don’t forget to set the Relationship type to ‘Prospect’

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image43.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb43.png" width="244" height="101" border="0" /></a>

&nbsp;

Once done click on the ‘Save &amp; Close’ button

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image44.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb44.png" width="205" height="192" border="0" /></a>

&nbsp;

After a few minutes we can check both our output directory and the BizTalk Administrator, and we should notice that in the output directory a file has been written

&nbsp;

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image45.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb45.png" width="244" height="184" border="0" /></a>

and we should have an ‘Routing Failure’ error in BizTalk.

<a href="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image46.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="http://blogbrauwers.azurewebsites.net/wp-content/uploads/2012/01/image_thumb46.png" width="244" height="175" border="0" /></a>

&nbsp;
<h2>Closing Note</h2>
So this sums up our first part in which we build our own Workflow activity, imported it into CRM2011, constructed a workflow and last but not least saw that it worked.

&nbsp;

Hope you enjoyed the read

 Cheers

 René