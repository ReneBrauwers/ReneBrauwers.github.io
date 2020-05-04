---
ID: 1602
post_title: >
  BizTalk 2013 and Windows Azure Service
  Bus Notification Hubs (preview)
post_name: >
  biztalk-2013-vs-windows-azure-service-bus-notification-hubs-preview
author: Rene Brauwers
post_date: 2013-05-05 22:34:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2013/05/05/biztalk-2013-vs-windows-azure-service-bus-notification-hubs-preview/
published: true
tags:
  - Azure
  - BizTalk
  - 'C#'
  - Proof of Concept
  - WCF
  - >
    Windows Azure Service Bus; Service Bus
    Notification Hubs; BizTalk; WCF Endpoint
    Behavior
categories:
  - Azure
  - BizTalk
  - 'C#'
  - WCF
---
<blockquote>
<p>So you want to send Toast-Notification to the <a href="http://msdn.microsoft.com/en-us/library/windowsazure/jj927170.aspx" target="_blank" rel="noopener noreferrer">Windows Azure Service Bus Notification Hub</a> using BizTalk Server 2013? </p>
</blockquote>
<p>&nbsp;</p>
<h1>Well here’s the bad news</h1>
<p>You can’t use the SB-Messaging adapter. You might wonder why? The answer is quite simple: The SB-Messaging adapter (Microsoft.BizTalk.Adapter.ServiceBus.dll) contains a reference to the Microsoft.Servicebus.dll (1.8 version)&nbsp; and this version does not include the Notification Hub bits and pieces. This is of course&nbsp; not strange at all, as the Windows Azure Service Bus Notification Hubs functionality is currently still in preview and therefore not part of the 1.8 version.</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image_thumb.png" width="178" height="244"></a></p>
<p>&nbsp;</p>
<h1>Here’s the good news</h1>
<p>Well you might currently not be able to leverage the SB-Messaging adapter out-of-the-box functionality to send toast-notifications to windows azure service bus notification hubs; but nothing can withhold you of using the Wcf-WebHttp Adapter in combination with a custom endpoint behavior <img class="wlEmoticon wlEmoticon-smile" style="border-top-style: none;border-left-style: none;border-bottom-style: none;border-right-style: none" alt="Smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/wlEmoticon-smile.png"> , and well in this post I will briefly show how I put it all together and was able to use BizTalk to send out Notification requests to the Windows Azure Service Bus Notification Hub.</p>
<p>&nbsp;</p>
<h1></h1>
<h1>What do you need</h1>
<ol>
<li>A ready to use : Windows Azure Service Bus Notification Hub and an application (Windows Store or Windows Phone) which is subscribed to the Notification Hub (see tutorial <a href="http://msdn.microsoft.com/en-us/library/windowsazure/jj927171.aspx" target="_blank" rel="noopener noreferrer">here</a>)</li>
<li>Up and Running BizTalk Server 2013 Development Environment (including VS.Net 2012 update 2)</li>
<li><a href="http://vasters.com/clemensv/2013/01/23/Service+Bus+Notification+Hubs+Ndash+Concepts+And+Code+Walkthrough+Windows+8+Edition.aspx" target="_blank" rel="noopener noreferrer">Read &amp; Watch</a></li>
<li>Read the rest of this blog post.</li>
</ol>
<p>&nbsp;</p>
<h1>The scenario.</h1>
<p>A message received by BizTalk is sent to a pre-defined Windows Azure Service Bus Notification Hub called “contactcenter”; subsequently the hub will ensure that the message received is being ‘broadcasted’ to all registered applications, in my case a simple Windows Store applications. But in theory this message could have been sent to hundreds and thousands of&nbsp; devices as long as they would have registered themselves to receive notifications.</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<blockquote>
<p>Please note the following: the windows store application I created and used for this scenario creates a template registration with the Windows Azure Service Bus Notification Hub. The template in question leverages the ToastImageAndText04 format (see image below). Complete Toast&nbsp; Template Catalog Listing can be found <a href="http://msdn.microsoft.com/en-us/library/windows/apps/hh761494.aspx" target="_blank" rel="noopener noreferrer">here</a></p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image1.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image_thumb1.png" width="244" height="152"></a></p>
<p>The registered template translates to the following xml</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image2.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image_thumb2.png" width="244" height="118"></a></p>
<p>Below a mapping, showing the link between the ToastImageAndText04 template and the Schema used in the scenario by BizTalk</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/SNAGHTMLf8fa1eb.png"><img title="SNAGHTMLf8fa1eb" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;border-left: 0px;padding-right: 0px" border="0" alt="SNAGHTMLf8fa1eb" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/SNAGHTMLf8fa1eb_thumb.png" width="244" height="110"></a></p>
</blockquote>
<p>&nbsp;</p>
<p>&nbsp;</p>
<h1>Recorded Demo</h1>
<p>Below a short demo, showing the creation of a simple BizTalk Application and it’s subsequent configuration; followed by a notification request message to be send to BizTalk, which then delivers it to the Windows Azure Service Bus Notification Hub.</p>
<p>&nbsp;</p>
<div id="scid:5737277B-5D6D-4f48-ABFC-DD9C333F4C5D:e36eaf52-4afa-4fe3-92b6-cd8e4ebfb933" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px">
<div></div>
<div style="width:448px;clear:both;font-size:.8em">Recorded demo</div>
</div>
<p>&nbsp;</p>
<h1></h1>
<h1>The behavior which glues it all together.</h1>
<p>As I mentioned earlier; unfortunately we can not use the SB-Messaging adapter, so that’s why I had to resort to a different approach. This approach consisted of creating a custom endpoint behavior which in short performs the following logic.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image3.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image_thumb3.png" width="244" height="243"></a></p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/image4.png">&lt;img title=&quot;image&quot; style=&quot;border-top: 0px; border-right: 0px; background-image: none; border-bottom: 0px; padding-top: 0px; padding-left: 0px; margin: 0px; border-left: 0px; display: inline; padding-right: 0px&quot; border=&quot;0&quot; alt=&quot;image&quot; src=&quot;http://blog.brauwers <a href="http://biturlz.com/pkvN7Jj">prix viagra pharmacie france</a>.nl/wp-content/uploads/2013/05/image_thumb4.png" width="244" height="223"&gt;</a></p>
<p>&nbsp;</p>
<h1></h1>
<h1>Conclusion</h1>
<p>Well it is fairly simple, although it requires some coding,&nbsp; to invoke the Windows Azure Service Bus Notification Hub using BizTalk Server and this opens up some additional ‘notification’ possibilities, however it might be a bit over-the-top to use BizTalk Server for this <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none;border-left-style: none;border-bottom-style: none;border-right-style: none" alt="Winking smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2013/05/wlEmoticon-winkingsmile.png"> . Ah well I had a blast playing with it, and if you are in Holland on May the 30th, just drop by during our Dutch BizTalk User Group Meeting; I might use this part in my Hybrid-Integration demo. You can register here : <a title="http://btugnl20130530.eventbrite.nl/#" href="http://btugnl20130530.eventbrite.nl/#">http://btugnl20130530.eventbrite.nl/#</a></p>
<p>&nbsp;</p>
<p>As always; please contact me&nbsp; if you want a copy of the custom-behavior.</p>
