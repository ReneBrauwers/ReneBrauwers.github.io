---
ID: 1262
post_title: 'How to: Store LinkedIn profile info in CRM2011 using BizTalk 2013 and Windows Azure Service Bus'
post_name: >
  how-to-store-linkedin-profile-info-in-crm2011-using-biztalk-2013-and-windows-azure-service-bus
author: Rene Brauwers
post_date: 2013-04-08 22:22:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2013/04/08/how-to-store-linkedin-profile-info-in-crm2011-using-biztalk-2013-and-windows-azure-service-bus/
published: true
tags:
  - Azure
  - BizTalk
  - BizTalk 2013
  - CRM2011
  - How to
  - Hybrid Integration
  - LinkedIn
  - Social
categories:
  - Azure
  - BizTalk
---
<blockquote><span style="color: #000000;">Note: Please be advised, this post most likely contains grammar mistakes I am sorry for that but I was in a hurry and wanted to share this with you all. Anyway; enjoy…</span></blockquote>
<h1><span style="color: #000000;">Once upon a time…</span></h1>
<span style="color: #000000;">there was this BizTalk Integrator who was wondering how hard it would be to get LinkedIn profile information and store this in Microsoft Dynamics CRM 2011. He didn’t waste a lot of time and within no time he produced a diagram showing on a high level how he intended to tackle this issue. </span>

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/Flow.png"><img class="aligncenter" title="Flow" alt="Flow" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/Flow_thumb.png" width="430" height="314" border="0" /></a>
<ol>
	<li>
<div align="left">Redirect user to LINKEDIN to get Authorization Code.</div></li>
	<li>
<div align="left">Authorization code is send back.</div></li>
	<li>
<div align="left">Request OAUTH 2.0 Access Token using obtained authorization code</div></li>
	<li>
<div align="left">OAUTH Access token is send back.</div></li>
	<li>
<div align="left">Establish Service Bus Connection. Create a Topic, Subscription and Filters if they don’t exists. Create message containing OAUTH Access token and set brokered message properties. Once done publish message.</div></li>
	<li>
<div align="left">BizTalk retrieves message from Topic Subscription.</div></li>
	<li>
<div align="left">BizTalk creates a GET Request including the OAUTH Token and sends it to the LinkedIn api to request the user’s information.</div></li>
	<li>
<div align="left">BizTalk receives a response from LinkedIn, and creates a new Atom Feed message containing the Contact Properties required to add a new contact in CRM</div></li>
	<li>
<div align="left">BizTalk sends a POST request to the CRM 2011 XRMServices/2011/OrganizationData.svc/ContactSet/ endpoint such that a new Contact is created</div></li>
</ol>
&nbsp;

Okay all kidding aside; in this blog post I will dive deeper in to the above mentioned <span style="color: #555555;">scenar</span>io, and will highlight the ins-and-outs as mentioned in steps 1 to 9. So let’s get started.
<blockquote><span style="font-size: small;">For all those, not interested in reading about it; but rather want to see the pre-recorded demo. Click <a href="http://www.youtube.com/watch?v=-NONmqs1K5c" target="_blank" rel="noopener noreferrer">here</a> <img class="wlEmoticon wlEmoticon-winkingsmile" alt="Winking smile" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/wlEmoticon-winkingsmile.png" /></span></blockquote>
<h1>Website (steps 1 to 5) – Get OAUTH token and send it to Service bus</h1>
One of the first challenges we have to tackle consist of obtaining a token from LinkedIn which allows us to perform authorized requests against a certain user profile. In English; in order to get LinkedIn profile information we need to acquire approval from the profile owner. Once we have obtained this permission we need to obtain an OATH 2.0 token which can then be used to authorize our requests to the LinkedIn API.

So how did I solve this ‘challenge’, well I build this small <a href="http://brauwers.azurewebsites.net/" target="_blank" rel="noopener noreferrer">website</a> based on MVC 4*  and hosted it for free in <a href="http://www.windowsazure.com/en-us/pricing/details/#header-1" target="_blank" rel="noopener noreferrer">Azure</a>. This website uses the default MVC 4 template, and all I did is change some text and added a new controller and subsequent view. The controller, which I gave a self-explanatory name ‘LinkedInController’ takes care of all logic required (getting the OAUTH token and sending it so Windows Azure Service Bus).
<blockquote><span style="font-size: small;">Read all about </span><a href="http://www.asp.net/mvc" target="_blank" rel="noopener noreferrer"><span style="font-size: small;">MVC</span></a><span style="font-size: small;"> here, and in order to learn more about MVC and other cool stuff go to </span><a href="http://www.pluralsight.com/training" target="_blank" rel="noopener noreferrer"><span style="font-size: small;">Pluralsight</span></a><span style="font-size: small;">)</span></blockquote>
Below a more detailed overview, on how I took care of steps 1 to 4.
<blockquote><span style="font-size: small;">Note: I gathered the information on how to authenticate using OAUTH 2.0 from  </span><a href="http://developer.linkedin.com/documents/authentication" target="_blank" rel="noopener noreferrer"><span style="font-size: small;">this page</span></a></blockquote>
&nbsp;
<h2> </h2>
<h4>1) Obtaining profile user permission.</h4>
On the default action (Index) in my case, I perform a good-old fashioned redirect to a LinkedIn site in order to obtain the user’s authorization (permission) to query his/hers LinkedIn profile. On this site the user is presented with the following screen.

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb.png" width="174" height="244" border="0" /></a>
<h4>2) Obtaining linkedIN’s authorization code.</h4>
Once the user has allowed access; he or she will be redirected back to the mvc website. This redirection request from LinkedIn contains a Authorization_Code parameter and points to a specific Action within the LinkedIn Controller.This action will extract the Authorization Code and this code will be stored internally.
<h4>3) Request OATH 2.0 token.</h4>
Next a HTTP Post to the LinkedIn API will be executed, containing the users authorization code and some other vital information
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb1.png" width="244" height="66" border="0" /></a>
<h4>4) Process JSON response.</h4>
As a result we retrieve a JSON response which is serialized back into a custom object.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb2.png" width="244" height="68" border="0" /></a>

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb3.png" width="244" height="96" border="0" /></a>
<h4>5) Publish message to windows azure service bus.</h4>
Once we’ve filled out custom object, I send it to a custom build class*, which then sends this object to a Windows Azure Topic.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image4.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb4.png" width="244" height="91" border="0" /></a>
<blockquote><span style="font-size: small;">* The custom class I’ve build is my own implementation of the REST API for creating TOPICs, Subscriptions, Filters and sending messages to Windows Azure Service Bus (see <a href="http://msdn.microsoft.com/en-us/library/windowsazure/hh780717" target="_blank" rel="noopener noreferrer">Service Bus REST API Reference</a>). I’ve build this class a while back and the main reason I;’ve build it, is such that I can reuse it in my Demo Windows Phone Apps and Demo Windows Store Apps). I will not list the code here, however if you are interested just send me an </span><a href="mailto:info@brauwers.nl" target="_blank" rel="noopener noreferrer"><span style="font-size: small;">email</span></a><span style="font-size: small;"> and I’ll send it to you</span></blockquote>

<hr />

<em><span style="color: #555555; font-size: x-large;">The above mentioned class requires that some specific information is passed in, this is explained below</span></em>

<hr />

<h5>Payload</h5>
The first required parameter is of type Object and should contain the message object which is to be sent to service bus.
<h5>Brokered message Properties</h5>
This parameter contains the key and value pairs adding additional context to the message send to service bus. In my demo I’ve added the following key value pairs to the message. The importance of these properties will be made clear in a little while.

Source=linkedIN
Method=select
Object=personalData
Auth=[DYNAMIC]
<blockquote><span style="font-size: small;">Note: Auth will be replaced in code with the actual OAuth 2.0 token received earlier and is added to the context of the message. Later on this is used by BizTalk (but more on this later)</span></blockquote>
<h5> </h5>
<h5>Service bus name space</h5>
One of the required parameters is the service namespace which is to be used. In my demo I created a namespace called brauwers
<blockquote><span style="font-size: small;">A service namespace provides a scoping container for addressing Service Bus resources within your application.</span></blockquote>
<h5>Service bus Management Credentials</h5>
Two other parameters which are required, consist of the so-called issuer-name and issuer-token. These credentials are required if we want to perform actions on the service bus resource like (posting messages, or performing management operations). My component requries that you supply the owner (or full access) credentials as this component will try to perform some management operations (creating subscriptions) which needs these elevated privileges
<h5>Service bus Topic name</h5>
This parameter should consist of the name of the Topic to which the message will be published. In my demo I’ve chosen the value “SocialAccessTokens”.
<blockquote><span style="font-size: small;">Note: in case the topic does not exists the custom class will create it (hence the reason we need to use an issuer who has elevated (create) privileges</span></blockquote>
<h5>Service bus Topic subscription</h5>
This parameter holds the desired topic subscription name, which we want to create. In my demo I’ve chosen the value “LinkedINSubscription”
<blockquote><span style="font-size: small;">Note: Eventually a filter will be added to this subscription, such that messages matching a this filter or rule (see brokered message properties mentioned earlier) will be ‘attached’ to this subscription. The importance of this, will be made clear further down in this blog post when we will explain how BizTalk is configured). On another note; in case this subscription does not exist a the custom class will create it (hence the reason we need to use an issuer who has elevated (create) privileges.</span></blockquote>
<h5>Service bus Topic Filter name</h5>
This parameter contains the name to use for the filter, which will be attached to the topic subscription.
<h5>Service bus Topic Filter expression (Sqlfilter)</h5>
This parameter is actually pretty important as this parameter defines the rule which applies to the subscription it is attached to. The filter I used in my example  --&gt; Source = 'linkedIN' (keep in mind that we set a brokered message property earlier containing these values)
<blockquote><span style="font-size: small;">Note: the filter in my example should be interpreted as follows: messages send to the topic (SocialAccessTokens) which contain a brokered message property key named source, and of which the value matches the string “linkedIN” should be ‘attached’ to the subscription “LinkedINSubscription”. </span></blockquote>

<hr />

<h1>BizTalk (steps 6 to 9) – Connecting the dots, from service bus to crm2011</h1>
Well the previous steps; ensured that a message containing the OAUTH 2.0 token, required to perform LinkedIn profile requests on behalf of the profile owner. The next steps should result in a new contact in Dynamics CRM 2011 based upon the data retrieved from LinkedIn. These steps involve BizTalk and some of it’s ‘new’ (or let’s reformulate this: out of the box) capabilities, consisting of the SB-Message adapter and the WCF-WebHttp Adapter.

So let’s get started.
<h4>6) Setting up BizTalk such that is connects to windows azure service bus.</h4>
At this point a message has been delivered to a Windows Azure Service Bus topic. However now we need to pull this message into BizTalk and for this to work we need to perform a few steps like
<ul>
	<li>a) defining a property schema used to capture the custom brokered message properties.</li>
	<li>b) define a schema which reflects the message send to windows azure service bus</li>
	<li>c) deploy the artifacts</li>
	<li>d) configure a receive port and location to get the service bus message(s) in BizTalk</li>
</ul>
These steps will be explained further below
<h5>Defining a property schema used to capture the custom brokered message properties.</h5>
&nbsp;
<blockquote><span style="font-size: small;">Note: Before proceeding ensure you’ve created a new solution and BizTalk project </span><span style="font-size: small;">and don’t forget to sign it with a strong key and give it an application name.
</span><a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image5.png"><span style="font-size: small;"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb5.png" width="244" height="130" border="0" /></span></a>
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image6.png"><span style="font-size: small;"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb6.png" width="244" height="121" border="0" /></span></a>
<ol><!--EndFragment--></ol>
</blockquote>
In step 5; we added a few custom properties to the message context consisting of the keys “Source,Method,Object and Auth” . In my example I want to use these properties as Promoted Properties within BizTalk, such that I can do some nice stuff with them (more about this in step 7). So in order to cater for this we need to create a new Property Schema in BizTalk and all this schema needs to contain are elements which are named <span style="text-decoration: underline;">exactly</span> like the brokered message property key names.

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image7.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb7.png" width="244" height="113" border="0" /></a>
<h5>Define a schema which reflects the message send to windows azure service bus</h5>
In order for BizTalk to recognize a message it’s message definition needs to be known internally. In English; if we want to work with the message as send to windows azure service bus in BizTalk, we need to ensure that BizTalk can recognize this message and thus we need to define. How this is done is explained below
<ol>
	<li>In my example I send a message to the service bus; this message is actually a representation of an internal defined class. This class is defined as mentioned below:
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image8.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb8.png" width="244" height="103" border="0" /></a></li>
	<li>So how do we translate this message to a BizTalk Schema? Well we have a few options, and I’ll use the not so obvious approach and that’s the manual approach. In order to do so, we need to create a new schema in BizTalk which has the following properties and structure:Target Namespace should match the DataContract Namespace, thus in my case : <a title="http://brauwers.nl/socialauthenticator/linkedin/v1.0/" href="http://brauwers.nl/socialauthenticator/linkedin/v1.0/">http://brauwers.nl/socialauthenticator/linkedin/v1.0/</a>
The Main Root should match the class name, thus in my case :ApiAccessInformation
The main root should contain the following elements, all of them of type string
1) apiName
2) apiClientId
3) apiClientSecret
4) apiAccessScope
5) authorizationKeyThe end result should look like depicted in the image below
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image9.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb9.png" width="244" height="78" border="0" /></a></li>
</ol>
<h5> </h5>
<h5>Deploy the artifacts</h5>
So far so good, but first let’s deploy the BizTalk project, and once it is deployed we can go ahead and start configuring it. (I named my BizTalk Application CRM2011)

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image10.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb10.png" width="244" height="125" border="0" /></a>
<h5>Set up BizTalk Receive Port and Receive Location</h5>
So far so good, but now we need to put everything together such that we can configure BizTalk to read the Service Bus Topic Subscription and process any messages published to it. How this is done is explained below.
<ol>
	<li>Go to your just deployed BizTalk Application and create a one-way receive-port and name it appropriately.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image11.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb11.png" width="244" height="197" border="0" /></a></li>
	<li>Created a new receive location, choose the SB-Messaging adapter, select your receive host and select the xml_receive pipeline component.<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image12.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb12.png" width="244" height="197" border="0" /></a></li>
	<li>Configured the SB-Messaging adapter. Enter the appropriate subscription url. In my case this would be: <a title="sb://brauwers.servicebus.windows.net/socialaccesstokens/subscriptions/LinkedINSubscription" href="sb://brauwers.servicebus.windows.net/socialaccesstokens/subscriptions/LinkedINSubscription">sb://brauwers.servicebus.windows.net/socialaccesstokens/subscriptions/LinkedINSubscription</a>
sb://[NAMESPACE].servicebus.windows.net/[TOPIC_NAME]/subscriptions/[SUBSCRIPTION CONTAINING MESSAGES]
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image13.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb13.png" width="176" height="244" border="0" /></a></li>
	<li>Fill out the required authentication information, consisting of
the Access Control Service uri and a valid Issuer Name and Issuer Key
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image14.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb14.png" width="177" height="244" border="0" /></a></li>
	<li>Select the property schema to use such that the Brokered Message Properties defined earlier are treated as Promoted Properties within BizTalk.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image15.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb15.png" width="177" height="244" border="0" /></a></li>
	<li>Press OK</li>
</ol>
<h4>7) Setting up BizTalk such that the LinkedIn API can be queried.</h4>
At this point we’ve set up BizTalk such that it can retrieve the messages published to windows azure service bus and tread the brokered message properties as Promoted properties. This section will now list and explain the next steps which are needed in order to query the LinkedIn API using the wcf-webhttp adapter.
<h5>Creating a new send port leveraging the new wcf-webhttp adapter</h5>
So how do we configure the wcf-webhttp adapter such that we can send a GET request to the linkedIn API, well the steps below will show how this is achieved.
<ol>
	<li>Create a new static solicit-response port and name it appropriately, choose the WCF-WebHttp adapter, choose your prefered send handler and select XML_Transmit as Send Pipeline and XML_Receive as Receive Pipeline
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image16.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb16.png" width="244" height="198" border="0" /></a></li>
	<li>Now click on configure, such that we can configure the WCF-WebHttp adapter.</li>
	<li>First configure the Endpoint address; I’ve used <a title="https://api.linkedin.com/v1/people" href="https://api.linkedin.com/v1/people">https://api.linkedin.com/v1/people</a>
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image17.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb17.png" width="244" height="73" border="0" /></a></li>
	<li>Now we set the HTTP Method and URL Mapping, for this I use the BtsHttpUrlMapping element, which contains an operation method which is set to GET and the rest of the Address Uri which consists of a list of profile fields we want to select and an oauth2 access token which value we will set using variable mapping (step 5 below)
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/SNAGHTML3f1a1cce.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="SNAGHTML3f1a1cce" alt="SNAGHTML3f1a1cce" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/SNAGHTML3f1a1cce_thumb.png" width="244" height="108" border="0" /></a></li>
	<li>In our BtsHttpUrlMapping we have set a dynamic parameter which was named {Auth}. The value we want to assign to this, will be the oAuth 2.0 token which is available to us as a promoted property (see explanation step 6 “Setting up BizTalk such that is connects to windows azure service bus“). So for Propery Name we select the element name containing the promoted propery value (in my case Auth) and for property namespace we enter the namespace name of our created property schema.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image18.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb18.png" width="244" height="198" border="0" /></a></li>
	<li>Once done press OK.</li>
	<li>Now select the security tab and ensure that the security mode is set to Transport and the Transport client credential type is set to NONE
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image19.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb19.png" width="194" height="244" border="0" /></a></li>
	<li>Now go to the Messages Tab and ensure that the following Outbound Http Headers are set
Accept: application/atom+xml;
Content-Type: application/atom+xml;charset=utf-8
Accept-Encoding: gzip, deflate
User-Agent: BizTalk<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image20.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb20.png" width="244" height="113" border="0" /></a></li>
	<li>Make sure that the outbound message is surpressed, by indicating the GET verb-name
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image21.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb21.png" width="244" height="63" border="0" /></a></li>
</ol>
<h4>8) Dealing with the response message received from LinkedIn.</h4>
Well at this point we have send a GET request to linkedIn, however we haven’t set up the logic yet to cater for processing the response message send back to BizTalk by the LinkedIn API. So let’s dive in to it straight away.
<h5>Figuring out the response format from LinkedIn</h5>
In order to figure out the response message which is sent back to BizTalk, I have used Fiddler and composed a GET request as it would be sent by BizTalk.
<ol>
	<li>Open fiddler, and go to the compose tab, and use the following Uri, and replace [TOKEN] with the OAUTH token (note: you can get your token by going to the <a href="http://brauwers.azurewebsites.net/">web application</a>, but be aware if you allow access to your profile, you will be part of my DEMO and as a result your profile information will be stored in my Demo CRM environment <img class="wlEmoticon wlEmoticon-smilewithtongueout" alt="Smile with tongue out" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/wlEmoticon-smilewithtongueout.png" />)
<a title="https://api.linkedin.com/v1/people~:(first-name,last-name,email-address,date-of-birth,main-address,primary-twitter-account,picture-url,headline,interests,summary,languages,skills,specialties,educations,certifications,courses,three-current-positions,three-past-positions,following,job-bookmarks,phone-numbers,bound-account-types,twitter-accounts,im-accounts)?oauth2_access_token=[TOKEN]" href="https://api.linkedin.com/v1/people~:(first-name,last-name,email-address,date-of-birth,main-address,primary-twitter-account,picture-url,headline,interests,summary,languages,skills,specialties,educations,certifications,courses,three-current-positions,three-past-positions,following,job-bookmarks,phone-numbers,bound-account-types,twitter-accounts,im-accounts)?oauth2_access_token=[TOKEN]">https://api.linkedin.com/v1/people~:(first-name,last-name,email-address,date-of-birth,main-address,primary-twitter-account,picture-url,headline,interests,summary,languages,skills,specialties,educations,certifications,courses,three-current-positions,three-past-positions,following,job-bookmarks,phone-numbers,bound-account-types,twitter-accounts,im-accounts)?oauth2_access_token=[TOKEN]</a></li>
	<li>Add the following request headers
Accept: application/atom+xml;
Content-Type: application/atom+xml;charset=utf-8
Accept-Encoding: gzip, deflate
User-Agent: BizTalk<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image22.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb22.png" width="244" height="78" border="0" /></a></li>
	<li>Press Execute, and double click on the 200 Response, and select the RAW tab
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image23.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb23.png" width="244" height="139" border="0" /></a></li>
	<li>Copy and past the response message, and store it as an xml file</li>
	<li>Now go back to your BizTalk Solution and right click in your project and select –&gt; Add new generated item)
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image24.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb24.png" width="244" height="166" border="0" /></a></li>
	<li>Select Generate Schema
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image25.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb25.png" width="244" height="126" border="0" /></a></li>
	<li>Select Well-Formed XML and browse to your saved XML file containing the response and press OK
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image26.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb26.png" width="244" height="190" border="0" /></a></li>
	<li>A new schema should be created and it should look similar to the one displayed below
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image27.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border-width: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb27.png" width="244" height="121" border="0" /></a></li>
</ol>
<h4>9) Transforming the LinkedIn Profile RESPONSE and constructing a CRM POST request.</h4>
Well at this point we have the response definition, and now we are ready for constructing our POST request which eventually will ensure that we store the data in CRM. So let’s get started with these final pieces.
<h5>Figuring out the request message structure needed for inserting data into CRM</h5>
In order to figure out the request or payload message which is sent as body in the POST request to CRM, I have used this CRM Codeplex project <a href="http://crm2011odatatool.codeplex.com/">CRM ODATA Query Designer</a> Below the steps I performed to obtain the correct XML structure (note you need to have installed the codeplex project)
<ol>
	<li>Open up CRM in your browser and select the CRM Odata Query Designer</li>
	<li>Select ContactSet
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image28.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb28.png" width="244" height="114" border="0" /></a></li>
	<li>Press Execute and switch to the results (ATOM) tab
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image29.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb29.png" width="244" height="108" border="0" /></a></li>
	<li>Now look for the first ID and copy the url
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image30.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb30.png" width="244" height="59" border="0" /></a></li>
	<li>Copy this url in your browser and then select source</li>
	<li>Copy the result xml, and store it locally</li>
	<li>Open the xml file and let’s clean it up, such that only those fields are left which matter to you. See below my cleaned-up xml structure (please note the elements named new_ ,these are custom fields I’ve added)
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image31.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb31.png" width="244" height="107" border="0" /></a></li>
	<li>Now open up your BizTalk Solution, right click and select the “Add new generated item” like you did before and generate a schema using the well-formed xml.</li>
	<li>An xsd schema has been created, and it should look similar to the one depicted below. Note that I manually added the min and max occurance
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image32.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb32.png" width="244" height="115" border="0" /></a></li>
</ol>
<h5>Create a BizTalk Mapping which creates the CRM Insert Request</h5>
At this point we can create a mapping between the LinkedInResponse message and the CRM Insert message; I’ve done so and named this map “LinkedInPersonResponse_TO_AtomEntryRequest_CrmInsert.btm”
<h5>Build the BizTalk solution and Deploy</h5>
Now that we have all artifacts ready, we can deploy our BizTalk Solution once again and finish up the last part of our BizTalk configuration.
<h5>Creating a new send port leveraging the new WCF-WebHttp adapter to send a POST request to CRM</h5>
Our final step, before testing, consists of adding one last send port which leverages the WCF-WebHttp adapter once more; but this time we will be performing a HTTP POST to the crm endpoint (..XRMServices/2011/OrganizationData.svc/ContactSet/) which will result in a new contact record in CRM 2011. Below I’ll show you how to configure this last sendport.
<ol>
	<li>Create a new static One-Wayport and name it appropriately, choose the WCF-WebHttp adapter, choose your prefered send handler and select XML_Transmit as Send Pipeline
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image33.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb33.png" width="244" height="196" border="0" /></a></li>
	<li>Now click on configure, such that we can configure the WCF-WebHttp adapter.</li>
	<li>First configure the Endpoint address, which should point to an address ending on OrganisationData.svc/ContactSet .The full endpoint I’ve used is <a title="http://lb-2008-s07/Motion10/XRMServices/2011/OrganizationData.svc/ContactSet/" href="http://lb-2008-s07/Motion10/XRMServices/2011/OrganizationData.svc/ContactSet/">http://lb-2008-s07/Motion10/XRMServices/2011/OrganizationData.svc/ContactSet/</a>
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image34.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb34.png" width="244" height="105" border="0" /></a></li>
	<li>Now we set the HTTP Method and URL Mapping, this is pretty straightforward as we only have to add the verb POST.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image35.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb35.png" width="244" height="124" border="0" /></a></li>
	<li>Now select the security tab and ensure that the security mode is set to TransportCredentialOnly and the Transport client credential type is set to NTLM
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image36.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb36.png" width="244" height="82" border="0" /></a></li>
	<li>Now go to the Messages Tab and ensure that the following Outbound Http Headers are set
User-Agent: BizTalk
Content-Type: application/atom+xml<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image37.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb37.png" width="175" height="244" border="0" /></a></li>
	<li>Once done press OK</li>
	<li>Now select the outbound maps section, and select the map you’ve created and deployed earlier.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image38.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb38.png" width="244" height="199" border="0" /></a></li>
	<li>Last but not least, add a Filter, stating “BTS.MessageType = person” such that the we can route the LinkedIn Response message (Person) to this sendport.
<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image39.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb39.png" width="244" height="199" border="0" /></a></li>
	<li>At this point, we are done and we can now start the application and see if everything works.</li>
</ol>
<blockquote><span style="color: #555555; font-size: small; background-color: #ffffff;">Note: in order to be able to add records to CRM, you need to ensure that you’ve added the Host Intance User Account used for the selected Send Handler in CRM and granted this account the correct permissions. Failure doing so, will result in a 400 Bad Request which initially will throw you off.</span>

<span style="color: #555555; font-size: small; background-color: #ffffff;">Below I’ve added a few screenshots
</span>

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/SNAGHTML404b9a3d.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTML404b9a3d" alt="SNAGHTML404b9a3d" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/SNAGHTML404b9a3d_thumb.png" width="244" height="162" border="0" /></a>

<span style="font-size: small;">The account svc-bts trustetd is the account used by the LowLatency_Host which I use to connect to the XRM service from CRM.</span>

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image40.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb40.png" width="244" height="149" border="0" /></a>

<span style="font-size: small;">Subsequently I’ve created an account in CRM2011 for this user </span>

<a href="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image41.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" alt="image" src="https://blogbrauwersimages.blob.core.windows.net/images/uploads/2013/04/image_thumb41.png" width="244" height="59" border="0" /></a>

<span style="font-size: small;">and added this account (for testing purposes) to the System Administrator role.</span>

&nbsp;

&nbsp;

&nbsp;</blockquote>

<hr />
<p align="center"><em><span style="color: #555555; font-size: x-large;"><a href="http://www.youtube.com/watch?v=-NONmqs1K5c" target="_blank" rel="noopener noreferrer">Click Here to see a video showcasing above mentioned scenario</a></span></em></p>


<hr />