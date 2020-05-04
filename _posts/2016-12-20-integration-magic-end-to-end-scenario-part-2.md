---
ID: 5171
post_title: 'Integration Magic &#8211; End-to-End Scenario part 2'
post_name: >
  integration-magic-end-to-end-scenario-part-2
author: Rene Brauwers
post_date: 2016-12-20 23:35:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2016/12/20/integration-magic-end-to-end-scenario-part-2/
published: true
tags:
  - Step by step
categories:
  - App Services
  - Azure
  - DocumentDB
  - Logic Apps
---
In our previous post, I guided you through setting up a WCF service and protecting it using URL Authentication. Although a lengthy post you would have noticed that setting up url-authentication is actually quite simple and only involving a few steps.

Anyways, in this post we will be focusing on adding the integration magic, without adding a single line of custom code, using Azure Logic Apps.

The integration magic which we will be adding will take care of the following functionality within our end-to-end scenario.

A request will come in which will start our process which is to retrieve a list of customers.

The customer information to be retrieved combines the result from two sources; the first source being the WCF service we build in our previous post and the second source a public rest api. The data which is to be returned to the caller as such will consist of the base data originating from the wcf services enriched with data obtained from the public rest api.
<h3>Visualizing the flow</h3>
Before we start implementing the solution using Logic Apps it is always a good practice to work-out the actual process flow using a tool such as Microsoft Visio.

Having said that, let’s eat my own dogfood. Low and behold, see below the diagram depicting the process and an explanation of the process.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0021.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image002[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0021_thumb.jpg" alt="clip_image002[1]" width="244" height="175" border="0" /></a>

The process kicks off whenever a <b>http post</b> requesting a list of customer data is being made to Logic Apps (1). Once received within logic apps a new message (soap request) has to be created (2). Once created this message is being offloaded to the custom WCF service (3), we created in the previous post. If the call is successful the webservice will return a list of customers (4). The information contained within the response contains the following data: customerId, FirstName, SurName and postcode.

The postcode value(s) contained within this response is subsequently used to retrieve detailed location information.

In order to retrieve this location information, logic apps will perform a loop over the response message (5), extract the postal code and invoke a custom rest API to do the location lookup (6). The response received contains the following data: Suburb name, postcode, state-name, state abbreviation, locality and the latitude and longitude of the locality.

This data and the basic customer data is then combined and temporarily persisted in DocumentDB (7).

//Reason, for leveraging this external persistence store is to make life easier for us, as we want //enrich all the customer data with additional information retrieved from the second api call and //return it in one go to the caller. Currently there is no easy way of doing this directly from within //logic-apps as, however, have no fear; in one of the next releases a feature to store session state //within a logic app will be implemented and thus we would no longer need to result to an //intermediate ‘session state’ store.

This process is then repeated for all customers and once we have iterated over all customer records we exit the loop and retrieve all ‘enriched’ documents stored in DocumentDB (8) which we then will return to the caller. The information returned to the caller will then contain the following data; FirstName, LastName and Location information consisting of Locality, State Name, SubUrb, Postcode and longitude and latitude (9).
<h3>Provision the logic App</h3>
At this point we have worked out the high-level flow and logic and as such we can now go-ahead and create the logic app, so let’s go ahead and do so

1. Login to the Azure Portal

2. Select the resource-group which you created in part-1, in which you deployed your custom wcf service. In my case this resource-group is called Demos

3. Once the resource-group blade is visible, click on the Add button

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0031.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image003[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0031_thumb.png" alt="clip_image003[1]" width="244" height="91" border="0" /></a>

4. A new blade will popup, within this blade search for Logic App and click on the Logic App artefact published by Microsoft and of the Category Web + Mobile

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0041.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image004[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0041_thumb.png" alt="clip_image004[1]" width="244" height="81" border="0" /></a>

5. Click on create

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0051.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image005[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0051_thumb.png" alt="clip_image005[1]" width="167" height="244" border="0" /></a>

6. Now fill out the details and once done click Create, after which your logic app will be created

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0061.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image006[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0061_thumb.png" alt="clip_image006[1]" width="89" height="244" border="0" /></a>

7. Once the logic app has been created, open it and you should be presented with a screen which allows you to create a new logic app using one of the pre-build templates. In our case we will choose the “Blank LogicApp”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image008.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image008" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image008_thumb.jpg" alt="clip_image008" width="244" height="167" border="0" /></a>
<h3>Implement the ‘Blank LogicApp’</h3>
Once you’ve clicked on the blank logic app template, the designer will pop up. We will be using this designer to develop the below depicted flow which will be explained in the following sections. Well let’s get started.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image010.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image010" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image010_thumb.jpg" alt="clip_image010" width="244" height="153" border="0" /></a>
<h4>Step 1: Request Trigger</h4>
Within this designer, you will be presented with a ‘card selector’. This card selector, being the first of many, contains so-called triggers. These triggers can best be explained as ‘event listeners’ which indicate when a logic app is to be instantiated. <b></b>

In our scenario, we want to trigger our logic app by means of sending a request. So, in our case we would select the Request trigger. Now select this Request Trigger.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image012.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image012" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image012_thumb.jpg" alt="clip_image012" width="244" height="124" border="0" /></a>

To dig up more information regarding the different triggers and actions you can click on the Help button, which will open up a Quick Start Guide blade containing links too more information.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0141.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image014[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0141_thumb.png" alt="clip_image014[1]" width="244" height="142" border="0" /></a>
<h5>Configure</h5>
Once you’ve selected the trigger, the Request Trigger ‘Card’ will be expanded and will allow you to configure this trigger.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0151.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image015[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0151_thumb.png" alt="clip_image015[1]" width="244" height="236" border="0" /></a>

1. This section is not customizable, but once the logic app is saved will contain the generated endpoint. This endpoint is to be used by clients who which to invoke the logic app.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0171.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image017[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0171_thumb.png" alt="clip_image017[1]" width="244" height="72" border="0" /></a>

2. The request body JSON schema section, is an optional section, which allows us to add a schema describing what the inbound request message should look like.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image019.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image019" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image019_thumb.png" alt="clip_image019" width="244" height="127" border="0" /></a>

You might be wondering why bother? Well if we bother by adding a schema we get the benefit of an ‘intellisense like’ experience from within the designer, which can help us down the road in case we want to easily access one of the properties of the request message in a follow up action.

So let’s go ahead and add a schema. In our case, we will only require one property to be send to our logic-app and this property is RequestId. We will be using the property further down the stream to uniquely identify the request and use it to store our ‘session state’.

As such our Json request can be represented as follows:

{

"RequestId":"2245775543466"

}

Now that we know what the payload message looks like, we need to derive the Json schema. Well luckily for us, we can go to <a href="http://jsonschema.net/">JSONSchema.net </a>and generate a schema. J The generated schema, subsequently would be represented as

{

"type": "object",

"properties": {

"RequestIds": {

"type": "string"

}

},

"required": [

"RequestIds"

]

}

At this point we have all the information required to fill out the ‘Request Body JSON Schema’ section, so all we have to do is copy and paste it into that section.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image020.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image020" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image020_thumb.png" alt="clip_image020" width="244" height="114" border="0" /></a>

3. At this point we are ready to proceed with our next step. Which according to our high-level design consists of an activity which composes a new message, which represents the request message (soap) which is to be send to the customer WCF service.

So, let’s proceed and click on the + New Step button
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image021.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image021" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image021_thumb.png" alt="clip_image021" width="244" height="60" border="0" /></a>

4. Now several options appear, but we are currently only interested in the option ‘Add an action’, so select this.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0221.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image022[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0221_thumb.png" alt="clip_image022[1]" width="244" height="108" border="0" /></a>
<h4>Step 2: Compose SOAP request message</h4>
As part of our last step we clicked on the “new step” button and selected “Add an action”. Which subsequently would display the ‘card selector’ again, only this time displaying available actions to choose from.

Please note: typical actions to choose from would include

· connectors to SaaS services such as Dynamics CRM Online, on premise hosted Line of business applications such as SAP and connectors to existing logic-apps, azure functions and API’s hosted in API Management

· typical workflow actions which allow us to delay processing or even allow us to terminate further processing.

Looking back at our overall scenario which we are about to implement one of the initial actions would be retrieving a list of customers.

In order to retrieve this list of customers we would need to invoke our Customer WCF service, we build earlier. As our WCF service is SOAP based, it requires us to implement one additional step before we can actually invoke the service from within Logic Apps and this steps involves creating the SOAP request message, using a Compose Action.

So from within the ‘Card Selector’ select the compose Action.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0231.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image023[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0231_thumb.png" alt="clip_image023[1]" width="244" height="234" border="0" /></a>

Please note: In the near future this additional step will no longer be required as API Management will be able to RESTify your soap endpoints which than can easily consumed from within logicapps (see <a href="https://trello.com/b/FAA147vS/azure-api-management-product-roadmap">roadmap</a>). Besides having functionality in API Management, the chances are pretty good as well that a first-class SOAP connector will be added to logic apps in the future as it is ranked high on the logic apps functionality <a href="https://feedback.azure.com/forums/287593-logic-apps">wishlist</a>.
<h5>Configure</h5>
Once you’ve selected the compose action the following ‘Card’ will show up on in the designer which allows you to compose a Message, which in our case will be the SOAP Request message.

1. The input section allows us to construct the soap (xml) message, which will act as the request which we will be sending to our customer WCF service.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0241.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image024[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0241_thumb.png" alt="clip_image024[1]" width="244" height="91" border="0" /></a>

So how would you determine what this message would look like. Well the easiest way would be by using a tool such as SOAPUI which can generate a sample request message. In the <a href="https://blog.brauwers.nl/2016-11-30/no-more-excuses-protect-your-azure-soaprest-api-always-end-to-end-scenario-part-1/">previous post</a>, I’ve added a section which explains how to do this and in our scenario the soap request message looks as follow:

&lt;?xml version="1.0" encoding="UTF-8"?&gt;

&lt;Envelope xmlns="http://schemas.xmlsoap.org/soap/envelope/"&gt;

&lt;Body&gt;

&lt;GetCustomers xmlns="http://tempuri.org/" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" /&gt;

&lt;/Body&gt;

&lt;/Envelope&gt;

2. Once we have our sample SOAP request message, we simply copy and paste it into the input field.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image025.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image025" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image025_thumb.png" alt="clip_image025" width="244" height="125" border="0" /></a>

Please note; once you click on the Inputs section a windows will appear which will allow you to select ‘dynamic content, used within this flow’. This is the ‘intellisense like’ experience I referred to earlier in this post. Anyways we will be ignoring this for now, but in future steps we will be using this.

3. At this point we are ready to proceed with our next step. Which will actually call our customer WCF service.

So, let’s proceed and click on the + New Step button
<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0211.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image021[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0211_thumb.png" alt="clip_image021[1]" width="244" height="60" border="0" /></a>

4. Once again several options appear and once again select the option ‘Add an action’.<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0222.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image022[2]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0222_thumb.png" alt="clip_image022[2]" width="244" height="108" border="0" /></a>
<h4>Step 3: Invoke our Customer WCF Service</h4>
After completing step 2 we are now able to actually implement calling our customer WCF service. In order to do so all, we need to do is select the ‘HTTP’ Action from within the ‘Card Selector’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0261.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image026[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0261_thumb.png" alt="clip_image026[1]" width="244" height="234" border="0" /></a>
<h5>Configure</h5>
Once you’ve selected the HTTP action the following ‘Card’ will show up on in the designer which allows you to configure the HTTP request in order to receive the customer information.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0271.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image027[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0271_thumb.png" alt="clip_image027[1]" width="244" height="203" border="0" /></a>

As you might remember the custom WCF Service which we are about to invoke uses URL Authorization using Azure Active Directory (see <a href="https://blog.brauwers.nl/2016-11-30/no-more-excuses-protect-your-azure-soaprest-api-always-end-to-end-scenario-part-1/">previous post</a>) and as such requires any (POST) request to be authenticated. Long story short; One of the nice things of the HTTP action is that it makes it a breeze invoking web-services even if they require authentication, all we need to do is configure the action correctly and this is done by expanding the advanced options of the HTTP Card, which will allow us to do so.

1. The Method which we need to select is ‘POST’ as we will be posting the soap request to the customer WCF service.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0281.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image028[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0281_thumb.png" alt="clip_image028[1]" width="244" height="56" border="0" /></a>

2. The Uri sections allows us to enter the Request URL of the web-service. In our case that would be <a href="https://demo-apis.azurewebsites.net/Customers.svc">https://demo-apis.azurewebsites.net/Customers.svc</a>

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image029.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image029" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image029_thumb.png" alt="clip_image029" width="244" height="51" border="0" /></a>

3. The Headers sections will be used to add both the SOAP Action which needs to be invoked as well as the Content-Type of the actual request message.

The easiest way to retrieve the SOAP Action would be by means of SOAPUI as well. So from within SOAPUI open the request and then select WS-A (bottom menu-bar), and then copy and paste the Action

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0301.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image030[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0301_thumb.png" alt="clip_image030[1]" width="244" height="130" border="0" /></a>

The header information needs to be passed in as a Json string, and looks as follows

{

"Content-Type":"text/xml",

"SOAPAction":"http://tempuri.org/ICustomers/GetCustomers"

}

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0311.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image031[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0311_thumb.png" alt="clip_image031[1]" width="244" height="93" border="0" /></a>

4. The body section will contain the message which we composed in the previous step. As such once you click in this section, additional information will displayed on the desiger which allows you to select ‘dynamic content’. (this is the ‘intellisense like’ experience I referred to earlier). From this menu, select the variable ‘’ This variable contains the message which we composed earlier.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0321.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image032[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0321_thumb.png" alt="clip_image032[1]" width="244" height="187" border="0" /></a>

5. Now click on the Show Advanced Options, which will allow us to fill out the required authentication information.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0331.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image033[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0331_thumb.png" alt="clip_image033[1]" width="244" height="78" border="0" /></a>

6. From the dropdown select Active Directory OAuth

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0341.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image034[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0341_thumb.png" alt="clip_image034[1]" width="244" height="169" border="0" /></a>

7. For Active Directory OAuth we will require to fill out the Tenant, Audience, Client ID and Secret. This information is to be retrieved as follows

a. In the Azure Portal, go to Azure Active Directory Blade and click on APP Registrations

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image036.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image036" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image036_thumb.jpg" alt="clip_image036" width="228" height="244" border="0" /></a>

b. Select the application in question (see previous blog-post) which you registered for the WCF Customer service. In my case demo-apis

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image037.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image037" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image037_thumb.png" alt="clip_image037" width="244" height="215" border="0" /></a>

c. Now on the settings blade click on Properties and make a note of the following:
<b>Application ID</b> – This is the equivalent of the <b>Client ID</b>

<b>App ID Uri </b>– This is the equivalent of the <b>Audience</b>

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0381.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image038[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0381_thumb.png" alt="clip_image038[1]" width="244" height="126" border="0" /></a>

d. GO back to the settings blade, click on Keys

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image040.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image040" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image040_thumb.jpg" alt="clip_image040" width="244" height="113" border="0" /></a>

e. Now it is time to generate the <b>secret</b>. In order to do this, add a description and select how long the secret should be valid. Once done save the entry and make a note of the value (this is the <b>secret</b>)

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0421.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image042[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0421_thumb.jpg" alt="clip_image042[1]" width="244" height="63" border="0" /></a>

f. Now on the portal page, click on the Help Icon and select ‘Show diagnostics’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image043.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image043" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image043_thumb.png" alt="clip_image043" width="244" height="152" border="0" /></a>

g. In the window, which pops up, search for tenants. Find your tenant (most likely the one which states ‘isSignedInTenant = true’ and note down the <b>Tenant ID</b>

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image044.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image044" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image044_thumb.png" alt="clip_image044" width="244" height="61" border="0" /></a>

h. At this point we have all the information in order to fill out the required information

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image046.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image046" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image046_thumb.png" alt="clip_image046" width="229" height="244" border="0" /></a>
<h5>Test</h5>
Now that we’ve implemented the call, it would be a good time to go ahead and test the logic app. Luckily for us, this is quite simple.

1. Click on the Save button to save your logic app

2. Now Click on the run button.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image048.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image048" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image048_thumb.jpg" alt="clip_image048" width="244" height="112" border="0" /></a>

3. Wait a few seconds and you should see a debug output. If everything went Ok, it should look similar to the image below.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0501.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image050[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0501_thumb.jpg" alt="clip_image050[1]" width="244" height="93" border="0" /></a>

4. Now click on the HTTP – GetCustomers shape. Which allows you to look at the debug / tracking information. It will show you the input as well as the output information.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image051.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image051" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image051_thumb.png" alt="clip_image051" width="111" height="244" border="0" /></a>

5. Now go to the OUTPUTS section and copy and paste the Body section. We will be needing this in Step 4 J

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image052.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image052" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image052_thumb.png" alt="clip_image052" width="226" height="244" border="0" /></a>
<h4>Step 4: Loop over the customer result</h4>
Our last step resulted in the fact that we configured our HTTP Action which was responsible for invoking our customer wcf service and returning us a list of customers.

Now in this step we will need to loop over the returned customer list, such that we can enrich each individual record with localization information obtained from a different API.

In order to do so we will have to select a for-each action. This action can be selected by clicking on the “+ New Step button”. Several options will appear of which we need to select the ‘more’ followed with the ‘add a for each’ action.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image053.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image053" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image053_thumb.png" alt="clip_image053" width="244" height="134" border="0" /></a>
<h5>Configure</h5>
1. Once the for-each step has been selected it is being dropped on the designer. The designer than offers us a section in which we can add in input over which we want to loop.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image054.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image054" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image054_thumb.png" alt="clip_image054" width="244" height="130" border="0" /></a>

2. If our WCF service would have returned an Json Array object, we would have been able to simply select this output using the ‘Dynamic Content’ selection process (aka intellisense). However in our case the output over which we want to loop is a customer resultset formatted in XML. So, in our case we will need to help the the logic-apps engine a bit, and they way to do this, is by adding a custom expression. Which in our case is a <a href="https://docs.microsoft.com/en-us/rest/api/logic/definition-language#expressions">Xpath expression</a>, pointing to the node over which we want to loop.

The xpath expression in our case would be:

/*[local-name()="Envelope"]/*[local-name()="Body"]/*[local-name()="GetCustomersResponse"]/*[local-name()="GetCustomersResult"]/*

Easiest way to test this xpath expression, would be by using the response message we extracted when we tested our logic app earlier and subsequently use an online tool <a href="http://www.freeformatter.com/xpath-tester.html">to test the xpath expression</a>.

Now that we have our xpath expression, we can use it in the following Logic App Expression

@xpath(xml(body(‘<b>Replace with the name of action of which we want to use the response’</b>,’<b>Xpath Expression</b>’)

In my scenario the expression would be as follows

@xpath(xml(body('HTTP_-_GetCustomers')), '/*[local-name()="Envelope"]/*[local-name()="Body"]/*[local-name()="GetCustomersResponse"]/*[local-name()="GetCustomersResult"]/*')

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0561.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image056[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0561_thumb.jpg" alt="clip_image056[1]" width="244" height="52" border="0" /></a>
<h4>Step 5: Extract individual customer info</h4>
In our previous step we instantiated our for-each loop which will loop over our xml result set. Now our next step is to extract the individual customer info and store it in a intermediate json format which we will be using in subsequent actions.

So from within our for-each action, select the Add an action.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image058.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image058" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image058_thumb.jpg" alt="clip_image058" width="244" height="52" border="0" /></a>

From within the ‘Card Selector’ select the compose Action.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0232.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image023[2]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0232_thumb.png" alt="clip_image023[2]" width="244" height="234" border="0" /></a>
<h5>Configure</h5>
Once you’ve selected the compose action the following ‘Card’ will show up on in the designer which allows you to compose a Message, which in our case will be a custom Json message which holds the individual customer information consisting of CustomerId, FirstName, LastName and PostCode

Note: As in Step 4 when configuring the for-each iteration path. We will be leveraging xpath expressions in order to extract the individual customer data. Alternatively, I could have leveraged an Azure Function to convert the received XML Customer response into JSON or I could have leveraged API Management which by means of policies can perform conversion from xml to json out of the box. In my next post (part 3 of this series) I will be using this.

.

1. The input section allows us to construct our custom Json message which holds the individual customer information consisting of CustomerId, FirstName, LastName and PostCode

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0242.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image024[2]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0242_thumb.png" alt="clip_image024[2]" width="244" height="91" border="0" /></a>

2. In order to extract the required fields from the xml will be leveraging the following xpath queries

<b>a. </b><b>customerId extraction:</b>

string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"CustomerId\"])')

<b>b. </b><b>FirstName extraction:</b>

string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"FirstName \"])')

<b>c. </b><b>SurName extraction:</b>

string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"SurName\"])')

<b>d. </b><b>PostCode extraction:</b>

string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"PostCode\"])')

the logic app expression which we will be leveraging to extract a value using xpath will be

@{xpath(xml(decodeBase64(item().$content)), '<b>Xpath Expression</b>') where item() refers to the current item (customer record) in the loop and $content represents the content (customer record xml part)

Combined in a Json construct the complete message construction would look like (note that we escape using \)

{

"CustomerId": "@{xpath(xml(decodeBase64(item().$content)), 'string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"CustomerId\"])')}",

"FirstName": "@{xpath(xml(decodeBase64(item().$content)), 'string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"FirstName\"])')}",

"LastName": "@{xpath(xml(decodeBase64(item().$content)), 'string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"SurName\"])')}",

"PostCode": "@{xpath(xml(decodeBase64(item().$content)), 'string(/*[local-name()=\"CustomerData\"]/*[local-name()=\"PostCode\"])')}"

}

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image059.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image059" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image059_thumb.png" alt="clip_image059" width="237" height="244" border="0" /></a>
<h5>Test</h5>
Now that we’ve implemented the xml extraction within the for-each, it would be a good time to go ahead and test the logic app, and see if everything works accordingly.

1. Click on the Save button to save your logic app

2. Now Click on the run button.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0481.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image048[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0481_thumb.jpg" alt="clip_image048[1]" width="244" height="112" border="0" /></a>

3. Wait a few seconds and you should see a debug output. If everything went Ok, it should look similar to the image below.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0601.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image060[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0601_thumb.png" alt="clip_image060[1]" width="190" height="244" border="0" /></a>

4. As you can see the last item in the flow, contains a Json output depicting the customer values extracted.
<h4>Step 6: Invoke the postcodeapi</h4>
Now that we have extracted our customer data and stored it in a json format. We can proceed with the next step, which invokes invoking a public postcode api. In order to do so we will once again select the HTTP Action within the ‘Card Selector’

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0262.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image026[2]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0262_thumb.png" alt="clip_image026[2]" width="244" height="234" border="0" /></a>
<h5>Configure</h5>
Once you’ve selected the HTTP action the following ‘Card’ will show up on in the designer which allows you to configure the HTTP request in order to receive localization information based on a postal code.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0272.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image027[2]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0272_thumb.png" alt="clip_image027[2]" width="244" height="203" border="0" /></a>

1. The Method which we need to select is ‘GET as we will be retrieving data from a rest endpoint.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0611.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image061[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0611_thumb.png" alt="clip_image061[1]" width="244" height="79" border="0" /></a>

2. The Uri sections allows us to enter the Request URL of the web-service. In our case that would be <a href="http://v0.postcodeapi.com.au/suburbs.json?postcode=XXXXX">http://v0.postcodeapi.com.au/suburbs.json?postcode=XXXXX</a> where XXXX is a dynamic parameter, to be more specific; we will be using the PostCode field which we extracted in step 5. In order to use this PostCode value we will

a. Enter the value <a href="http://v0.postcodeapi.com.au/suburbs.json?postcode=">http://v0.postcodeapi.com.au/suburbs.json?postcode=</a> in the Uri field.

b. Select the dynamic content ‘Outputs’ from the Extracted xml

We are currently not able to directly access the PostCode field from within the designer as the designer currently is not aware of this property. It is only aware of the fact that the ‘compose step – Extracted xml’ has a output which is a ‘message’ and as such we can only select the complete message.

Note: In a future release of logic-apps this experience will be improved and additional magic will be added such that the designer can ‘auto-discover’ these message properties. How this will be implemented is not 100% clear, but one of the possibilities would be; that we would manually add a ‘description’ of the output (Json schema, for example) to the compose action or any other action which returns / creates an object.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image062.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image062" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image062_thumb.png" alt="clip_image062" width="244" height="79" border="0" /></a>

3. In order to select the PostCode field from the Outputs, we will be needing to switch to Code View.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image064.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image064" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image064_thumb.png" alt="clip_image064" width="244" height="38" border="0" /></a>

4. Once in code view, find the code block which contains the <a href="http://v0.postcodeapi.com.au/suburbs.json?postcode=">http://v0.postcodeapi.com.au/suburbs.json?postcode=</a> url. Once found we simple modify the code from
<a href="http://v0.postcodeapi.com.au/suburbs.json?postcode=@%7boutputs('Extracted_xml')%7d">http://v0.postcodeapi.com.au/suburbs.json?postcode=@{outputs('Extracted_xml')}</a>

to

<a href="http://v0.postcodeapi.com.au/suburbs.json?postcode=@%7boutputs('Extracted_xml').PostCode%7d">http://v0.postcodeapi.com.au/suburbs.json?postcode=@{outputs('Extracted_xml').PostCode}</a>

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image066.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image066" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image066_thumb.jpg" alt="clip_image066" width="244" height="108" border="0" /></a>

5. Now go back to the designer

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0681.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image068[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0681_thumb.png" alt="clip_image068[1]" width="244" height="30" border="0" /></a>

6. And behold the designer now states “http://v0.postcodeapi.com.au/suburbs.json?postcode={} PostCode”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image070.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image070" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image070_thumb.png" alt="clip_image070" width="244" height="61" border="0" /></a>
<h5>Test</h5>
Now that we’ve implemented the postcode api call, it would be a good time to go ahead and test the logic app.

1. Click on the Save button to save your logic app

2. Now Click on the run button.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0711.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image071[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0711_thumb.jpg" alt="clip_image071[1]" width="244" height="112" border="0" /></a>

3. Wait a few seconds and you should see a debug output. If everything went Ok, it should look similar to the image below. If you expand the HTTP action, you wull notice that the URI now is composed using the extracted PostCode value

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0721.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image072[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0721_thumb.png" alt="clip_image072[1]" width="122" height="244" border="0" /></a>
<h4>Step 7: Compose an enriched customer message</h4>
Now that we have invoked the postcode API it is time to combine both the original customer data and the postcode data. In order to do this, we will be composing a new Json message using the Compose Action.

From within the ‘Card Selector’ select the compose Action.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0233.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image023[3]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0233_thumb.png" alt="clip_image023[3]" width="244" height="234" border="0" /></a>
<h5>Configure</h5>
Once you’ve selected the compose action the following ‘Card’ will show up on in the designer which allows you to compose a Message, which in our case will be a new Json message which holds both the customer data as well as the location data retrieved from the PostCode lookup.

1. The input section allows us to construct our custom Json message which will hold all the combined data

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0243.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image024[3]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0243_thumb.png" alt="clip_image024[3]" width="244" height="91" border="0" /></a>

2. Now copy and paste the below ‘sample’ json message into the input section

This message will be of the following structure:

{

"FirstName": "## FirstName from the WCF Customer web service##",

"LastName": "## LastName from the WCF Customer web service##",

"Location": {

"Latitude": "## Latitude obtained from the postal api##",

"Locality": "## Locality obtained from the postal api##",

"Longitude": "## Longitude obtained from the postal api##",

"PostCode": "# PostCode from the 'extract_xml' message##",

"State": "## State obtained from the postal api##",

"Suburb": "## Suburb obtained from the postal api##"

},

"RequestId": "## Obtained from the request trigger##",

"id": "# CustomerId from the 'extract_xml' message##"

}

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image073.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image073" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image073_thumb.png" alt="clip_image073" width="188" height="244" border="0" /></a>

3. Now go to code view

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image074.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image074" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image074_thumb.png" alt="clip_image074" width="244" height="38" border="0" /></a>

4. Once in code view, find the code block which represents the Json message which we just copied and pasted in the input section.

Note: In a future release of logic-apps this experience will be improved and additional magic will be added such that the designer can ‘auto-discover’ these message properties, which we will now add manually. How this will be implemented is not 100% clear, but one of the possibilities would be; that we would manually add a ‘description’ of the output (Json schema, for example) to the compose action or any other action which returns / creates an object.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image076.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image076" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image076_thumb.png" alt="clip_image076" width="244" height="174" border="0" /></a>

5. Now replace the json such that it looks like depicted below

"Enrich_with_postal_code": {

"inputs": {

"FirstName": "@{outputs('Extracted_xml').FirstName}",

"LastName": "@{outputs('Extracted_xml').LastName}",

"Location": {

"Latitude": "@{body('HTTP')[0].latitude}",

"Locality": "@{body('HTTP')[0].locality}",

"Longitude": "@{body('HTTP')[0].longitude}",

"PostCode": "@{outputs('Extracted_xml').PostCode}",

"State": "@{body('HTTP')[0].state.name}",

"Suburb": "@{body('HTTP')[0].name}"

},

"RequestId": "@{triggerBody()['RequestId']}",

"id": "@{outputs('Extracted_xml').CustomerId}"

},

"runAfter": {

"HTTP": [

"Succeeded"

]

},

"type": "Compose"

},

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image078.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image078" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image078_thumb.png" alt="clip_image078" width="244" height="143" border="0" /></a>
<b></b>
<h5>Test</h5>
Now that we’ve composed a message containing both the WCF and PostCode API data, it would be another good time to go ahead and test if everything works and this time we will be testing our logic app using Fiddler

1. Download <a href="http://www.telerik.com/fiddler">Fiddler</a>, if you already have.

2. Go to you logic app and expand the Request Trigger and press on the “Copy Icon”, this will copy the logic app endpoint to your clipboard.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image079.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image079" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image079_thumb.png" alt="clip_image079" width="244" height="181" border="0" /></a>

3. Open fiddler, and select the composer tab

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image080.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image080" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image080_thumb.png" alt="clip_image080" width="244" height="48" border="0" /></a>

4. In the composer

a. Set the HTTP Action to POST

b. Copy and Paste the uri in the Uri field

c. In the header section add

i. Content-Type:application/json

d. In the body section add the following json

{

"RequestId":"20161220"

}

e. Click on the Execute button

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image082.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image082" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image082_thumb.png" alt="clip_image082" width="244" height="88" border="0" /></a>

5. Now go back to your Logic App

6. In the run history, select the last entry

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image084.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image084" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image084_thumb.png" alt="clip_image084" width="244" height="73" border="0" /></a>

7. If everything went Ok, it should look similar to the image below.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image085.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image085" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image085_thumb.png" alt="clip_image085" width="198" height="244" border="0" /></a>
<h4>Step 8: Store Session State in DocumentDB</h4>
At this point we have implemented functionality which

· allows us to iterate over all the customer records

· retrieve localization data from the postal code api using the postal code extracted from the customer record.

· Compose a new message which contains all the data.

The functionality which is left to implement at this point in time consists of; combining all the composed new messages, containing the customer and localization data, in one document and returning it to the caller.

Note: Currently there is no easy way of doing this directly from within logic-apps as logic apps currently does not contain the functionality which would allow us to ‘combine the data’ in memory. But have no fear; in one of the next releases of Logic Apps will have support for storing session state and once this is available we will no longer require this additional step, which is explained below.
<h5>Configure</h5>
As Logic Apps currently has no means of storing session state, we will be resorting to an external session state store. In our case, the most obvious choice would be DocumentDB.

So before we proceed, let’s go and create a DocumentDB service.

1. Go to the Azure Portal and click on the New Icon

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image086.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image086" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image086_thumb.png" alt="clip_image086" width="179" height="103" border="0" /></a>

2. Search for DocumentDB

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image088.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image088" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image088_thumb.jpg" alt="clip_image088" width="244" height="141" border="0" /></a>

3. Select DocumentDB from Publisher Microsoft

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image090.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image090" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image090_thumb.jpg" alt="clip_image090" width="244" height="43" border="0" /></a>

4. Fill out the required information and once done create the DocumentDB instance

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image092.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image092" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image092_thumb.jpg" alt="clip_image092" width="184" height="244" border="0" /></a>

5. After creation has completed, open the DocumentDB Instance.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image094.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image094" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image094_thumb.png" alt="clip_image094" width="244" height="156" border="0" /></a>

6. Now Add a Collection

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image096.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image096" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image096_thumb.jpg" alt="clip_image096" width="244" height="28" border="0" /></a>

7. Fill out the required information for the Collection Creation, and press OK once done

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image098.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image098" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image098_thumb.jpg" alt="clip_image098" width="244" height="164" border="0" /></a>

8. Go back to the main DocumentDB Blade, and click on Keys

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image100.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image100" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image100_thumb.jpg" alt="clip_image100" width="244" height="225" border="0" /></a>

9. From within the Keys, Copy and Paste the Primary or Secondary Key

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image102.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image102" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image102_thumb.jpg" alt="clip_image102" width="244" height="119" border="0" /></a>

10. Now go back to our logic app, and open it in the designer

11. In the Logic App, Click on the Add New Item

12. Now search for DocumentDB Actions and select “Azure DocumentDB – Create or update document”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image103.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image103" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image103_thumb.png" alt="clip_image103" width="244" height="206" border="0" /></a>

13. The connector will now be displayed and will require some configuration

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image104.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image104" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image104_thumb.png" alt="clip_image104" width="244" height="163" border="0" /></a>

14. Fill out the required information. For which it has to be noted that for Database Account Nam is the actual name of the documentDB. In my case docdb-playground.<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image106.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image106" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image106_thumb.png" alt="clip_image106" width="244" height="71" border="0" /></a>

15. Once filled out the information should look similar to the one depicted below in the image

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image107.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image107" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image107_thumb.png" alt="clip_image107" width="244" height="159" border="0" /></a>

16. At this point the connection has been created, and we can now proceed with the actual configuration in which we will

a. select the correct Database ID from the dropdown

b. select the collection to use

c. add the dynamic content (message) which we want to store

d. we set the value to True for IsUpsert

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image109.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image109" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image109_thumb.jpg" alt="clip_image109" width="244" height="154" border="0" /></a>
<h4>Step 9: Exit Loop, Retrieve and return Stored Data</h4>
Our last step resulted in the fact that we persisted all documents into DocumentDB. Now before we proceed, let’s have a look at Step 7 in which we composed the following message, which eventually was stored in DocumentDB.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image111.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image111" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image111_thumb.jpg" alt="clip_image111" width="244" height="238" border="0" /></a>

Have a good look at the field: RequestId. This field is actually passed in whenever we invoke our LogicApp. (see step 7, the test section).

There was a reason why we added this field and have it stored in DocumentDB. The reason? Well this way we are able to select all documents stored in DocumentDB belonging to the specific ID of the current Request and return them to the caller.
<h5>Configure</h5>
1. Select the Add an action button located just below the for-each scope.

2. Now search for DocumentDB Actions and select “Azure DocumentDB – Query documents”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image112.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image112" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image112_thumb.png" alt="clip_image112" width="244" height="206" border="0" /></a>

3. The Document DB Query Documents connector, can now be configured as follows

a. Select the correct database ID from the dropdown in our case ProcessingState

b. Select the applicable collection from the dropdown in our case LogicApp

c. Now add a query, which will return all documents stored in the collection which have the same request id.

SELECT c.Id as CustomerId, c.FirstName,c.LastName,c.Location FROM c where c.RequestId = …..<b></b>

d. Where c.RequestId = “ SELECT REQUEST ID from the Dynamic Content window”

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image114.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image114" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image114_thumb.png" alt="clip_image114" width="244" height="164" border="0" /></a>

4. At this point we have completed the action which will retrieve all the applicable stored documents. So the only thing which is left to do is, returning this list of document back to the caller. In order to do this, we add one more action. This action is called Response

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image115.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image115" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image115_thumb.png" alt="clip_image115" width="244" height="168" border="0" /></a>

5. The Response action, can can now be configured as follows

a. Enter 200 for the return status code, this indicates the HTTP Status code ‘OK’

b. In the response header we will need to set the content-type. We will do this by adding the following piece of json

{ “Content-Type”:”application/json” }

c. In the body we will add the dynamic content which relates to documents which were returned from document DB

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image117.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image117" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image117_thumb.jpg" alt="clip_image117" width="244" height="170" border="0" /></a>
<h5>Test</h5>
Well now that we have implemented the complete flow, it is time to do our final test and once again we will be using <a href="http://www.telerik.com/fiddler">Fiddler</a> to perform this test.

1. Open fiddler, and select the composer tab

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0801.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image080[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0801_thumb.png" alt="clip_image080[1]" width="244" height="48" border="0" /></a>

2. In the composer

a. Set the HTTP Action to POST

b. Copy and Paste the uri in the Uri field

c. In the header section add

i. Content-Type:application/json

d. In the body section add the following json

{

"RequestId":"20161221"

}

e. Click on the Execute button

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0821.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image082[1]" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image0821_thumb.png" alt="clip_image082[1]" width="244" height="88" border="0" /></a>

3. Now open the result and you should see a response similar to the one below

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image119.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image119" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image119_thumb.jpg" alt="clip_image119" width="244" height="128" border="0" /></a>

4. No go back to your logic app and in the run history, select the last entry

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image120.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; margin: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image120" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image120_thumb.png" alt="clip_image120" width="244" height="89" border="0" /></a>

5. If everything went Ok, it should look similar to the image below.

<a href="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image122.jpg"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image122" src="https://brauwersnl.blob.core.windows.net/images/uploads/2017/02/clip_image122_thumb.jpg" alt="clip_image122" width="187" height="244" border="0" /></a>
<h3>Conclusion</h3>
This post has guided you through setting up a logic app which calls two api’s a, combines the data and returns the aggregated result back to the caller.

In my next post I will introduce API Management into the mix which will be using to expose the two api mentioned and apply some api management magic which further simplify our logic app implementation.

So until next time, stay tuned.

Cheers

René