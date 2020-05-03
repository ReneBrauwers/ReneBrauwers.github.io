---
ID: 4870
post_title: PowerApps– Are you ready to power UP?
post_name: powerapps-are-you-ready-to-power-up
author: Rene Brauwers
post_date: 2016-03-09 10:37:04
layout: post
link: >
  https://brauwers.azurewebsites.net/2016/03/09/powerapps-are-you-ready-to-power-up/
published: true
tags: [ ]
categories:
  - PowerApps
---
<p>I still remember when I first I heard Microsoft was working on Project Siena (which later became PowerApps), the first thing which popped up in my mind was Visual Studio Lightswitch, which allows us to easily create business applications. Anyways I am digressing J … I might blog about Lightswitch vs PowerApps in the near future, otherwise just attend the upcoming <a href="http://j.mp/globalazure">Global Azure Bootcamp in Sydney</a>, and ask me in person.</p>
<h1>IntegrationUserGroup</h1>
<p>On February 22<sup>nd</sup> I gave a closet presentation on PowerApps on <a href="http://www.integrationusergroup.com/powerapps-great-power-comes-great-responsibility/">IntegrationUsergroup.com</a>. Yeah you read it right, a closet presentation J. Just imagine the following, to get a better understanding…</p>
<p>6 am Sydney, as the sun rises the temperature slowly increases. I just got my first cup of Coffee. I hooked up my laptop to the Big Screen TV. While the laptop is booting I take a sip of my Coffee and put on my Bluetooth headset. After logging in, I perform a final sound check and open up my power point presentation. Once everything seems to be working, I connect to the IntegrationUserGroup webinar. There I am, sitting alone in the living with a cup of coffee, ready to present to a virtual audience.</p>
<p>darn, I’ve been digressing again. Ok where was I. O yeah PowerApps</p>
<h1>Business App Gap</h1>
<p>Before I dive a bit more into PowerApps, lets briefly look at why Microsoft has created the PowerApps platform.</p>
<p>In this Mobile First, Cloud First world we are surrounded with thousands if not millions of mobile apps, but only a fraction of these apps are truly mobile business apps.</p>
<p>So what is the reason for these mobile business app type apps to be lacking behind? Well that’s the question Microsoft asked, and they were able to narrow it down for the following reasons:</p>
<h2>It’s hard to develop true mobile business apps.</h2>
<p><b><br />
</b>Mainly because one will have to target devices running in multiple form factors (phone, phablet, tablet, laptop, desktop) across multiple operating systems (iOS, Android, Windows)</p>
<h2><b>Data is spanned both on premise as well as in the cloud</b>.</h2>
<p>&nbsp;</p>
<p><b></b>Company data is nowadays stored virtually anywhere. Some data might be stored in a SaaS application hosted somewhere in the cloud, while other data might be stored on premise. Accessing this data and integrating these systems is not an easy job.</p>
<h2>Application Deployment.</h2>
<p><b><br />
</b>Once the application(s) have been developed they need to be deployed to a user’s device. Currently in order to be able to do this an application has to be published to an official market-place, and in case we target multiple operating systems (iOS, Android, Windows) we will have to target multiple market-places, with their own processes.</p>
<p>An alternative to the above would of course be side-loading apps, or setting up company market-places. However not all platforms might allow this.</p>
<h2>In short</h2>
<p>it is pretty damn hard to not only create, integrate and deploy business apps it is most likely takes time, hard work and money to develop and maintain these apps.</p>
<p>Concluding we could say</p>
<p>If you are able to build a business app within minutes, and deploy it to all kinds of devices regardless of their operating systems. You should have a pretty darn valuable business proposition.</p>
<p>Wait…Having said the above…</p>
<h1>Introducing PowerApps</h1>
<p>In short PowerApps is Microsoft’s answer to address the business app gap, it does so by offering a platform which includes tooling to enable employees, developers, and integrators to create and share mobile business apps. These apps work on phones, tablets or desktops and they work across iOS, Windows and soon Android and allow to seamlessly connect to disparate data sources spanning both on-prem and the cloud in a secure way <a href="http://biturlz.com/kG5Leks">viagra en belgique</a>.</p>
<p>Microsoft PowerApps was <a href="http://blogs.microsoft.com/blog/2015/11/30/introducing-microsoft-powerapps/">announced</a> a few months ago and so far getting access to the preview is on an invitation basis only. You can go to <a href="https://powerapps.microsoft.com/en-us/">https://powerapps.microsoft.com/en-us/</a> and request an invite.</p>
<p>Pfff, enough of this blurb stuff, let’s cut to the chase. (click <a href="https://azure.microsoft.com/en-us/documentation/suites/powerapps/">here</a> for some more technical details on what PowerApps are)</p>
<h2>Why should you use PowerApps?</h2>
<p>Easy answer, why shouldn’t you. No seriously. In my opinion, if you as a business have an active ‘power user’ base who are more than comfortable with Excel and Access and want to quickly leverage ‘intranet’ like business apps which boost productivity or simply allow user to gain quick access, anytime and anywhere to business information, PowerApps is the platform to go for, your imagination is your limitation.</p>
<p>Okay, okay; it would be ideal if the following infrastructure and solutions/platforms are being leveraged by your company: Azure Active Directory Tenant, Dropbox, Dynamics CRM Online, Google Drive, Microsoft Translator, Office 365 Outlook, Office 365 Users, OneDrive, SQL Azure, Salesforce, SharePoint Online, Twitter or any publically facing rest API (preferably with a swagger definition)</p>
<p>In short, if you meet the above requirements, go sign up for PowerApps and start prototyping.</p>
<p>Lacking inspiration, well why not build a PowerApp which allows users to report on ‘Hazardous situations’?</p>
<p>So this app would allow users, if they see a hazardous situation to instantly report this by describing the situation, attaching a picture and the exact location (using GPS). Once reported the responsible business unit can take action and once resolved the one who reported would be notified that the hazardous situation is resolved.</p>
<p>Sounds too good to be true; nah in my following post. I’ll show you how to build and deploy this within an hour; Yeah using the Free or Standard version</p>
<h2>What type of apps can I build using PowerApps?</h2>
<p>In my presentation I identify 2 types of business apps which make prime candidates for PowerApps.</p>
<ul>
<li>Intranet like mobile business apps</li>
<li>Line of business like mobile business apps</li>
</ul>
<p>The differences between these two apps are best to be explained by listing a few examples</p>
<p>Intranet like mobile business apps, are typically mobile business apps which offer intranet like functionality and usually contain (if any) a simple workflow (logic flows AKA Logic App) such as</p>
<ul>
<li>Expense declaration</li>
<li>Timesheets</li>
<li>Leave request</li>
<li>Service Desk</li>
<li>Meeting room planner</li>
<li>Event signup</li>
<li>Company news</li>
</ul>
<p>Line of business like mobile business apps, typically would expose and tap into core business processes, have more complex workflows (logic flows AKA Logic App) which could span multiple back-end systems, typically these business apps would want to leverage functionality which is contained within the space of</p>
<ul>
<li>Warehouse Management</li>
<li>Order Processing</li>
<li>Supply Chain</li>
<li>Payroll</li>
<li>Transport Planning</li>
</ul>
<p>This latter type of mobile business apps, usually takes some more time to develop and in my opinion requires a good ‘Design’ process and of course special attention needs to be given to the Integration Architecture.</p>
<p>In short, an integration specialist needs to build a decoupled API layer leveraging the full (professional) integration stack which is to one’s disposal. These APIs can then be surfaces as custom business connections such that they can be dragged, dropped and configured by the ‘Power / Business’ user.</p>
<h2><b>My golden rules</b></h2>
<p>If you want to display ‘simple’ information, go ahead and hook directly into the required API’s or leverage the default connections</p>
<p>However, if you want to display composite information, you’d better keep integration practices in the back of your mind. I can only recommend building custom rest APIs with a ‘single’ purpose.</p>
<p>For applications which tap into a business process and mutate / manipulate / insert / update data which affect the process and applications, require ‘guaranteed processing’ are transactional based or require compensation logic I’d recommend to leverage for example Azure Service Bus (light-weight but powerful) maybe in combination with an on premise Middleware platform such as our beloved BizTalk.</p>
<h2>What’s next.</h2>
<p>My upcoming posts will be covering more hands-on topics, in which I both will guide you through the building process of a simple intranet like business app as well as a LOB business app.</p>
<p>Well I hoped this post was of any use to you. I purposely did not specifically dive into the different components which make up PowerApps (Designer, Logic Flows) nor the difference between the different PowerApps tiers as there are already some good resources available on the world wide web diving into these.</p>
<p>So if you require some more details on PowerApps please have a look at the following link &#8211; <a href="http://bit.ly/1THnLL8">http://bit.ly/1THnLL8</a> -, which I find very helpful and will guide you in your discovery path into the wonderful world of PowerApps</p>
<p>Cheers and feel free to add some comments to get the discussion going!</p>
<p>René</p>
