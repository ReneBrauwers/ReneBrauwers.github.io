---
ID: 4886
post_title: 'No more excuses; protect your (Azure) Soap/rest API, ALWAYS!  &#8211; End-to-End Scenario part 1'
post_name: >
  no-more-excuses-protect-your-azure-soaprest-api-always-end-to-end-scenario-part-1
author: Rene Brauwers
post_date: 2016-11-30 17:57:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2016/11/30/no-more-excuses-protect-your-azure-soaprest-api-always-end-to-end-scenario-part-1/
published: true
tags:
  - end-to-end scenario
categories:
  - API Management
  - App Services
  - 'Authorization &amp; Authentication'
  - Azure Active Directory
  - Logic Apps
  - Security
---
&nbsp;
<h2>Intro</h2>
you don't want to know, how many times I’ve encountered web apps / API’s which are publically accessible and as such don't use any form of authentication whatsoever. So please do me a favour. From now on please protect your APIs.

Azure really makes this easy, especially with the recent (preview) release of <a href="https://blogs.msdn.microsoft.com/appserviceteam/2016/11/17/url-authorization-rules/" target="_blank" rel="noopener noreferrer">URL Authorization</a> and having the Azure Active Directory management functionality available in the <a href="https://blogs.technet.microsoft.com/enterprisemobility/2016/09/12/the-azuread-admin-experience-in-the-new-azure-portal-is-now-in-public-preview/" target="_blank" rel="noopener noreferrer">new portal</a>.

This blogpost will detail an simple but effective end to end scenario in which we will be leveraging
<ul>
 	<li><a href="https://docs.microsoft.com/en-us/azure/active-directory/" target="_blank" rel="noopener noreferrer">Azure Active Directory</a></li>
 	<li><a href="https://blogs.msdn.microsoft.com/appserviceteam/2016/11/17/url-authorization-rules/" target="_blank" rel="noopener noreferrer">URL Authorization</a></li>
 	<li><a href="https://docs.microsoft.com/en-us/azure/logic-apps/" target="_blank" rel="noopener noreferrer">Logic Apps</a></li>
 	<li><a href="https://docs.microsoft.com/en-us/azure/app-service/" target="_blank" rel="noopener noreferrer">App Services</a></li>
 	<li>WCF (yes soap :-)  <a href="https://docs.microsoft.com/en-us/azure/app-service-web/" target="_blank" rel="noopener noreferrer">web app</a></li>
 	<li>Public RESTful service (<a href="http://postcodeapi.com.au/" target="_blank" rel="noopener noreferrer">Postcode API</a>)</li>
 	<li><a href="https://docs.microsoft.com/en-us/azure/api-management/" target="_blank" rel="noopener noreferrer">Azure API Management</a></li>
</ul>
&nbsp;
<h2></h2>
<h2>End-to-End Scenario</h2>
The scenario which we are going to build is a fairly simple one and merely encompasses retrieving customer information. However to make it a bit more interesting the customer information which is to be returned combines two API calls. The initial call will be towards our, by AAD protected ‘CRM’ system (using a WCF Service) and the second call will be going towards a public postal code lookup service (RESTful). The data returned from both of these API’s will then be combined and returned to the user.
<blockquote>The whole end-to-end scenario will be discussed in a few blogposts, and this first post will focus on publishing a WCF service and protecting it using Azure Active Directory and URL Authorization.</blockquote>
<blockquote><span style="color: #303030;">If you’ve been reading my blog for a long time, you will now that I tend to be pretty elaborate and this blogpost is no difference. So yes you might want to skip some items as you might already be familiar with it. </span></blockquote>
<h3>Create the customer WCF service</h3>
So let’s start of with building the sample customer WCF service. For simplicity sake I named this service SimpleSoapService and all it does is return a ‘fixed’ set of customers when invoked.

So first of create a new WCF service using your favourite IDE, in my case Visual Studio <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/wlEmoticon-smile.png" alt="Smile" />

1) In VS select File –&gt; New
2) Select Project

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf0466a0.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTMLf0466a0" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf0466a0_thumb.png" alt="SNAGHTMLf0466a0" width="407" height="269" border="0" /></a>

3) Search for WCF in the searchbox
4) Select WCF Service Application
5) Enter the required details (Name, Location and Solution Name)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf126ce5.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTMLf126ce5" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf126ce5_thumb.png" alt="SNAGHTMLf126ce5" width="422" height="295" border="0" /></a>

6) Delete the IService1.cs and Service1.svc file
7) Add a new WCF service, by right clicking on the WCF project –&gt;  add a new item

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb.png" alt="image" width="430" height="268" border="0" /></a>

8) Name the service for example Customers.svc

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-1.png" alt="image" width="436" height="303" border="0" /></a>

9) Create a new class, by right clicking on the WCF project –&gt; Add and select Class

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf2039bb.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTMLf2039bb" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf2039bb_thumb.png" alt="SNAGHTMLf2039bb" width="442" height="402" border="0" /></a>

10) Name the class CustomerData

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-2.png" alt="image" width="452" height="314" border="0" /></a>

11) Copy and paste the following code into the class. This code will be a representation of the data to be returned
<blockquote>using System;
using System.Runtime.Serialization;

namespace CustomerService
{
[Serializable]
[DataContract]
public class CustomerData
{
[DataMember]
public int CustomerId { get; set; }

[DataMember]
public string FirstName { get; set; }

[DataMember]
public string SurName { get; set; }

[DataMember]
public int PostCode { get; set; }

}
}</blockquote>
12) Now open the ICustomers.cs file and paste the following code (this is the interface class which dictates the service operation to expose)
<blockquote>using System.Collections.Generic;
using System.ServiceModel;

namespace CustomerService
{

[ServiceContract]
public interface ICustomers
{
[OperationContract]
List&lt;CustomerData&gt; GetCustomers();

}

}</blockquote>
13) Now open the Customers.svc.cs file (expland the Customers.svc) and paste the following code (this class implement the ICustomers interface)
<blockquote>using System.Collections.Generic;
namespace CustomerService
{
public class Customers : ICustomers
{
public List&lt;CustomerData&gt; GetCustomers()
{

return CustomerFactory();
}

private List&lt;CustomerData&gt; CustomerFactory()
{
var result = new List&lt;CustomerData&gt;()
{
new CustomerData()
{
CustomerId = 1,
FirstName = "Rene",
SurName = "Brauwers",
PostCode = 2034
}
};

result.Add(new CustomerData()
{
CustomerId = 2,
FirstName = "Mick",
SurName = "Badran",
PostCode = 2031
});

result.Add(new CustomerData()
{
CustomerId = 3,
FirstName = "Crocodile",
SurName = "Dundee",
PostCode = 4880
});

return result;
}
}
}</blockquote>
<h3>Deploy the customer WCF service</h3>
Now that we have created our WCF service, we are ready and can deploy the service into Azure.
<blockquote>In order to do so, you will require a Azure Subscription. If you do not have a Azure Subscription, you can sign up here for a <a href="https://azure.microsoft.com/en-gb/free/" target="_blank" rel="noopener noreferrer">free trial</a></blockquote>
1) From within VS.Net, right click on your WCF service and select Publish…
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf455399.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTMLf455399" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf455399_thumb.png" alt="SNAGHTMLf455399" width="522" height="339" border="0" /></a>

2) The publishing Wizard will now popup, on the initial screen; select the publish Target “Microsoft Azure App Service”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-3.png" alt="image" width="533" height="423" border="0" /></a>

3) Select your Azure Account (if there is no account available, you need to select Add a new account)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-4.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-4.png" alt="image" width="544" height="206" border="0" /></a>

4) As I’ve already added my account, I selected my main account and choose the Azure  Subscription to use, and selected Resource Group as View and then selected NEW to create a new App Service.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-5.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-5.png" alt="image" width="550" height="411" border="0" /></a>

5) On the initial screen fill out the required details, which consist of
<ul>
 	<li>the Web App Name (Name of the web application to deploy – your wcf service)</li>
 	<li>the Azure Subscription in which the app will be deployed</li>
 	<li>the resource-group to deploy to (you can select an existing Resource Group or create a new one (using the new button). In my case I selected an existing resource-group)</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-6.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-6.png" alt="image" width="555" height="197" border="0" /></a>
<ul>
 	<li>the App Service Plan to use (you can select an existing app service plan, or create a new one. In my case I created a new one. Creating a new App Service Plan is straight forward as well. Just select the NEW button next to the App Service Plan.</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-7.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-7.png" alt="image" width="562" height="239" border="0" /></a>
<ul>
 	<li>After you’ve pressed the new button, you simply fill out the required information consisting of the name for the App Service Plan, the location (data-centre) to deploy to and the size (pricing tier)</li>
</ul>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-8.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-8.png" alt="image" width="450" height="509" border="0" /></a>

6) Now that you’ve entered all the data, you can click on the Create button. Which will provision the App Service and the Web App.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-9.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-9.png" alt="image" width="559" height="420" border="0" /></a>

7) Once provisioned, the publish screen will appear. Leave the defaults and select the Publish button.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-10.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-10.png" alt="image" width="563" height="444" border="0" /></a>

8) Once the site has been published a browser window will pop up pointing to your web app.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-11.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-11.png" alt="image" width="561" height="564" border="0" /></a>
<h3>Test customer WCF service</h3>
Now that you’ve published the webservice it is time to test if everything works. The easiest way to test the service would be by means of SOAPUI. So in case you don’t have SOAPUI go and download it <a href="https://www.soapui.org/downloads/soapui.html" target="_blank" rel="noopener noreferrer">here</a>

1) Go ahead and start SOAPUI once it is installed

2) Now in order to test the webservice we will need to WSDL and import it into SOAPUI. The WSDL can be downloaded by browsing to your web-app which you just deployed. In my case the endpoint address would be “http://customerwcfservice.azurewebsites.net/Customers.svc”

3) Now copy any paste the WSDL address (http://customerwcfservice.azurewebsites.net/Customers.svc?singleWsdl)
<blockquote>you can choose from two options, which one you choose does not matter for SOAPUI. In my case I selected the singleWSDL one as this ensures that the whole web-service description is contained in one single file and as such it does not reference any other files. I usually choose this option to avoid any potential issues importing as some clients have issues importing a WSDL which has dependencies on multiple files)</blockquote>
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image65.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image[65]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image65_thumb.png" alt="image[65]" width="589" height="374" border="0" /></a>

3) In SOAP UI, select the SOAP button to create a new project

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-12.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-12.png" alt="image" width="590" height="348" border="0" /></a>

4) Now add the WSDL into the appropriate text box and press OK

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-13.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-13.png" alt="image" width="594" height="228" border="0" /></a>

5) Now open the Request 1 item in the GetCustomers node

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf754e5f.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="SNAGHTMLf754e5f" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/SNAGHTMLf754e5f_thumb.png" alt="SNAGHTMLf754e5f" width="597" height="276" border="0" /></a>

6) Click on the Play button, to test your webservice

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-14.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-14.png" alt="image" width="596" height="199" border="0" /></a>

7)  Your webservice should return a list of customers, which indicates that everything is working <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/wlEmoticon-smile.png" alt="Smile" />

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-15.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-15.png" alt="image" width="607" height="421" border="0" /></a>
<h3>Secure your WCF Service</h3>
Finally now we get to the fun part. Securing your WCF service! As I mentioned in the introduction; Azure really makes securing your web apps easy.

So let’s get started with adding <a href="https://blogs.msdn.microsoft.com/appserviceteam/2016/11/17/url-authorization-rules/" target="_blank" rel="noopener noreferrer">URL Authorization</a> to the earlier created WCF Service.
<h4>Enabling AAD as authentication provider for our web-app</h4>
1) Go to the Azure Portal and select All Resources

2)  search for your web app which you created earlier, in my case this is CustomerWCFService

3) Click on it

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-16.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-16.png" alt="image" width="628" height="204" border="0" /></a>

4) A blade with all settings will now appear. In this blade select the Authentication / Authorization item (part of settings)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-17.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-17.png" alt="image" width="626" height="834" border="0" /></a>

5) Enable App Service Authentication On the Authentication / Authorization screen

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-18.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-18.png" alt="image" width="651" height="198" border="0" /></a>

6) Leave the Allow Anonymous requests (no action) setting

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-19.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-19.png" alt="image" width="660" height="280" border="0" /></a>

7) Click on the Azure Active Directory Authentication Provider

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-20.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-20.png" alt="image" width="660" height="400" border="0" /></a>

8) Select Express as Management mode

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-21.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-21.png" alt="image" width="660" height="269" border="0" /></a>

9) Select Create New AD App and press OK

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-22.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-22.png" alt="image" width="658" height="774" border="0" /></a>

10) Press the Save Icon

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-23.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-23.png" alt="image" width="660" height="594" border="0" /></a>
<h4>Protecting our web-service</h4>
Now we’ve set up the initial authentication / authorization part it is time to actually protect our web-service.

In order to do so, we will be required to add a Authorization.json file to our www-root directory.

This file will contain information of which web resources to protect (Ie which require a user to be authenticated). In our scenario we will set it up such that a user will be able to obtain the WSDL, however posting to the webservice will require a user to be authenticated.

In short we want GET operations to be allowed and a POST operation will require a user to be authenticated. To meet these requirements the contents of the Authorization.json file needs to be as follows
<blockquote>{
"routes": [
{
"http_methods": [ "GET" ],
"path_prefix": "/",
"policies": {
"unauthenticated_action": "AllowAnonymous"
}
},
{
"http_methods": [ "POST" ],
"path_prefix": "/Customers.svc",
"policies": {
"unauthenticated_action": "RejectWith401"
}
}
]
}</blockquote>
The configuration mentioned above, indicates that all GET operations are allowed and that POST operations to /Customers.svc require a user to be authenticated.
<blockquote><span style="color: #303030;"><strong>Please note</strong>; adding the authorization.json file to you vs.net project and making it part of your deployment, will cause an error. “The page cannot be displayed because an internal server error has occurred.”  This is most likely due to the fact that the feature is currently in preview. Looking at the log indicates that there is a deserialization issue. </span>

System.Runtime.Serialization.SerializationException: There was an error deserializing the object of type Microsoft.Azure.AppService.Routes.RoutesConfig. Encountered unexpected character 'ï'. ---&gt; System.Xml.XmlException: Encountered unexpected character 'ï'.

<span style="color: #333333;">In order to resolve this issue we will be adding the authorization file manually by leveraging the App Service Editor from within the Portal <img class="wlEmoticon wlEmoticon-smile" style="border-style: none;" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/wlEmoticon-smile.png" alt="Smile" />. Which is being described below:</span></blockquote>
<h5>Enabling authentication</h5>
1) On our settings blade of the web app, scroll down to the DEVELOPMENT TOOLS section and click on App Service Editor (preview)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-24.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-24.png" alt="image" width="660" height="225" border="0" /></a>

2) Click on the GO link, which will redirect you to the online editor.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-25.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-25.png" alt="image" width="660" height="280" border="0" /></a>

3) In the online-editor, right click on the wwwroot and select New File

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-26.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-26.png" alt="image" width="411" height="531" border="0" /></a>

4) Name the file Authorization.json

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-27.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-27.png" alt="image" width="423" height="547" border="0" /></a>

5) Copy and paste the following contents
<blockquote>{
"routes": [
{
"http_methods": [ "GET" ],
"path_prefix": "/",
"policies": {
"unauthenticated_action": "AllowAnonymous"
}
},
{
"http_methods": [ "POST" ],
"path_prefix": "/Customers.svc",
"policies": {
"unauthenticated_action": "RejectWith401"
}
}
]
}</blockquote>
6) Your file will be auto-saved and as authentication enabled which will require POST operation to the customer service to be authenticated.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-28.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-28.png" alt="image" width="660" height="370" border="0" /></a>
<h4>Test the protected webservice</h4>
Before we can test if the authentication part is working, it is required to restart the web-app (this is only a temporary workaround, once the url authentication feature goes GA this will no longer be required). Once restarted we can go ahead and test the web-service.

1) Go back to our web-app blade in the portal and click on the overview item.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-29.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-29.png" alt="image" width="660" height="365" border="0" /></a>

2) We now can see options to restart the web-app. Click on it

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-30.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-30.png" alt="image" width="660" height="234" border="0" /></a>

3). Now browse to your web site, and you should notice that it will display the Customer Service page (It is a get operation, which is allowed)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-31.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-31.png" alt="image" width="660" height="493" border="0" /></a>

4) Now go back to SOAPUI and try to execute the webservice. uOu will notice that you will receive a 401, as this one requires you to be authenticated.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image-32.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2016/11/image_thumb-32.png" alt="image" width="660" height="116" border="0" /></a>
<h3></h3>
<h3>Conclusion</h3>
This post has guided you through setting up a WCF service and protecting it using URL Authentication. Although the post was lengthy you will notice that setting up url-authentication is actually quite simple and only involving a few steps.

Anyways, In my next post we will be further expanding on our end-to-end solution focusing on Logic Apps and API Management.

So stay tuned.

Cheers

René