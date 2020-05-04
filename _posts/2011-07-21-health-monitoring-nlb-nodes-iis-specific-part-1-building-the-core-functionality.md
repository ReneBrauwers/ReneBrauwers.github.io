---
ID: 39
post_title: >
  Health Monitoring NLB Nodes (IIS
  specific)
post_name: >
  health-monitoring-nlb-nodes-iis-specific-part-1-building-the-core-functionality
author: Rene Brauwers
post_date: 2011-07-21 15:32:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2011/07/21/health-monitoring-nlb-nodes-iis-specific-part-1-building-the-core-functionality/
published: true
tags:
  - Application Pool
  - BizTalk
  - BizTalk Server 2010
  - 'C#'
  - Event log manager
  - IIS
  - NLB
  - Windows Service
  - WMI
categories:
  - BizTalk
  - 'C#'
  - WMI
---
In our previous post we’ve setup a Network Load Balancing solution for BizTalk Server 2010; this solution ensures that ‘Web Traffic’ is balanced between two dedicated BizTalk Servers.

&nbsp;

Well one of the caveats of a software NLB solution is the fact that it’s main role is to balance network traffic to 2 or more servers and it will not check if the ‘Traffic Destination Service (endpoint)’ is available, it will only check that the NLB nodes (servers) are available.

&nbsp;

In our case this could mean that if either the BizTalk Application Pool or the BizTalk Website (endpoint) on one or both of the BizTalk NLB nodes are malfunctioning that traffic could still be rerouted to this node; resulting in those specific BizTalk Endpoints no longer being accessible/available. And of course this is something which is not desirable in our High Availability BizTalk Server Environment.

&nbsp;

So in order to address above mentioned ‘issue’; I’ve decided to blog about one of the possible solutions which in theory comes down to the following:

&nbsp;
<ul>
	<li>Build a service which monitors if the participating Application Pools and Websites in our NLB node are up and running and in case they are malfunctioning disable that particular node in our NLB Cluster</li>
</ul>
&nbsp;

This post will only covering building the core functionality and I will leave it up to the reader to implement this logic in their own windows service or other monitoring tool.

Let’s start!

&nbsp;
<blockquote><em>Please note; the style of this article will be quite different compared to the previous posts and will consist more of a ‘Challenge –&gt; Solution’ approach using C# Code samples. </em></blockquote>
&nbsp;
<h2>Setting up our Visual Studio 2010 Solution</h2>
So Start up Visual Studio 2010 and create a new ‘Class Library’ Project and name it ‘WmiMonitor’ and name the solution to be created ‘ServerMonitor’.

&nbsp;

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/07/image.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/07/image_thumb.png" width="244" height="169" border="0" /></a>

&nbsp;

Include the following reference to this project: <strong><em>System.Management.</em></strong>

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/07/image1.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/07/image_thumb1.png" width="244" height="166" border="0" /></a>

&nbsp;

Add a new ‘Class item’ and name it: <strong><em>WmiMonitor.cs </em></strong>
<blockquote>This class will hold all functionality with regards to our WMI Functionality</blockquote>
<a href="https://brauwers-nl.azureedge.net/images/blog/2011/07/image2.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/07/image_thumb2.png" width="244" height="170" border="0" /></a>

&nbsp;

Add a new ‘Class item’ and name it: <strong><em>WmiMonitorResults.cs</em></strong>
<blockquote>This class will contain our properties used to hold our ‘WMI Query Results</blockquote>
Add a new ‘Class item’ and name it: <strong><em>EventLogManager.cs</em></strong>
<blockquote>This class will contain functionality used for writing any exceptions which might occur to the windows Eventlog</blockquote>
At this point your solution should look as follows:

<a href="https://brauwers-nl.azureedge.net/images/blog/2011/07/image3.png"><img style="background-image: none; margin: 0px; padding-left: 0px; padding-right: 0px; display: inline; padding-top: 0px; border-width: 0px;" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2011/07/image_thumb3.png" width="211" height="244" border="0" /></a>

&nbsp;
<h3></h3>
<h2>Completing the project</h2>
At this point we’ve set up our solution and defined the artifacts needed for our project. In the next few subchapters we will actually add the code, which completes this project.

&nbsp;
<h3>EventlogManager</h3>
Well in all applications exceptions might occur and as our end result will be a windows service which needs to run continuously (meaning; it should not crash when an error occurs) it would be beneficial if we would have functionality which would allow us to log the exception details to the windows event log, such that we can monitor our monitor :-) Below I’ve listed the functionality which does this.

&nbsp;

So open up your EventLogmanager.cs file, and replace the default contents with the code below and see the inline comments for a more detailed explanation.

&nbsp;
<pre class="code"><span style="color: blue;">using </span>System;
<span style="color: blue;">using </span>System.Collections.Generic;
<span style="color: blue;">using </span>System.Linq;
<span style="color: blue;">using </span>System.Text;
<span style="color: blue;">using </span>System.Diagnostics;

<span style="color: blue;">namespace </span>Monitoring
{
    <span style="color: gray;">/// &lt;summary&gt;
    /// </span><span style="color: green;">Static Class which contains functionality to write 'events' to the default Windows Application Log
    </span><span style="color: gray;">/// &lt;/summary&gt;
    </span><span style="color: blue;">public static class </span><span style="color: #2b91af;">EventLogManager
    </span>{
        <span style="color: blue;">private const string </span>DefaultLog = <span style="color: #a31515;">"Application"</span>;

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Write Warning to EventLog
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="eventSource"&gt;</span><span style="color: green;">Name of source which will be displayed in the eventlog</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="warning"&gt;</span><span style="color: green;">Warning Text which will be the description of the eventlog entry </span><span style="color: gray;">&lt;/param&gt;   
        </span><span style="color: blue;">public static void </span>WriteWarning(<span style="color: blue;">string </span>eventSource, <span style="color: blue;">string </span>warning)
        {
            <span style="color: green;">//Call method which actually writes to the eventlog
            </span>WriteToEventlog(eventSource, warning, <span style="color: #2b91af;">EventLogEntryType</span>.Warning);
            warning = <span style="color: blue;">null</span>;
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Write Info to EventLog
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="eventSource"&gt;</span><span style="color: green;">Name of source which will be displayed in the eventlog</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="info"&gt;</span><span style="color: green;">Info Text which will be the description of the eventlog entry</span><span style="color: gray;">&lt;/param&gt;   
        </span><span style="color: blue;">public static void </span>WriteInfo(<span style="color: blue;">string </span>eventSource, <span style="color: blue;">string </span>info)
        {
            <span style="color: green;">//Call method which actually writes to the eventlog
            </span>WriteToEventlog(eventSource, info, <span style="color: #2b91af;">EventLogEntryType</span>.Information);
            info = <span style="color: blue;">null</span>;
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Write Error to EventLog
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="eventSource"&gt;</span><span style="color: green;">Name of source which will be displayed in the eventlog</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="error"&gt;</span><span style="color: green;">Error Text which will be the description of the eventlog entry</span><span style="color: gray;">&lt;/param&gt;   
        </span><span style="color: blue;">public static void </span>WriteError(<span style="color: blue;">string </span>eventSource, <span style="color: blue;">string </span>error)
        {
            <span style="color: green;">//Call method which actually writes to the eventlog
            </span>WriteToEventlog(eventSource, error, <span style="color: #2b91af;">EventLogEntryType</span>.Error);
            error = <span style="color: blue;">null</span>;
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">private Method which actually stores data in the eventlog
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="source"&gt;</span><span style="color: green;">Name of source which will be displayed in the eventlog</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="message"&gt;</span><span style="color: green;">Message which will be the description of the eventlog entry</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="entryType"&gt;</span><span style="color: green;">Indication of the eventlog entry type</span><span style="color: gray;">&lt;/param&gt;
        </span><span style="color: blue;">private static void </span>WriteToEventlog(<span style="color: blue;">string </span>source, <span style="color: blue;">string </span>message, <span style="color: #2b91af;">EventLogEntryType </span>entryType)
        {

                <span style="color: green;">//Check if the EventSource exists, if not create it and use the default log for this
                </span><span style="color: blue;">if </span>(!<span style="color: #2b91af;">EventLog</span>.SourceExists(source))
                {
                    <span style="color: #2b91af;">EventLog</span>.CreateEventSource(source, DefaultLog);
                }

                <span style="color: green;">//Write entry to eventlog, if the message exceeds the max allowed size it will be truncated
                </span><span style="color: #2b91af;">EventLog</span>.WriteEntry(source, TruncateEventEntry(message), entryType);            
                message = <span style="color: blue;">null</span>;            
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Truncates an eventlog entry if it exceeds the maximum available characters
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="input"&gt;</span><span style="color: green;">String to be checked on length</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;returns&gt;</span><span style="color: green;">input string which will be truncated when exceeding 20000 characters</span><span style="color: gray;">&lt;/returns&gt;
        </span><span style="color: blue;">private static string </span>TruncateEventEntry(<span style="color: blue;">string </span>input)
        {
            <span style="color: green;">//Check if string is null or empty
            </span><span style="color: blue;">if </span>(!<span style="color: #2b91af;">String</span>.IsNullOrEmpty(input))
            {
                <span style="color: green;">//Check length
                </span><span style="color: blue;">if </span>(input.Length &gt; 20000)
                {
                    <span style="color: green;">//return truncated string and add ... at the end indicating a truncated string
                    </span><span style="color: blue;">return </span>input.Substring(0, 19900) + <span style="color: #a31515;">"..."</span>;
                }
                <span style="color: blue;">else
                </span>{
                    <span style="color: green;">//return original string
                    </span><span style="color: blue;">return </span>input;
                }
            }
            <span style="color: blue;">else
            </span>{
                <span style="color: green;">//return string which mentions that there was no infomration
                </span><span style="color: blue;">return </span><span style="color: #a31515;">"No Information"</span>;
            }
        }
    }
}</pre>
<em>Note: that it does not include exception handling and if an exceptions are thrown they will have to be caught in the opertation invoking this class</em>

&nbsp;
<h3>WmiMonitorResult</h3>
This class will contain properties which can hold the status information with regards to the monitored objects; in our case (1) Application Pools (2) Websites.

&nbsp;

So open up your WmiMontorResult.cs file, and replace the default contents with the code below and see the inline comments for a more detailed explanation.

&nbsp;
<pre class="code"><span style="color: blue;">using </span>System;

<span style="color: blue;">namespace </span>Monitoring
{
    <span style="color: gray;">/// &lt;summary&gt;
    /// </span><span style="color: green;">Class used to hold information with regards to the status of the monitored items
    </span><span style="color: gray;">/// &lt;/summary&gt;
    </span><span style="color: blue;">public class </span><span style="color: #2b91af;">WmiMonitorResults
    </span>{
        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Server Name
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">public string </span>ServerName { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Name of the monitoring object
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">public string </span>ItemName { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Status Code which indicates the status of an item
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">public int </span>Status { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Friendly description of the Status code
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">public string </span>FriendlyStatusName { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }

    }
}</pre>
&nbsp;
<h3>WmiMonitor</h3>
This class will include all logic required for obtaining a NLB Server Node status with regards to the application pool and websites. Besides this it will include functionality to enable or disable a NLB node if required.

&nbsp;

Below I’ve listed the functionality which does this. So open up your WmiMonitor.cs file, and replace the default contents with the code below and see the inline comments for a more detailed explanation.

&nbsp;
<pre class="code"><span style="color: blue;">using </span>System;
<span style="color: blue;">using </span>System.Collections.Generic;
<span style="color: blue;">using </span>System.Management;
<span style="color: blue;">using </span>System.Linq;
<span style="color: blue;">using </span>System.Text;

<span style="color: blue;">namespace </span>Monitoring
{
    <span style="color: gray;">/// &lt;summary&gt;
    /// </span><span style="color: green;">Class which includes all functionality to determine a NLB nodes status with regards to Application Pools and Websites
    </span><span style="color: gray;">/// </span><span style="color: green;">as well as stopping and starting a NLB node. All of this is done by means of WMI events and thus requires elevated rights
    </span><span style="color: gray;">/// </span><span style="color: green;">to be executed succesfully
    </span><span style="color: gray;">/// &lt;/summary&gt;
    </span><span style="color: blue;">public class </span><span style="color: #2b91af;">WmiMonitor
    </span>{
        <span style="color: blue;">#region </span>properties
        <span style="color: green;">//Private properties
        </span><span style="color: blue;">private string </span>UserName { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }
        <span style="color: blue;">private string </span>Password { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }
        <span style="color: blue;">private string </span>Domain {<span style="color: blue;">get</span>;<span style="color: blue;">set</span>;}
        <span style="color: blue;">private string </span>RemoteComputer { <span style="color: blue;">get</span>; <span style="color: blue;">set</span>; }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Determines if WMI requests need to be performed using Impersonation
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">private bool </span>PerformImpersonation
        {
            <span style="color: blue;">get
            </span>{
                <span style="color: green;">//In case UserName/password is null or empty or the Remote Computer Name equals the current servername return true else false
                </span><span style="color: blue;">return </span>((<span style="color: #2b91af;">String</span>.IsNullOrEmpty(UserName) || <span style="color: #2b91af;">String</span>.IsNullOrEmpty(Password) || RemoteComputer.ToUpper() == <span style="color: #2b91af;">Environment</span>.MachineName.ToUpper()) ? <span style="color: blue;">true </span>: <span style="color: blue;">false</span>);
            }
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Object used to hold settings which are required to set up a WMI connection
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">private </span><span style="color: #2b91af;">ConnectionOptions </span>WmiConnectionOption
        {
            <span style="color: blue;">get
            </span>{
                <span style="color: green;">//initialize
                </span><span style="color: #2b91af;">ConnectionOptions </span>conOption = <span style="color: blue;">new </span><span style="color: #2b91af;">ConnectionOptions</span>();

                <span style="color: green;">//Set settings according to the choice of impersonation or not
                </span><span style="color: blue;">if </span>(<span style="color: blue;">this</span>.PerformImpersonation)
                {
                    conOption.Impersonation = <span style="color: #2b91af;">ImpersonationLevel</span>.Impersonate;

                    <span style="color: green;">/*IF WE DONT SET THE AUTHENTICATIONLEVEL TO PACKETPRIVACY WE'LL RECEIVE THE FOLLOWING ERROR
                    * The rootWebAdministration namespace is marked with the RequiresEncryption flag. 
                    * Access to this namespace might be denied if the script or application does not have the appropriate authentication level. 
                    * Change the authentication level to Pkt_Privacy and run the script or application again. 
                    */
                    </span>conOption.Authentication = <span style="color: #2b91af;">AuthenticationLevel</span>.PacketPrivacy;
                }
                <span style="color: blue;">else
                </span>{
                    conOption.Username = UserName;
                    conOption.Password = Password;

                    <span style="color: green;">/*IF WE DONT SET THE AUTHENTICATIONLEVEL TO PACKETPRIVACY WE'LL RECEIVE THE FOLLOWING ERROR
                    * The rootWebAdministration namespace is marked with the RequiresEncryption flag. 
                    * Access to this namespace might be denied if the script or application does not have the appropriate authentication level. 
                    * Change the authentication level to Pkt_Privacy and run the script or application again. 
                    */
                    </span>conOption.Authentication = <span style="color: #2b91af;">AuthenticationLevel</span>.PacketPrivacy;

                }

                <span style="color: blue;">return </span>conOption;
            }
        }

        <span style="color: blue;">#endregion
        #region </span>constructors
        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Default constructor which is used when we need to use the callers credentials when executing WMI events
        </span><span style="color: gray;">/// &lt;/summary&gt;
        </span><span style="color: blue;">public </span>WmiMonitor()
        {

        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Constructor used in case we want to override the used credentials to execute WMI events
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="userName"&gt;</span><span style="color: green;">Username</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="passWord"&gt;</span><span style="color: green;">Password</span><span style="color: gray;">&lt;/param&gt;
        </span><span style="color: blue;">public </span>WmiMonitor(<span style="color: blue;">string </span>userName, <span style="color: blue;">string </span>passWord, <span style="color: blue;">string </span>domain)
        {
            UserName = userName;
            Password = passWord;
            Domain = domain;
        }

        <span style="color: blue;">#endregion
        #region </span>Public Methods
        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Function which returns the application pool state
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="applicationPoolNames"&gt;</span><span style="color: green;">Name of application pool to check</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="computer"&gt;</span><span style="color: green;">Name of Computer</span><span style="color: gray;">&lt;/param&gt;
        </span><span style="color: blue;">public </span><span style="color: #2b91af;">WmiMonitorResults </span>GetApplicationPoolStatus(<span style="color: blue;">string </span>applicationPoolName, <span style="color: blue;">string </span>computer)
        {
            <span style="color: green;">//Set RemoteComputer
            </span>RemoteComputer = computer;

            <span style="color: green;">//prefill our mwi result class, which contains the state of the application pools
            </span><span style="color: #2b91af;">WmiMonitorResults </span>results = <span style="color: blue;">new </span><span style="color: #2b91af;">WmiMonitorResults</span>()
            {
                ServerName = computer,
                ItemName = applicationPoolName,
                FriendlyStatusName = <span style="color: #a31515;">"Not Found"</span>,
                Status = -1
            };

            <span style="color: blue;">try
            </span>{
                    <span style="color: green;">//WMI Connection and Scope
                    </span><span style="color: #2b91af;">ManagementScope </span>WmiScope = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementScope</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">@"{0}rootWebAdministration"</span>, computer),WmiConnectionOption);

                    <span style="color: green;">//WMI Query
                    </span><span style="color: #2b91af;">ObjectQuery </span>WmiQuery = <span style="color: blue;">new </span><span style="color: #2b91af;">ObjectQuery</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"SELECT * FROM ApplicationPool WHERE Name ='{0}'"</span>, applicationPoolName));

                    <span style="color: green;">//Actual 'wmi worker'
                    </span><span style="color: #2b91af;">ManagementObjectSearcher </span>searcher = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementObjectSearcher</span>(WmiScope,WmiQuery);

                <span style="color: green;">//Execute query and process the results which are stored as WmiMonitorResults object
                </span><span style="color: blue;">foreach </span>(<span style="color: #2b91af;">ManagementObject </span>queryObj <span style="color: blue;">in </span>searcher.Get())
                {
                    <span style="color: green;">//Get State
                    </span><span style="color: blue;">int </span>StateValue = -1;
                    <span style="color: blue;">if </span>(<span style="color: blue;">int</span>.TryParse(queryObj.InvokeMethod(<span style="color: #a31515;">"GetState"</span>, <span style="color: blue;">null</span>).ToString(), <span style="color: blue;">out </span>StateValue))
                    {
                        <span style="color: green;">//Store state status in return class
                        </span>results.Status = StateValue;

                        <span style="color: green;">//Determine friendly name of state and store this in the return class
                        </span>results.FriendlyStatusName = GetFriendlyApplicationPoolState(StateValue);

                    }

                }
            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">ManagementException </span>e)
            {
                results.Status = -2;
                results.FriendlyStatusName = e.Message;

                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[GetApplicationPoolStatus] {0}"</span>, e.Message));

            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">Exception </span>gex)
            {
                results.Status = -3;
                results.FriendlyStatusName = gex.Message;

                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[GetApplicationPoolStatus] {0}"</span>, gex.Message));
            }
            <span style="color: blue;">return </span>results;
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Method which returns the state of the websites
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="WebSiteName"&gt;</span><span style="color: green;">Name of website to check</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="computer"&gt;</span><span style="color: green;">Name of Computer</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        </span><span style="color: blue;">public </span><span style="color: #2b91af;">WmiMonitorResults </span>GetWebSiteStatus(<span style="color: blue;">string </span>WebSiteName, <span style="color: blue;">string </span>computer)
        {
            <span style="color: green;">//Set RemoteComputer
            </span>RemoteComputer = computer;

            <span style="color: green;">//prefill our mwi result class, which contains the state of the application pools
            </span><span style="color: #2b91af;">WmiMonitorResults </span>results = <span style="color: blue;">new </span><span style="color: #2b91af;">WmiMonitorResults</span>()
            {
                ServerName = computer,
                ItemName = WebSiteName,
                FriendlyStatusName = <span style="color: #a31515;">"Not Found"</span>,
                Status = -1
            };

            <span style="color: blue;">try
            </span>{
                <span style="color: green;">//WMI Connection and Scope
                </span><span style="color: #2b91af;">ManagementScope </span>WmiScope = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementScope</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">@"{0}rootWebAdministration"</span>, computer), WmiConnectionOption);

                <span style="color: green;">//WMI Query
                </span><span style="color: #2b91af;">ObjectQuery </span>WmiQuery = <span style="color: blue;">new </span><span style="color: #2b91af;">ObjectQuery</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"SELECT * FROM Site WHERE Name ='{0}'"</span>, WebSiteName));

                <span style="color: green;">//Actual 'wmi worker'
                </span><span style="color: #2b91af;">ManagementObjectSearcher </span>searcher = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementObjectSearcher</span>(WmiScope, WmiQuery);

                <span style="color: green;">//Execute query and process the results which are stored as WmiMonitorResults object
                </span><span style="color: blue;">foreach </span>(<span style="color: #2b91af;">ManagementObject </span>queryObj <span style="color: blue;">in </span>searcher.Get())
                {
                    <span style="color: blue;">int </span>StateValue = -1;
                    <span style="color: green;">//Get State
                    </span><span style="color: blue;">if </span>(<span style="color: blue;">int</span>.TryParse(queryObj.InvokeMethod(<span style="color: #a31515;">"GetState"</span>, <span style="color: blue;">null</span>).ToString(), <span style="color: blue;">out </span>StateValue))
                    {
                        <span style="color: green;">//Store state status in return class
                        </span>results.Status = StateValue;

                        <span style="color: green;">//Determine friendly name of state and store this in the return class
                        </span>results.FriendlyStatusName = GetFriendlyApplicationPoolState(StateValue);
                    }

                }
            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">ManagementException </span>e)
            {
                results.Status = -2;
                results.FriendlyStatusName = e.Message;

                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[GetWebSiteStatus] {0}"</span>, e.Message));

            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">Exception </span>gex)
            {
                results.Status = -3;
                results.FriendlyStatusName = gex.Message;

                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[GetWebSiteStatus] {0}"</span>, gex.Message));
            }
            <span style="color: blue;">return </span>results;
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Method which returns the nodes which are part of the NLB Server
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="nlbServer"&gt;</span><span style="color: green;">Server Name containing the NLB Feature</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;returns&gt;</span><span style="color: green;">String list of nodes which are part of the NLB Server</span><span style="color: gray;">&lt;/returns&gt;
        </span><span style="color: blue;">public </span><span style="color: #2b91af;">List</span>&lt;<span style="color: blue;">string</span>&gt; GetNLBComputers(<span style="color: blue;">string </span>nlbServer)
        {
            <span style="color: green;">//Set RemoteComputer
            </span>RemoteComputer = nlbServer;

            <span style="color: green;">//prefill our mwi result class, which contains the state of the application pools
            </span><span style="color: #2b91af;">List</span>&lt;<span style="color: blue;">string</span>&gt; returnValue = <span style="color: blue;">new </span><span style="color: #2b91af;">List</span>&lt;<span style="color: blue;">string</span>&gt;();
            <span style="color: blue;">try
            </span>{
                <span style="color: green;">//WMI Connection and Scope
                </span><span style="color: #2b91af;">ManagementScope </span>WmiScope = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementScope</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">@"{0}rootMicrosoftNLB"</span>, nlbServer), WmiConnectionOption);

                <span style="color: green;">//WMI Query
                </span><span style="color: #2b91af;">ObjectQuery </span>WmiQuery = <span style="color: blue;">new </span><span style="color: #2b91af;">ObjectQuery</span>(<span style="color: #a31515;">"SELECT * FROM MicrosoftNLB_Node"</span>);

                <span style="color: green;">//Actual 'wmi worker'
                </span><span style="color: #2b91af;">ManagementObjectSearcher </span>searcher = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementObjectSearcher</span>(WmiScope, WmiQuery);

                <span style="color: green;">//Execute Query and Get NLB Nodes                
                </span><span style="color: blue;">foreach </span>(<span style="color: #2b91af;">ManagementObject </span>queryObj <span style="color: blue;">in </span>searcher.Get())
                {
                    returnValue.Add(queryObj[<span style="color: #a31515;">"ComputerName"</span>].ToString());
                }
            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">ManagementException </span>e)
            {
                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[GetNLBComputers] {0}"</span>, e.Message));
                <span style="color: blue;">return null</span>;
            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">Exception </span>gex)
            {
                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[GetNLBComputers] {0}"</span>, gex.Message));
                <span style="color: blue;">return null</span>;
            }

            <span style="color: blue;">return </span>returnValue;
        }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Method which actually stops or starts a NLB Node; if stateStopped paramaters is true, the node will be started and vica versa
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="serverNode"&gt;</span><span style="color: green;">Node to perform action on</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;param name="stateStopped"&gt;</span><span style="color: green;">True if current state is stopped</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;returns&gt;</span><span style="color: green;">True if node was succesfully stopped/started</span><span style="color: gray;">&lt;/returns&gt;
        </span><span style="color: blue;">public bool </span>SetNlbNodeState (<span style="color: blue;">string </span>serverNode, <span style="color: blue;">bool </span>stateStopped)
        {
            <span style="color: green;">//Set RemoteComputer
            </span>RemoteComputer = serverNode;
            <span style="color: blue;">bool </span>ReturnValue = <span style="color: blue;">false</span>;
            <span style="color: blue;">try
            </span>{
                <span style="color: green;">//WMI Connection and Scope
                </span><span style="color: #2b91af;">ManagementScope </span>WmiScope = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementScope</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">@"{0}rootMicrosoftNLB"</span>, serverNode), WmiConnectionOption);

                <span style="color: green;">//WMI Query
                </span><span style="color: #2b91af;">ObjectQuery </span>WmiQuery = <span style="color: blue;">new </span><span style="color: #2b91af;">ObjectQuery</span>(<span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"SELECT * FROM MicrosoftNLB_Node WHERE ComputerName ='{0}'"</span>, serverNode));

                <span style="color: green;">//Actual 'wmi worker'
                </span><span style="color: #2b91af;">ManagementObjectSearcher </span>searcher = <span style="color: blue;">new </span><span style="color: #2b91af;">ManagementObjectSearcher</span>(WmiScope, WmiQuery);

                    <span style="color: blue;">foreach </span>(<span style="color: #2b91af;">ManagementObject </span>queryObj <span style="color: blue;">in </span>searcher.Get())
                    {
                        <span style="color: blue;">int </span>StateValue = -1;
                        <span style="color: blue;">int </span>NodeStatusCode = 0;
                        <span style="color: blue;">string </span>FriendeNodeStatus = <span style="color: blue;">string</span>.Empty;

                        <span style="color: green;">//Get NLB Node State
                        </span><span style="color: blue;">if</span>(<span style="color: blue;">int</span>.TryParse(queryObj[<span style="color: #a31515;">"StatusCode"</span>].ToString(),<span style="color: blue;">out </span>NodeStatusCode))
                        {
                            <span style="color: green;">//Determine friendly name of NLB Node State 
                            </span>FriendeNodeStatus = GetFriendlyNlbNodeStatusCode(NodeStatusCode);
                        }

                        <span style="color: blue;">if </span>(stateStopped)
                        {
                            <span style="color: green;">//Only stop if started
                            </span><span style="color: blue;">if</span>(FriendeNodeStatus.ToUpper() != <span style="color: #a31515;">"STOPPED"</span>)
                            {
                            <span style="color: blue;">if </span>(<span style="color: blue;">int</span>.TryParse(queryObj.InvokeMethod(<span style="color: #a31515;">"Stop"</span>, <span style="color: blue;">null</span>).ToString(), <span style="color: blue;">out </span>StateValue))
                            {
                                ReturnValue = <span style="color: blue;">true</span>;                               

                            }
                            }
                        }
                        <span style="color: blue;">else
                        </span>{
                            <span style="color: green;">//Only start if STOPPED
                            </span><span style="color: blue;">if </span>(FriendeNodeStatus.ToUpper() == <span style="color: #a31515;">"STOPPED"</span>)
                            {
                                <span style="color: blue;">if </span>(<span style="color: blue;">int</span>.TryParse(queryObj.InvokeMethod(<span style="color: #a31515;">"Start"</span>, <span style="color: blue;">null</span>).ToString(), <span style="color: blue;">out </span>StateValue))
                                {
                                    ReturnValue = <span style="color: blue;">true</span>;                                   

                                }
                            }
                        }

                    }

            }

            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">ManagementException </span>e)
            {
                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[SetNlbNodeState] {0}"</span>, e.Message));

            }
            <span style="color: blue;">catch </span>(<span style="color: #2b91af;">Exception </span>gex)
            {
                <span style="color: green;">//log exception
                </span><span style="color: #2b91af;">EventLogManager</span>.WriteError(<span style="color: #a31515;">"WmiMonitor"</span>, <span style="color: #2b91af;">String</span>.Format(<span style="color: #a31515;">"[SetNlbNodeState] {0}"</span>, gex.Message));

            }

            <span style="color: blue;">return </span>ReturnValue;
        }
        <span style="color: blue;">#endregion 
        #region </span>Private Methods
        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Method which performs a friendly lookup of possible NLB States
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="statusCode"&gt;</span><span style="color: green;">Original status code</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;returns&gt;</span><span style="color: green;">Friendly status code description</span><span style="color: gray;">&lt;/returns&gt;
        </span><span style="color: blue;">private string </span>GetFriendlyNlbNodeStatusCode(<span style="color: blue;">int </span>statusCode)
        {
            <span style="color: blue;">switch </span>(statusCode)
            {
                <span style="color: blue;">case </span>0:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"Node is remote. The StatusCode value cannot be retrieved."</span>;
                <span style="color: blue;">case </span>1005:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"STOPPED"</span>;
                <span style="color: blue;">case </span>1006:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"CONVERGING"</span>;
                <span style="color: blue;">case </span>1007:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"CONVERGED"</span>;
                <span style="color: blue;">case </span>1008:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"CONVERGED DEFAULT HOST"</span>;
                <span style="color: blue;">case </span>1009:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"DRAINING"</span>;
                <span style="color: blue;">case </span>1013:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"SUSPENDED"</span>;
                <span style="color: blue;">default</span>:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"UNKNOWN"</span>;

            }

       }

        <span style="color: gray;">/// &lt;summary&gt;
        /// </span><span style="color: green;">Method which performs a friendly lookup of possible ApplicationPool States
        </span><span style="color: gray;">/// &lt;/summary&gt;
        /// &lt;param name="stateCode"&gt;</span><span style="color: green;">Original state code</span><span style="color: gray;">&lt;/param&gt;
        /// &lt;returns&gt;</span><span style="color: green;">Friendly state code description</span><span style="color: gray;">&lt;/returns&gt;
        </span><span style="color: blue;">private string </span>GetFriendlyApplicationPoolState(<span style="color: blue;">int </span>stateCode)
        {
            <span style="color: blue;">switch</span>(stateCode)
            {
                <span style="color: blue;">case </span>0:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"Starting"</span>;
                <span style="color: blue;">case </span>1:
                       <span style="color: blue;">return </span><span style="color: #a31515;">"Started"</span>;
                <span style="color: blue;">case </span>2:
                       <span style="color: blue;">return </span><span style="color: #a31515;">"Stopping"</span>;
                <span style="color: blue;">case </span>3:
                       <span style="color: blue;">return </span><span style="color: #a31515;">"Stopped"</span>;
                <span style="color: blue;">case </span>4:
                       <span style="color: blue;">return </span><span style="color: #a31515;">"Unknown"</span>;
                <span style="color: blue;">default</span>:
                    <span style="color: blue;">return </span><span style="color: #a31515;">"Undefined value"</span>;
            }

        }

        <span style="color: blue;">#endregion
    </span>}
}</pre>
&nbsp;
<h2>Closing Note</h2>
So this sums up tackling our nagging problem on how to ensure that a NLB node is disabled in case a website or application pool is malfunctioning.

&nbsp;

In case you are interested in the source code including a sample windows service application please feel free to send me an email (<a href="mailto:info@brauwers.nl">info@brauwers.nl</a>) and I’ll send it to you

&nbsp;

Well I hope you enjoyed this read until the next time.

&nbsp;

Kind regards

&nbsp;

René