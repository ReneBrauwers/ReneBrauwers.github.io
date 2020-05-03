---
ID: 3082
post_title: 'Resolved issue: Sending message from Windows Azure BizTalk Services Bridge to another Bridge'
post_name: >
  resolved-issue-sending-message-from-windows-azure-biztalk-services-bridge-to-another-bridge
author: Rene Brauwers
post_date: 2014-02-04 19:00:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2014/02/04/resolved-issue-sending-message-from-windows-azure-biztalk-services-bridge-to-another-bridge/
published: true
tags:
  - Azure
  - BizTalk Services
  - Bridge
  - 'C#'
  - Certificate
  - Cloud
  - Issue
  - MessageInspector
  - Microsoft
  - ServicePointManager
  - SSL
  - Troubleshooting
  - WABS
  - Windows Azure BizTalk Services
categories:
  - Azure
  - 'C#'
  - SSL
  - Windows Azure BizTalk Services
---
<p>Next week I will be doing a <a href="https://www.eventbrite.com/e/btugbe-february-11-workflow-and-bam-in-the-cloud-tickets-10069290519?ref=ebtnebregn" target="_blank" rel="noopener noreferrer">talk</a>&nbsp; on Windows Azure BizTalk Services&nbsp; with regards on how one can add BAM functionality. During this talk a demo will be the ‘Pièce de résistance’. this demo is based on an <a href="http://bit.ly/1lD2BYM" target="_blank" rel="noopener noreferrer">article I have written earlier and which can be found on TechNet</a>.&nbsp; Well to cut to the chase… I would not be who I am if I would have not taken this article to the next level and while doing so I encountered a nice challenge.</p> <p>&nbsp;</p> <p>In my demo there is a scenario in which I have a custom MessageInspector which can be configured as such that it can deliver messages to either a Servicebus Queue/Topic/Relay or BizTalk Service Bridge endpoint. While testing the various scenario's I encountered a particular issue when I tried to send a message to another bridge. The error message which was 'reported back stated' </p> <p>&nbsp;</p> <blockquote> <p>Component xpathExtractor. Pipeline Runtime Error Microsoft.ApplicationServer.Integration.Pipeline.PipelineException: An error occurred while parsing EntityName. Line 5, position 99.<br>at Microsoft.ApplicationServer.Integration.Pipeline.Activities.XmlActivitiesExceptionHelper.Throw(String message, XmlActivityErrorCode errorCode, String activityName, String stageName, IEnumerable`1 messageProperties)</p></blockquote> <p>&nbsp;</p> <p>The above mentioned error was caused by the error-message which was sent back and caused a failure which I could not easily debug any further without a complete rewrite. While I had no time for this I decided to use a different approach in sending the messages. Up to that point I used the <a href="http://msdn.microsoft.com/en-us/library/system.net.webclient(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">WebClient</a> class in combination with the <a href="http://msdn.microsoft.com/en-us/library/ms144225(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">UploadDataAsync</a> Method but I decided to give the <a href="http://msdn.microsoft.com/en-us/library/ms144215(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">OpenWrite</a> method a go. This decision proved to be very useful as my existing exception-handling was not able 'kick in' without causing the parent <a href="http://msdn.microsoft.com/en-us/library/hh750113.aspx" target="_blank" rel="noopener noreferrer">TASK</a> to go in to a <a href="http://msdn.microsoft.com/en-us/library/system.threading.tasks.task.isfaulted(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">faulted state</a>.&nbsp; Thus by using the OpenWrite method I was able to extract he 'real' exception. This exception message stated:</p> <p>&nbsp;</p> <blockquote> <p>"The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel."&nbsp; "The remote certificate is invalid according to the validation procedure."</p></blockquote> <p>&nbsp;</p> <p>At that point I reached the Eureka moment and it all started to make sense. It all boils down to the fact that for development/test purposes we all use a self-signed certificate (created for us when we provisioned out Windows Azure BizTalk Service). This certificate is installed on our client machines, thus allowing us to make requests to the Windows Azure BizTalk Service. This explained why my test-console application did not throw any exceptions when calling the Windows Azure BizTalk Service Bridge in question. However when this code is invoked from within the message inspector which is hosted in Windows Azure.' it runs into the fact that 'there is a problem with the security certificate', this makes sense as it is a self-signed certificate. </p> <p>&nbsp;</p> <p>So now that I figured out the why, I needed a way to tell my code to ignore these kind of warning when encountered. Well luckily for is, this little gem is available and can be found in the <a href="http://msdn.microsoft.com/en-us/library/system.net.servicepointmanager(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">System.Net.ServicePointManager</a> class. This gem is called <a href="http://msdn.microsoft.com/en-us/library/system.net.servicepointmanager.servercertificatevalidationcallback(v=vs.110).aspx" target="_blank" rel="noopener noreferrer">ServerCertificateValidationCallBack</a>. This CallBack will return false in case the Server Certificate is not complying with the policies (in our case, it is by default not trusted as it is self-signed), so all we need to do is ensure that it always returns true and thus ignoring the security check </p> <p>&nbsp;</p> <blockquote> <p>&nbsp;<strong><font color="#ff0000">please do not do this in production environments! You otherwise might be ending up sending data to a not trusted source (i.e spoofed server etc.)</font></strong></p></blockquote> <p>&nbsp;</p> <p>Below I added the piece of code which implements the ssl validation-check bypass:</p> <p>&nbsp;</p> <div class="csharpcode"><pre class="alt"><font size="1"><span class="kwrd">using</span> (var client = <span class="kwrd">new</span> WebClient())</font></pre><pre><font size="1">{</font></pre><pre class="alt"><font size="1">&nbsp;</font></pre><pre><font size="1">    Uri EndPointAddress = <span class="kwrd">new</span> Uri(<span class="kwrd">this</span>.Endpoint);</font></pre><pre class="alt"><font size="1">&nbsp;</font></pre><pre><font size="1">             </font></pre><pre class="alt"><font size="1">    <span class="rem">//Add WRAP ACCESS TOKEN</span></font></pre><pre><font size="1">    client.Headers[HttpRequestHeader.Authorization] = String.Format(<span class="str">"WRAP access_token="{0}""</span>, <span class="kwrd">this</span>.AcsToken);</font></pre><pre class="alt"><font size="1">&nbsp;</font></pre><pre><font size="1">    <span class="rem">//Add Content Type</span></font></pre><pre class="alt"><font size="1">    client.Headers[<span class="str">"Content-Type"</span>] = <span class="str">"application/xml"</span>;</font></pre><pre><font size="1">&nbsp;</font></pre><pre class="alt"><font size="1">    <span class="rem">//Publish  </span></font></pre><pre><font size="1">    <span class="kwrd">try</span></font></pre><pre class="alt"><font size="1">    {</font></pre><pre><font size="1">&nbsp;</font></pre><pre class="alt"><font size="1">        <span class="rem">//The 'GEM' Ignore validation callback which cause an error</span></font></pre><pre><font size="1">        ServicePointManager.ServerCertificateValidationCallback += (sender, certificate, chain, sslPolicyErrors) =&gt; <span class="kwrd">true</span>;</font></pre><pre class="alt"><font size="1">&nbsp;</font></pre><pre><font size="1">        <span class="kwrd">using</span> (var stream = client.OpenWrite(EndPointAddress, <span class="str">"POST"</span>))</font></pre><pre class="alt"><font size="1">        {</font></pre><pre><font size="1">            <span class="kwrd">byte</span>[] data = System.Text.Encoding.UTF8.GetBytes(messagePayload);</font></pre><pre class="alt"><font size="1">            stream.Write(data, 0, data.Length);</font></pre><pre><font size="1">        }</font></pre><pre class="alt"><font size="1">        </font></pre><pre><font size="1">    }</font></pre><pre class="alt"><font size="1">    <span class="kwrd">catch</span> (System.Net.WebException wex)</font></pre><pre><font size="1">    {</font></pre><pre class="alt"><font size="1">        AppendExceptionsToLog(wex);</font></pre><pre><font size="1">    }</font></pre><pre class="alt"><font size="1">    <span class="kwrd">catch</span> (ArgumentNullException anex)</font></pre><pre><font size="1">    {</font></pre><pre class="alt"><font size="1">        AppendExceptionsToLog(anex);</font></pre><pre><font size="1">    }</font></pre><pre class="alt"><font size="1">    <span class="kwrd">catch</span> (Exception ex)</font></pre><pre><font size="1">    {</font></pre><pre class="alt"><font size="1">        AppendExceptionsToLog(ex);</font></pre><pre><font size="1">    }</font></pre><pre class="alt"><font size="1">}</font></pre></div>
<style type="text/css">.csharpcode, .csharpcode pre
{
	font-size: small;
	color: black;
	font-family: consolas, "Courier New", courier, monospace;
	background-color: #ffffff;
	/*white-space: pre;*/
}
.csharpcode pre { margin: 0em; }
.csharpcode .rem { color: #008000; }
.csharpcode .kwrd { color: #0000ff; }
.csharpcode .str { color: #006080; }
.csharpcode .op { color: #0000c0; }
.csharpcode .preproc { color: #cc6633; }
.csharpcode .asp { background-color: #ffff00; }
.csharpcode .html { color: #800000; }
.csharpcode .attr { color: #ff0000; }
.csharpcode .alt 
{
	background-color: #f4f4f4;
	width: 100%;
	margin: 0em;
}
.csharpcode .lnum { color: #606060; }
</style>

<style type="text/css">.csharpcode, .csharpcode pre
{
	font-size: small;
	color: black;
	font-family: consolas, "Courier New", courier, monospace;
	background-color: #ffffff;
	/*white-space: pre;*/
}
.csharpcode pre { margin: 0em; }
.csharpcode .rem { color: #008000; }
.csharpcode .kwrd { color: #0000ff; }
.csharpcode .str { color: #006080; }
.csharpcode .op { color: #0000c0; }
.csharpcode .preproc { color: #cc6633; }
.csharpcode .asp { background-color: #ffff00; }
.csharpcode .html { color: #800000; }
.csharpcode .attr { color: #ff0000; }
.csharpcode .alt 
{
	background-color: #f4f4f4;
	width: 100%;
	margin: 0em;
}
.csharpcode .lnum { color: #606060; }
</style>

<p>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; <br>Cheers and hope to see you soon! Especially if you are going to attend the <a href="http://www.biztalk360.com/BizTalk-Summit-2014/index.html" target="_blank" rel="noopener noreferrer">BizTalk Summit 2014</a>&nbsp; in London this March, don't hesitate to say Hi :-)</p>