---
ID: 4012
post_title: >
  Exposing a service bus topic using Azure
  API Management
post_name: >
  exposing-a-service-bus-topic-using-azure-api-management
author: Rene Brauwers
post_date: 2014-05-29 19:17:18
layout: post
link: >
  https://brauwers.azurewebsites.net/2014/05/29/exposing-a-service-bus-topic-using-azure-api-management/
published: true
tags:
  - API Management
  - Azure
  - Azure API Management
  - Azure Service Bus
  - Cloud
  - How to
  - Service Bus
  - Step by step
  - Topics
categories:
  - Azure
---
<h2>Introduction</h2>
<p>Microsoft released a new service to Azure, called API Management. This service was released on May the 12<sup>th</sup> 2014.</p>
<p>Currently the Azure API Management is still in Preview, but already it enables us to easily create an API façade over a diverse set of currently available Azure services like Cloud Services, Mobile Services, Service bus as well as on premise web-services.</p>
<p>Instead of listing all the features and write an extensive introduction about Azure API Management I&#8217;d rather supply you with a few links which contains more information about the Azure API Management service:</p>
<p style="margin-left: 36pt">Microsoft:</p>
<p style="margin-left: 36pt"><a href="http://azure.microsoft.com/en-us/services/api-management/">http://azure.microsoft.com/en-us/services/api-management/</a></p>
<p style="margin-left: 36pt"><a href="http://azure.microsoft.com/en-us/documentation/services/api-management/">http://azure.microsoft.com/en-us/documentation/services/api-management/</a></p>
<p style="margin-left: 36pt">Blogs:</p>
<p style="margin-left: 36pt"><a href="http://trinityordestiny.blogspot.in/2014/05/azure-api-management.html">http://trinityordestiny.blogspot.in/2014/05/azure-api-management.html</a></p>
<p style="margin-left: 36pt"><a href="http://blog.codit.eu/post/2014/05/14/Microsoft-Azure-API-Management-Getting-started.aspx">http://blog.codit.eu/post/2014/05/14/Microsoft-Azure-API-Management-Getting-started.aspx</a></p>
<h2>Some more background</h2>
<p>As most of my day-to-day work involves around the Microsoft Integration space in which I am mainly focusing on BizTalk Server, BizTalk Services and Azure in general, I was looking to a find a scenario in which I, as an Integration person, could and would use Azure API management.</p>
<p>The first thing which popped in to my mind; wouldn&#8217;t it be great to virtualize my existing BizTalk Service Bridges using Azure API management. Well currently this is not possible, as the only authentication and authorization method on a BizTalk Service Bridge is ACS (Access Control Service) and this is not supported in the Azure API Management Service.</p>
<p>Luckily with the last feature release of BizTalk Services included support for Azure Service Bus Topics / queues as a source <span style="font-family: wingdings">J</span> and luckily for me Azure Service Bus supports SAS (Shared Access Signatures) and using such a signature I am able to generate a token and use this token in the HTTP Header – Authorization section of my http request.</p>
<p>Knowing the above, I should be able to define API&#8217;s which virtualize my Service bus Endpoints. Create a product combining the defined API&#8217;s and assign policies to my API operations.</p>
<p>Sounds easy? Doesn&#8217;t it. Well it actually is. So without further ado, let&#8217;s dive into a scenario which involves exposing an Azure Service bus Topic using Azure API Management.</p>
<h2>Getting started</h2>
<p>Before we actually start please note that the sample data (Topic names, user-accounts, topic subscriptions, topic subscription rules etc.) I use for this &#8216;Step by step&#8217; guide is meant for a future blog post 😉 <a href="http://bit.ly/1lD2BYM">extending an article I posted a while back on the TechNet Wiki</a></p>
<p>Other points you should keep in mind are</p>
<ul>
<li>Messages send to a TOPIC <a href="http://msdn.microsoft.com/en-us/library/hh694235.aspx">may not exceed 256Kb in size</a>, if a message is larger you will receive an error message from Service Bus telling you that the message is too large.</li>
<li>Communication to service bus is asynchronous; thus we send a message and all we get back is an HTTP code telling us the status of the submission (200 OK, 210 Accepted etc.) or an error message indicating that something went wrong (401 Access Denied, etc.). So actually our scenario is using the Fire and Forget principle.</li>
<li>You will have to create a SAS Token, which needs to be send in the header of the message in order to authenticate to Service bus.</li>
</ul>
<p>Enough set, let&#8217;s get started.</p>
<p>&nbsp;</p>
<blockquote><p>For the benefit of the reader I&#8217;ve added hyperlinks below such that you can skip those sections involved which you might already know.</p>
<h3>Sections Involved</h3>
<ul>
<li><a href="#Create_a_new_Azure_Account">Create a new Azure Account</a></li>
<li><a href="#Provision_an_Azure_Service_Bus_entity">Provision an Azure Service Bus entity</a></li>
<li><a href="#Create_a_new_Azure_Service_Bus_TOPIC">Create a new Azure Service Bus TOPIC</a></li>
<li><a href="#Add_Subscriptions_and_filters_to_your_newly_created_TOPIC">Add Subscriptions and filters to your newly created TOPIC</a></li>
<li><a href="#Assign_Shared_Access_Policies_to_your_TOPIC">Assign Shared Access Policies to your TOPIC</a></li>
<li><a href="#Generate_a_SAS_Token">Generate a SAS Token</a></li>
<li><a href="#Create_a_new_API_Management_Instance">Create a new API Management Instance</a></li>
<li><a href="#Azure_API_Management_Configuration">Azure API Management Configuration</a></li>
<li><a href="#Create_and_configure_a_new_API">Create and configure a new API</a></li>
<li><a href="#Create_a_new_Product">Create a new Product</a></li>
<li><a href="#Create_a_new_Policy_for_your_defined_API_operations">Create a new Policy for your defined API operations</a></li>
<li><a href="#Setting_up_your_Policy_definition">Setting up your Policy definition</a></li>
<li><a href="#Create_a_Group,_an_User_and_Add_subscribe_the_user_to_a_product">Create a Group, an User and Add subscribe the user to a product</a></li>
<li><a href="#Test_your_API">Test your API</a></li>
<li><a href="#Takeaways">Takeaways</a></li>
<li><a href="#Conclusion">Conclusion</a></li>
</ul>
</blockquote>
<h3><a name="#Create_a_new_Azure_Account">Create a new Azure Account</a></h3>
<p>If you don&#8217;t already have an Azure account, you can sign up for a free trial <a href="http://azure.microsoft.com/en-us/">here</a></p>
<p>&nbsp;</p>
<h3><a name="#Provision_an_Azure_Service_Bus_entity">Provision an Azure Service Bus entity</a></h3>
<p>Once you&#8217;ve signed up for an Azure account, login to the Azure Portal and create a new Azure Service Bus Topic by following the steps listed below.</p>
<ol>
<li>
<div>If you have logged in to the preview portal, click on the &#8217;tile&#8217; Azure Portal. This will redirect you to an alternative portal which allows for a more complete management of your Azure Services.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase1.png" alt="" /></li>
<li>
<div>In the &#8216;traditional&#8217; portal click on Service Bus</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase2.png" alt="" /></li>
<li>
<div>Create a new Namespace for your Service bus Entity</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase3.png" alt="" /></li>
<li>
<div>Enter a name for your Service bus namespace and select the region to which it should be deployed and select the checkmark which starts the provisioning.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase4.png" alt="" width="265" height="186" /></li>
<li>
<div>Once the provisioning has finished, select the Service Bus Entity and click on Connection Information</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase5.png" alt="" /></li>
<li>
<div>A window will appear with Access Connection Information, in this screen copy the ACS Connection String to your clipboard. (We will need this connection string later on) and then click on the checkbox to close to window</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase6.png" alt="" /></li>
</ol>
<p>&nbsp;</p>
<h3><a name="#Create_a_new_Azure_Service_Bus_TOPIC">Create a new Azure Service Bus TOPIC</a></h3>
<p>Now that a new Service Bus Entity has been provisioned, we can proceed with creating a TOPIC within the created Service Bus entity, for this we will use the Service Bus Explorer from Paolo Salvatori. Which you can download <a href="http://code.msdn.microsoft.com/windowsazure/Service-Bus-Explorer-f2abca5a">here</a>. Once you&#8217;ve downloaded this tool, extract it and execute the ServiceBusExplorer.exe file and follow the below mentioned steps.</p>
<ol>
<li>
<div>Press CRTL+N which will open the &#8220;Connect to Service Bus Namespace&#8221; window</div>
</li>
<li>
<div>In the Service Bus Namespaces box, select the following option from the dropdown: &#8220;Enter Connection String&#8221;</div>
</li>
<li>
<div>Copy and paste the ACS Connection String (you copied earlier, see previous section step 6) and once done press &#8220;OK&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase7.png" alt="" width="279" height="475" /></li>
<li>
<div>The Service Bus Explorer should now have made a connection to your Service Bus Entity</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase8.png" alt="" width="280" height="173" /></li>
<li>
<div>In the menu on your left, select the TOPIC node, right click and select &#8220;Create Topic&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase9.png" alt="" width="281" height="246" /></li>
<li>
<div>In the window which now appears enter a TOPIC name &#8220;managementapi_requests&#8221; in the &#8220;Path&#8221; box and leave all other fields blank (we will use the defaults). Once done press the &#8220;Create button&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase10.png" alt="" /></li>
<li>
<div>Your new Topic should now have been created</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase11.png" alt="" /></li>
</ol>
<h3></h3>
<h3><a name="#Add_Subscriptions_and_filters_to_your_newly_created_TOPIC">Add Subscriptions and filters to your newly created TOPIC</a></h3>
<p>Now that we have created a TOPIC it is time to add some subscriptions. The individual subscriptions we will create will contain a filter such that messages which are eventually posted to this TOPIC end up in a subscription based on values set in the HTTP header of the submitted messages. In order to set up some subscriptions follow the below mentioned steps:</p>
<ol>
<li>
<div>Go back to your newly created TOPIC in the Service Bus Explorer</div>
</li>
<li>
<div>Right click on the TOPIC and select &#8220;Create Subscription&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase12.png" alt="" width="360" height="376" /></li>
<li>
<div>The Create Subscription windows will now show in which you should execute the following steps</div>
<ul>
<li><strong>A)</strong> Subscription Name: <strong>BusinessActivityMonitoring</strong></li>
<li><strong>B)</strong> Filter: <strong>MessageAction=&#8217; BusinessActivityMonitoring&#8217;</strong></li>
<li><strong>C)</strong> Click on the Create button</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase13.png" alt="" /></li>
<li>
<div>Now repeat Steps 2 and 3 in order to create the following subscription</div>
<ul>
<li>Subscription Name: <strong>Archive</strong></li>
<li>Filter: <strong> 1 = 1</strong></li>
</ul>
</li>
</ol>
<h3></h3>
<h3><a name="#Assign_Shared_Access_Policies_to_your_TOPIC">Assign Shared Access Policies to your TOPIC</a></h3>
<p>At this point we have set up our Topic and added some subscriptions <a href="http://biturlz.com/revRzgU">le viagra est il efficace</a>. The next step consists of adding a Shared Access Policy to our topic. This policy than allows us to generate a SAS token which later on will be used to authenticate against our newly created Topic. So first things first, let&#8217;s assign a Shared Access Policy first. The next steps will guide you through this.</p>
<ol>
<li>
<div>Login to the <a href="https://manage.windowsazure.com/">Azure Portal</a></div>
</li>
<li>
<div>Go to the Service Bus menu item and select the Service Bus Service you created earlier by clicking on it.</div>
</li>
<li>
<div>Now select TOPICS from the tab menu</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase14.png" alt="" /></li>
<li>
<div>Select the Connection Information Icon on the bottom</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase15.png" alt="" /></li>
<li>
<div>A new window will pop up, in this windows click on the link &#8220;Click here to configure&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase16.png" alt="" /></li>
<li>
<div>Now un the shared access policies:</div>
<ul>
<li><strong>A)</strong> Create a new policy named &#8216;API_Send&#8217;</li>
<li><strong>B)</strong> Assign a Send permission to this policy</li>
<li><strong>C)</strong> Create a new policy named &#8216;API_Receive&#8217;</li>
<li><strong>D)</strong> Assign the Listen permission to this policy</li>
<li><strong>E)</strong> Create a new policy named &#8216;API_Manage&#8217;</li>
<li><strong>F)</strong> Assign the Manage permission to this policy</li>
<li><strong>G)</strong> Click on the SAVE icon on the bottom</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase17.png" alt="" /></li>
<li>
<div>At this point for each policy a Primary and Secondary key should be generated.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase18.png" alt="" /></li>
</ol>
<h3></h3>
<h3><a name="#Generate_a_SAS_Token">Generate a SAS Token</a></h3>
<p>Once we&#8217;ve added the policies to our Topic we can generate a token. In order to generate a token, I&#8217;ve build a small forms application which uses part of the code which was originally <a href="http://code.msdn.microsoft.com/wpapps/Shared-Access-Signature-0a88adf8">published by Santosh Chandwani</a>. Click the following link to start downloading the application &#8220;<a href="http://bit.ly/1htBEW1">Sas Token Generator</a>&#8220;. Using the SAS Token Generator application we will now generate the token signatures.</p>
<ol>
<li>
<div>Download the <a href="http://bit.ly/1htBEW1">Sas Token Generator</a> and start it</div>
</li>
<li>
<div>Fill out the following form data</div>
<ul>
<li><strong>A)</strong> Resource Uri = HTTPs Endpoint to your TOPIC</li>
<li><strong>B) </strong>Policy Name = API_Send</li>
<li><strong>C)</strong> Key = The Primary Key as previously generated</li>
<li><strong>D)</strong> Expiry Date = Select the date you want the Sas token to expire</li>
<li><strong>E) </strong>Click on generate</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase19.png" alt="" /></li>
<li>
<div>After you have clicked GENERATE by Default a file will be created on your desktop containing all generated Sas Tokens, the file is named SAS_tokens.txt. Once saved you will be asked if you want to copy the generated token to your Clipboard. Below 2 images depicting the message-prompt as well as the contents stored in the generated file.Perform step 2 for the other 2 policies as well (API_Listen and API_Manage)</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase20.png" alt="" width="350" height="126" /></p>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase21.png" alt="" width="351" height="172" /></li>
</ol>
<h3></h3>
<h3><a name="#Create_a_new_API_Management_Instance">Create a new API Management Instance</a></h3>
<p>At this point we have set up our Service Bus Topic, Subscriptions and have generated our Sas Tokens we are all set to start exposing the newly created service bus topic using Azure API Management, but before we can start with this we need to create a new API Management Instance. The steps below detail how to do this.</p>
<ol>
<li>
<div>Login to the <a href="https://manage.windowsazure.com/">Azure Portal</a></div>
</li>
<li>
<div>Click on API Management, in the right menu-bar, and click on the link &#8220;Create an API Management Service&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase22.png" alt="" /></li>
<li>
<div>A menu will pop up in which you need to select CREATE once more</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase23.png" alt="" /></li>
<li>
<div>At this point a new window will appear:</div>
<ul>
<li><strong>A)</strong> Url: Fill out an unique name</li>
<li><strong>B)</strong> Pricing Tier: Select Developer</li>
<li><strong>C) </strong>Subscription: Check your subscription Id (Currently it does not show the subscription name, which I expect to be fixed pretty soon)</li>
<li><strong>D)</strong> Region: Select a region close to you</li>
<li><strong>E)</strong> Click on the right arrow</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase24.png" alt="" /></li>
<li>
<div>You now will end up at step 2:</div>
<ul>
<li><strong>A)</strong> Organization Name: Fill out your organization name</li>
<li><strong>B)</strong> Administration E-Mail: Enter your email address</li>
<li><strong>C)</strong> Click on the &#8216;Checkmark&#8217; icon</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase25.png" alt="" /></li>
<li>
<div>In about 15 minutes your API management Service will have been created and you will be able to login.</div>
</li>
</ol>
<h2></h2>
<h2><a name="#Azure_API_Management_Configuration">Azure API Management Configuration</a></h2>
<p>Now that we have provisioned our Azure API management service it is time to create and configure an API which exposes the previously defined Azure Service Bus Topic such that we can send messages to it. The API which we are about to create will expose one operation pointing to the Azure Service Bus Topic and will accept both XML as JSON messages. Later on we will define a policy which will ensure that if a JSON message is received it is converted to XML and that the actual calls to the Service Bus Rest API are properly authenticated using our SAS token created earlier.</p>
<p>So let&#8217;s get started with the creation of our API by Clicking the Manage Icon which should be visible in the menu bar at the bottom of your window.</p>
<p style="margin-left: 36pt"><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase26.png" alt="" /></p>
<p>Once you&#8217;ve clicked the Manage icon, you should automatically be redirected to the API Management portal.</p>
<p style="margin-left: 36pt"><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase27.png" alt="" /></p>
<h3></h3>
<h3><a name="#Create_and_configure_a_new_API">Create and configure a new API</a></h3>
<p>Now that you are in the Azure API Management Administration Portal you can start with creating and configuring a new API, which will virtualize your Service Bus Topic Rest Endpoints. In order to do so follow the following steps.</p>
<ol>
<ol>
<li>
<div>Click on the API&#8217;s menu item on your left</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase28.png" alt="" /></li>
<li>
<div>Click on ADD API</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase29.png" alt="" /></li>
<li>
<div>A new window will appear, fill out the following details</div>
<ul>
<li>
<div><strong>A)</strong> Web API title</div>
<ul>
<li>Public name of the API as it would appear on the developer and admin portals.</li>
</ul>
</li>
<li>
<div><strong>B)</strong> Web Service Uri</div>
<ul>
<li>This should point to your Azure Service Bus Rest Endpoint. Click on this <a href="http://msdn.microsoft.com/en-us/library/hh780786.aspx">link</a> to get more information. The format would be: http{s}://{serviceNamespace}.servicebus.Windows.net/{topic path}/messages</li>
</ul>
</li>
<li>
<div><strong>C) </strong>Web API Uri suffix</div>
<ul>
<li>Last part of the API&#8217;s public URL. This URL will be used by API consumers for sending requests to the web service.</li>
</ul>
</li>
<li>
<div><strong>D) </strong>Once done, press Save</div>
</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase30.png" alt="" /></li>
<li>
<div>Once the API has been created you will end up at the API configuration page</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase31.png" alt="" /></li>
<li>
<div>Now click on the Settings Tab</div>
<ul>
<li><strong>A) </strong>Enter a description</li>
<li><strong>B)</strong> Ensure to set authentication to None (we will use the SAS token later on, to authenticate)</li>
<li><strong>C) </strong>Press Save</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase32.png" alt="" /></li>
<li>
<div>Now click on the Operations Tab, and click on ADD Operation.</div>
<ul>
<li>Note: detailed information on how to configure an operation can be found <a href="http://azure.microsoft.com/en-us/documentation/articles/api-management-howto-add-operations/">here</a></li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase33.png" alt="" /></li>
<li>
<div>A form will appear which will allow you to configure and add an operation to service. By default the Signature menu item is selected, so we start with configuring the signature of our operation.</div>
<ul>
<li><strong>A)</strong> HTTP verb: Choose POST as we will POST messages to our Service Bus Topic</li>
<li><strong>B)</strong> URL Template: We will not use an URL template, so as a default enter a forward-slash &#8221; / &#8220;</li>
<li><strong>C)</strong> Display Name: Enter a name, which will be used to identify the operation</li>
<li><strong>D)</strong> Description: Describe what your operation does</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase34.png" alt="" /></li>
<li>
<div>Now click on the Body item in the menu bar on the left underneath REQUESTS (we will skip caching as we don&#8217;t want to cache anything), and fill out the following fields</div>
<ul>
<li><strong>A) </strong>Description: add a description detailing how the request body should be represented</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase35.png" alt="" /></li>
<li>
<div>Now click on the ADD REPRESENTATION item just underneath the description part and enter Application/XML</div>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase36.png" alt="" width="166" height="84" /></p>
<ol>
<ol>
<li>
<div>Once you&#8217;ve added the representation type, you can add a representation example.</div>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase37.png" alt="" width="457" height="216" /></p>
<ol>
<ol>
<li>
<div>Now once again click on the ADD REPRESENTATION item just underneath the description part and enter Application/JSON</div>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase38.png" alt="" width="174" height="155" /></p>
<ol>
<ol>
<li>
<div>Once you&#8217;ve added the representation type, you can add a representation example.</div>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase39.png" alt="" width="496" height="207" /></p>
<ol>
<li>
<div>Now click on the ADD item in the menu bar on the left underneath RESPONSES (We will skip Caching, Parameters and Body as we don&#8217;t need it)</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase40.png" alt="" /></li>
<li>
<div>Start typing and select a response code you which to return once the message has been send to the service operation.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase41.png" alt="" width="178" height="357" /></li>
<li>
<div>Now you could add a description and add a REPRESENTATION, however in our case we will skip this as a response code 202 Accepted is all we will return.</div>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase42.png" alt="" /></li>
<li>Press save</li>
</ol>
<h3><a name="#Create_a_new_Product">Create a new Product</a></h3>
<p>Now that we have defined our API we need to make it part of a Product. Within Azure API Management this concept has been introduced as a container containing one or more API definitions to which consumers (developers) can subscribe. In short if your API is not part of a product definition consumers can not subscribe to it and use it. More information regarding the &#8216;Product&#8217; definition can be found <a href="http://azure.microsoft.com/en-us/documentation/articles/api-management-key-concepts/">here</a>.</p>
<p>In order to create a new product, we need to perform the following steps:</p>
<ol>
<li>
<div>Select the Products menu item from within the API Management Administration Portal and click on it.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase43.png" alt="" /></li>
<li>
<div>A new window will appear listing all products currently available, in this window click on the ADD PRODUCT item</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase44.png" alt="" /></li>
<li>
<div>Fill out the following form items in order to create a product.</div>
<ul>
<li><strong>A)</strong> Title: Enter the name of the product</li>
<li><strong>B)</strong> Description: Add a description of the product</li>
<li><strong>C)</strong> Require subscription approval: Ensure to check this, as this will require any subscription requests to this product to be approved first</li>
<li><strong>D)</strong> Press Save</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase45.png" alt="" /></li>
<li>
<div>Now that we have created our Product it is time to see if there are any other things we need to configure before we add policies to it and publish it. In order to check the settings by clicking on the newly created product</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase46.png" alt="" /></li>
<li>
<div>On the summary tab, click on the link ADD API TO PRODUCT</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase47.png" alt="" /></li>
<li>
<div>A new window will pop up, select the API you want to add to the product and once done click Save</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase48.png" alt="" /></li>
</ol>
<h3><a name="#Create_a_new_Policy_for_your_defined_API_operations">Create a new Policy for your defined API operations</a></h3>
<p>At this point we have created a product but have not yet published it. We will publish it in a bit, but first we need to set up some policies for the API and the operation we&#8217;ve created earlier, In order to do this follow these steps</p>
<ol>
<ol>
<li>
<div>From the menu bar on your left select the Policies item and in the main window in the policy scope section make the following selections</div>
<ul>
<li><strong>A)</strong> For the API select the API you created earlier</li>
<li><strong>B)</strong> For the Operation select the operation you created earlier</li>
</ul>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase49.png" alt="" /></p>
<ol>
<ol>
<li>
<div>Now in the Policy definition section, click on ADD Policy</div>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase50.png" alt="" /></p>
<ol>
<ol>
<li>
<div>At this point the Empty Policy definition is visible</div>
</li>
</ol>
</ol>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase51.png" alt="" /></p>
<h4><a name="#Setting_up_your_Policy_definition">Setting up your Policy definition</a></h4>
<p>For our API operation to correctly function we are going to have to add a few policies. These policies should take care of the following functionality</p>
<ul>
<li>Authenticate to our Service Bus Topic using our previously created SAS Token</li>
<li>Automatically convert potential JSON messages to their equivalent XML counterpart</li>
<li>Add some additional context information to the inbound message which are converted to brokered message properties when passed won to Azure Service Bus.</li>
</ul>
<p>General information on which policies are available to you within the Azure API Management Administration Portal and how to use them can be found <a href="http://azure.microsoft.com/en-us/documentation/articles/api-management-howto-policies/">here</a></p>
<p>The next few steps will show you how we can add policy statements which will ensure the above mentioned functionality is added.</p>
<ol>
<li>
<div>In the Policy definition section, ensure to place your cursor after the &lt;inbound&gt; tag</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase52.png" alt="" /></li>
<li>
<div>From the Policy statements, select and click on the &#8220;Set HTTP Header&#8221; statement.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase53.png" alt="" /></li>
<li>
<div>A &#8220;Set Header&#8221; Tag will be added to the Policy Definition area, which will leverage the Authorization Header containing our SAS token we created earlier. The steps required are listed below:</div>
<ul>
<li><strong>A)</strong> Put in the value &#8220;Authorization&#8221; for the attribute &#8220;name&#8221;</li>
<li><strong>B)</strong> Put in the following value &#8220;skip&#8221; for the attribute &#8220;exists-action&#8221;</li>
<li><strong>C) </strong>Now get the SAS Token you created earlier, wrap the token string in between a CDATA element and put all of this in between the &#8220;value&#8221; element</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase54.png" alt="" /></p>
<p>Textual example:</p>
<p style="margin-left: 18pt"><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;<span style="color: maroon">set-header<span style="color: red"> name<span style="color: blue">=&#8221;<span style="color: black">Authorization<span style="color: blue">&#8220;<span style="color: red"> exists-action<span style="color: blue">=&#8221;<span style="color: black">skip<span style="color: blue">&#8220;&gt;</span></span></span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p>
<p style="margin-left: 18pt"><span style="font-size: 10pt;font-family: arial;color: black;background-color: white"> <span style="color: blue">&lt;<span style="color: maroon">value<span style="color: blue">&gt;&lt;![CDATA[<span style="color: black"><strong>YOUR SAS TOKEN STRING</strong><span style="color: blue">]]&gt;&lt;/<span style="color: maroon">value<span style="color: blue">&gt;</span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p>
<p style="margin-left: 18pt"><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;/<span style="color: maroon">set-header<span style="color: blue">&gt;</span></span></span></p>
<p>&nbsp;</li>
<li>
<div>Place your cursor just after the closing tag &lt;/set-header&gt;</div>
</li>
<li>
<div>From the Policy statements, select and click on the &#8220;Convert JSON to XML&#8221; statement.</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase55.png" alt="" /></li>
<li>
<div>A &#8220;json-to-xml&#8221; Tag will be added to the Policy Definition area, which contains the instructions resulting in JSON messages to be converted to XML. Ensure that the tag is configured as mentioned below:</div>
<ul>
<li><strong>A)</strong> Put in the value &#8220;content-type-json&#8221; for the attribute &#8220;apply&#8221;</li>
<li><strong>B) </strong>Put in the following value &#8220;false&#8221; for the attribute &#8220;consider-accept-header&#8221;</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase56.png" alt="" /></p>
<p>Textual example:</p>
<p><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;<span style="color: maroon">json-to-xml<span style="color: red"> apply<span style="color: blue">=&#8221;<span style="color: black">content-type-json<span style="color: blue">&#8220;<span style="color: red"> consider-accept-header<span style="color: blue">=&#8221;<span style="color: black">false<span style="color: blue">&#8220;/&gt;</span></span></span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p>
<p>&nbsp;</li>
<li>
<div>Now add &#8220;set-header&#8221; policy statements adding the following headers</div>
<ul>
<li>
<div>
<p><strong><span style="font-size: 10pt;font-family: arial;color: black;background-color: white">A) </span></strong><span style="font-size: 10pt;font-family: arial;color: black;background-color: white">Header name: <strong>MessageId</strong></span></p>
<p>&nbsp;</p>
</div>
<ul>
<li><span style="font-size: 10pt;font-family: arial;color: black">exists-action: <strong>&#8220;skip&#8221;</strong></span></li>
<li><span style="font-size: 10pt;font-family: arial;color: black">value:</span><span style="font-size: 10pt;font-family: arial;color: black"><strong>&#8220;00000000-0000-0000-0000-000000000000&#8221;</strong></span>&nbsp;</li>
</ul>
</li>
<li>
<div>
<p><span style="font-size: 10pt;font-family: arial;color: black"><span style="background-color: white"><strong>B) </strong>Header name: </span><strong>MessageAction</strong></span></p>
<p>&nbsp;</p>
</div>
<ul>
<li><span style="font-size: 10pt;font-family: arial;color: black">exists-action: <strong>&#8220;skip&#8221;</strong></span></li>
<li><span style="font-size: 10pt;font-family: arial;color: black">value=</span><span style="font-size: 10pt;font-family: arial;color: black"><strong>&#8220;Undefined&#8221;</strong></span>&nbsp;</li>
</ul>
</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase57.png" alt="" /><br />
textual example:</p>
<blockquote><p><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;<span style="color: maroon">set-header<span style="color: red"> name<span style="color: blue">=&#8221;<span style="color: black">MessageId<span style="color: blue">&#8220;<span style="color: red"> exists-action<span style="color: blue">=&#8221;<span style="color: black">skip<span style="color: blue">&#8220;&gt;</span></span></span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p></blockquote>
<blockquote><p><span style="font-size: 10pt;font-family: arial;color: black;background-color: white"> <span style="color: blue">&lt;<span style="color: maroon">value<span style="color: blue">&gt;<span style="color: black">00000000-0000-0000-0000-000000000000<span style="color: blue">&lt;/<span style="color: maroon">value<span style="color: blue">&gt;</span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p></blockquote>
<blockquote><p><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;/<span style="color: maroon">set-header<span style="color: blue">&gt;</span></span></span></p>
<p>&nbsp;</p></blockquote>
<blockquote><p><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;<span style="color: maroon">set-header<span style="color: red"> name<span style="color: blue">=&#8221;<span style="color: black">MessageAction<span style="color: blue">&#8220;<span style="color: red"> exists-action<span style="color: blue">=&#8221;<span style="color: black">skip<span style="color: blue">&#8220;&gt;</span></span></span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p></blockquote>
<blockquote><p><span style="font-size: 10pt;font-family: arial;color: black;background-color: white"> <span style="color: blue">&lt;<span style="color: maroon">value<span style="color: blue">&gt;<span style="color: black">Undefined<span style="color: blue">&lt;/<span style="color: maroon">value<span style="color: blue">&gt;</span></span></span></span></span></span></span></span></p>
<p>&nbsp;</p>
<p><span style="font-size: 10pt;font-family: arial;color: blue;background-color: white">&lt;/<span style="color: maroon">set-header<span style="color: blue">&gt;</span></span></span></p>
<p>&nbsp;</p></blockquote>
</li>
<li>
<div>Once you have added all the policy statements, press the save button</div>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase58.png" alt="" /></li>
</ol>
<h3><a name="#Create_a_Group,_an_User_and_Add_subscribe_the_user_to_a_product">Create a Group, an User and Add subscribe the user to a product</a></h3>
<p>Now we have created a new product and assigned policies we need to perform some group/user related actions. This way we can set up a dedicated Group of users which is allowed to use our Product and it&#8217;s containing API.</p>
<p>The steps below will guide you through this process</p>
<ol>
<li>
<div>Now select the Visibility tab, and click on the MANAGE GROUPS link</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase59.png" alt="" /></li>
<li>
<div>You will be redirected to the GROUPS page, on this page click on the ADD GROUP Link</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase60.png" alt="" /></li>
<li>
<div>A new windows will pop up, fill out the following fields</div>
<ul>
<li><strong>A) </strong>Name: Unique name of the group</li>
<li><strong>B) </strong>Description: General description of the group and its purpose</li>
<li><strong>C) </strong>Click on Save</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase61.png" alt="" /></li>
<li>
<div>After you&#8217;ve created the new Group, select the developers menu item and in the main window click on ADD User</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase62.png" alt="" /></li>
<li>
<div>Once again a new window will pop up. In this window fill out the following fields:</div>
<ul>
<li><strong>A)</strong> Email</li>
<li><strong>B)</strong> Password</li>
<li><strong>C)</strong> First and last name</li>
<li><strong>D) </strong>Press Save</li>
</ul>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase63.png" alt="" /></li>
<li>
<div>Now that we have created a new user, we need to make it a member of our group. In order to do so follow the following steps</div>
</li>
<li>
<ul>
<li><strong>A) </strong>Ensure to select the new user</li>
<li><strong>B) </strong>Click on the ADD TO GROUP item and add the user to the earlier created Group</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase64.png" alt="" /></li>
<li>
<div>Now go back to the PRODUCTS menu item and select the product you created earlier</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase65.png" alt="" /></li>
<li>
<div>In the main window follow these steps</div>
<ul>
<li><strong>A) </strong>Click on the Visibility Tab</li>
<li><strong>B) </strong>Allow the new group to subscribe to your product</li>
<li><strong>C)</strong> Click Save</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase66.png" alt="" /></li>
<li>
<div>Now Click on the summary tab and click on the PUBLISH link</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase67.png" alt="" /></li>
<li>
<div>Now select the Developers menu item and click on the user you created earlier</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase68.png" alt="" /></li>
<li>
<div>The main window will now change, in this window click on ADD Subscription</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase69.png" alt="" /></li>
<li>
<div>A window will pop up, in this window ensure to put a checkmark in front of the product you want the user to subscribe to. Once done press the subscribe button</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase70.png" alt="" /></li>
</ol>
<h3><a name="#Test_your_API">Test your API</a></h3>
<p>At this point you have set up your API and now you can proceed with testing it. In order to test we will use the Azure API Management Developer Portal and we will log-on to it using the user account we set up previously.</p>
<p>The steps below list the steps involved:</p>
<ol>
<li>
<div>First log out of the Azure API Management Administration Portal</div>
</li>
<li>
<div>Now login using the email and password of the user you defined earlier</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase71.png" alt="" /></p>
<p>&nbsp;</li>
<li>
<div>In the top menu, select APIS</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase72.png" alt="" /></li>
<li>
<div>Click on the API you created earlier</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase73.png" alt="" /></li>
<li>
<div>Click on the button &#8220;Open Console&#8221;</div>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase74.png" alt="" /></li>
<li>
<div>A form will appear which allows you to send a message to the API. Follow the steps below to send a JSON formatted message to the API.</div>
<ul>
<ul>
<li>A) From the dropdown select a subscription-key (used to authenticate to the API)</li>
<li>B) Add two http headers
<ul>
<li>Content-type: application/json <span style="font-size: 10pt"><em>[indicates that the message you are about to send is formatted as JSON]</em></span></li>
<li>MessageAction: NonExisting <span style="font-size: 10pt"><em>[this will ensure that the message ends up in our Azure Service Bus subscription named Archive, as this subscription is our catch-all</em></span></li>
<li>MessageId: 11111111-1111-1111-1111-111111111111</li>
<li>MessageBatchId: SingleMessageID</li>
</ul>
</li>
<li>C) Add some sample JSON</li>
<li>D) Press HTTP POST</li>
</ul>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase75.png" alt="" /></li>
<li>
<div>Now open up the Service Bus Explorer and connect to your service bus instance</div>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase76.png" alt="" /></li>
<li>
<div>Right Click on the Archive subscription and select the option &#8220;Receive all messages&#8221;</div>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase77.png" alt="" /></li>
<li>
<div>One message should be received, which should meet the following test criteria:</div>
<ul>
<li>
<div>The message should be formatted in XML, although the original message we send was formatted as JSON</div>
<ul>
<li>The following custom message properties should be available</li>
<li>MessageAction: NonExisting</li>
<li>MessageId: 11111111-1111-1111-1111-111111111111</li>
<li>MessageBatchId: SingleMessageID</li>
</ul>
</li>
</ul>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase78.png" alt="" /></p>
<blockquote><p><strong>MY TEST RESULT: PASSED</strong></p>
<p>&nbsp;</p></blockquote>
</li>
<li>Now we will perform another test, but this time we will send an XML formatted message to the API.
<ul>
<li><strong>A)</strong> From the dropdown select a subscription-key (used to authenticate to the API)</li>
<li><strong>B)</strong> Add two http headers
<ul>
<li>Content-type: application/xml <span style="font-size: 10pt"><span style="font-size: 10pt"><em>[indicates that the message you are about to send is XML]</em></span></span></li>
<li>MessageAction: BusinessActivityMonitoring <span style="font-size: 10pt"><em>[this will ensure that the message ends up in our Azure Service Bus subscription named BusinessActivityMonitoring and it will end up in our Archive subscription (as it is our catch-all)]</em></span></li>
</ul>
</li>
<li><strong>C)</strong> Add some sample XML</li>
<li><strong>D) </strong>Press HTTP POST</li>
</ul>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase79.png" alt="" /></li>
<li>Right Click on the BusinessActivityMonitoring subscription and select the option &#8220;Receive all messages&#8221;. One message should be received, which should meet the following test criteria:
<ul>
<li>The message should be formatted in XML</li>
<li>The following custom message properties should be available
<ul>
<li>MessageAction: BusinessActivityMonitoring</li>
<li>MessageId: 00000000-0000-0000-0000-00000000000</li>
<li>MessageBatchId: SingleMessageID</li>
</ul>
</li>
</ul>
</li>
<li><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase80.png" alt="" /><br />
<blockquote><p><strong>MY TEST RESULT: PASSED</strong></p>
<p>&nbsp;</p></blockquote>
</li>
<li>
<div>Now let&#8217;s receive all message from our Archive subscription (it should contain a copy of the previous message). Reason for this, is the fact that the archive subscription is our Catch-All subscription and thus all messages send to the topic end up in this subscription as well.</div>
<p>&nbsp;</p>
<p><img src="https://brauwers-nl.azureedge.net/images/blog/2014/05/052914_1715_Exposingase81.png" alt="" /></p>
<blockquote><p><strong>MY TEST RESULT: PASSED</strong></p>
<p>&nbsp;</p></blockquote>
<p style="margin-left: 18pt">
</li>
</ol>
<h2><a name="#Takeaways">Takeaways</a></h2>
<ol>
<ul>
<li>Ensure to document your API well, this makes life easier for the consumers</li>
<li>Using SAS Tokens you can fairly easy integrate fairly with Azure Service Bus Queues / Topics</li>
<li>If using a policy to set Custom Headers (which you use for setting the Authorization header) ensure to enclose the SAS token within a &lt;![CDATA[ …… ]]&gt; tag</li>
<li>The Azure API Management Key can be found on the developer page of the Azure API <span style="text-decoration: underline"><strong>Developer</strong></span> Portal (f.e <a>https://{namespace}.portal.azure-api.net/developer</a>)</li>
<li>Code examples on how to consume an API are available from the Azure API Developer Portal, by clicking on the menu item APIS and then clicking on the API you want to test</li>
<li>Logging into the Azure API Management Administration Portal must be done using the Azure Management Portal</li>
<li>Azure Management API, in my opinion, could easily be used to virtualize your on premise internet-faced web services (BizTalk generated web services f.e). This way you have one central place to govern and manage them.</li>
</ul>
</ol>
<h2><a name="#Conclusion">Conclusion</a></h2>
<p>I hope this walkthrough contributed in gaining a better understanding of how we as integrators can leverage the Azure API Management Service to expose Service Bus entities. Once you&#8217;ve grasped the concepts you could easily take it a step further and for example involve Azure BizTalk Services which would process messages from certain subscriptions, do some transformations and deliver it to for example another Azure Service Bus Topic, the topic endpoint could then be incorporated into a new API which would allow your API consumers to retrieve their processed messages.</p>
<p>Ah well you get the idea, the possibilities are almost endless as Azure delivers all these building-blocks (services) which enable you to create some amazing stuff for your customers.</p>
<p>I hope to publish a new post in the coming weeks; I&#8217;ve already worked out a scenario on paper which involves Azure API Management, Azure Service Bus, Azure BizTalk Services, Azure File Services, and an Azure Website; however implementing it and writing it down might take some time and currently my spare-time is a bit on the shallow side. Ah well, just stay tuned, check my Twitter and this blog.</p>
<p>Until next time!</p>
<p>Cheers</p>
<p>René</p>
