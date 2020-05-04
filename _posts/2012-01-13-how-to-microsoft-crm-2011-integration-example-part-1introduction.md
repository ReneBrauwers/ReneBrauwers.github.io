---
ID: 62
post_title: 'How to: Microsoft CRM 2011 Integration example (Part 1–Introduction)'
post_name: >
  how-to-microsoft-crm-2011-integration-example-part-1introduction
author: Rene Brauwers
post_date: 2012-01-13 17:06:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2012/01/13/how-to-microsoft-crm-2011-integration-example-part-1introduction/
published: true
tags:
  - BizTalk
  - BizTalk 2010
  - 'C#'
  - CRM
  - CRM2011
  - Custom Activity
  - Entities
  - How to
  - Introduction
  - SDK
  - Trigger
  - WCF
categories:
  - BizTalk
  - 'C#'
---
Well it has been a while since my last post; however as I stated in my first <a href="https://blog.brauwers.nl/2011-03-27-back-into-business/" target="_blank" rel="noopener noreferrer">post</a>. “I’ll only try to blog whenever I have something which in my opinion adds value”, and well the topic I want to discuss today might just add that additional value.

&nbsp;

Please note: This post will supply you with background information, the actual implementation of the solution will be covered in the next blog posts. However the sample files which are mentioned in this post can already be downloaded.

&nbsp;
<h2>Scenario sketch</h2>
Let’s  say one of your customer’s are considering to replace their current CRM with Microsoft CRM2011.

&nbsp;

Now one of the company’s business processes dictates that whenever a new customer or contact has been added to their CRM system,  this data has to be send to their ERP system near-real-time. This customer or contact is then added into to ERP system and is assigned an unique account number. This account number then needs to be send back to the CRM system. As an end result the corresponding customer in CRM2011 is updated with the account number from the ERP system.

&nbsp;

Their current CRM solution already takes care of this functionality however this has been implemented using a point-to-point solution and therefore replacing their current CRM with Microsoft CRM2011 would break this ‘integration-point’.  The customer is aware that in the long-term it would be best to move away from these kind of point-to-point solutions and move more to a Service Oriented Architecture.

&nbsp;

At the end of the day it is up to you to convince your customer that it is no problem at all with Microsoft CRM2011 to setup a solution which includes an integration with their ERP system  and as you are aware of the fact that the customer wants to move to a Service Oriented Architecture, you see the opportunity fit to introduce the company to BizTalk Server 2010 as well.

&nbsp;

So eventually you propose the following Proof of Concept scenario to your customer: ‘You will show to the customer that it is possible with almost no effort to build a solution which connects Microsoft CRM 2011 to their ERP system, whilst adhering to the general known Service Oriented Architecture principles’; once you tell your customer that this POC does not involve any costs for them except time and cooperation; they are more than happy and agree to it.

&nbsp;

&nbsp;
<h2>Preparing your dish</h2>
In order to complete the solution discussed in this blog post you will need the following ingredients:

&nbsp;

A test environment consisting of:
<ul>
	<li>1 Windows Server 2008R2 which acts as Domain Server (Active Directory)</li>
	<li>1 Windows Server 2008R2 on which Microsoft CRM2011 is installed and configured</li>
	<li>1 Windows Server 2008R2 on which Microsoft BizTalk Server 2010 is installed and configured.</li>
	<li>One Development Machine with Visual Studio 2010 installed</li>
</ul>
&nbsp;
<h3>Step 1: How do I get data out of Microsoft CRM2011?</h3>
Well in order to get data (let me rephrase; an entity) out of CRM for our Integration scenario we will need to build a custom activity which can be added as a workflow step within CRM2011.

&nbsp;

So which ingredients are required to do this?
<ul>
	<li>We need to download the CRM2011 SDK; so go and fetch it <a href="http://www.microsoft.com/download/en/details.aspx?id=24004" target="_blank" rel="noopener noreferrer">here</a></li>
</ul>
&nbsp;

So what are we going to build?
<ul>
	<li>We will build a custom activity and deploy it to CRM2011 such that it can be used in a workflow, or <a href="http://www.brauwers.nl/downloads/Demo.Crm2011.zip" target="_blank" rel="noopener noreferrer">download</a> my example and install it</li>
</ul>
&nbsp;

&nbsp;
<h3>Step 2: How do I get data in my custom ERP?</h3>
Well for this step I’ve build a custom application which will act as a stub for our custom ERP. This custom ERP system will be exposed by means of a WCF service.

&nbsp;

So which ingredients are required to do this?
<ul>
	<li>An (sample) ERP system.</li>
</ul>
&nbsp;

So what are we going to build?
<ul>
	<li>Well you could build your own application, or <a href="http://www.brauwers.nl/downloads/Demo.Erp.zip" target="_blank" rel="noopener noreferrer">download</a> my example and install it.</li>
</ul>
&nbsp;
<h3>Step 3: How do I get data into CRM2011?</h3>
Well in order to get data into CRM; we will use the out of the box web services which are exposed by CRM2011.

&nbsp;

So which ingredients are required to do this?
<ul>
	<li>Well if you have not yet downloaded the CRM2011 SDK; go and fetch it <a href="http://www.microsoft.com/download/en/details.aspx?id=24004" target="_blank" rel="noopener noreferrer">here</a></li>
</ul>
&nbsp;

So what are we going to build?
<ul>
	<li>Well in order to make our life easier we will build a proxy web service; which will talk directly to CRM2011 this way we will make our integration efforts go smoother.</li>
</ul>
&nbsp;

&nbsp;
<h3>Step 4: How do I hook it all together?</h3>
Well for this part we will use BizTalk, BizTalk will receive the ‘Create Customer’ event from CRM and subsequently logic will be applied such that this data is send to the custom ERP application. Once the insert was successful the ERP system sends back an customer account number and subsequently we will update the corresponding Entity in CRM2011 with the account number obtained from the ERP system. 

&nbsp;

So which ingredients are required to do this?
<ul>
	<li>Well if you have not yet downloaded the CRM2011 SDK; go and fetch it <a href="http://www.microsoft.com/download/en/details.aspx?id=24004" target="_blank" rel="noopener noreferrer">here</a> :-)</li>
</ul>
&nbsp;

So what are we going to build?
<ul>
	<li>Well we need to make a customization to our Account Entity in CRM2011, to be more specific; we will add a custom field to the Account entity and call it Account Number.</li>
</ul>
<ul>
	<li>We will build a BizTalk solution which will hook all the bits together.</li>
</ul>
&nbsp;
<h2> </h2>
<h2>Closing Note</h2>
So this sums up the introduction part. Be sure to check back soon for the follow up part in which I’ll discuss how to build our CRM Trigger