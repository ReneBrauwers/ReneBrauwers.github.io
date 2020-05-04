---
ID: 2832
post_title: 'How to: BizTalk Server 2013 Sysprep Development Image'
post_name: >
  how-to-biztalk-server-2013-sysprep-development-image
author: Rene Brauwers
post_date: 2013-07-30 20:28:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2013/07/30/how-to-biztalk-server-2013-sysprep-development-image/
published: true
tags:
  - BizTalk
  - BizTalk 2013
  - How to
  - sysprep
categories:
  - BizTalk
---
<p>&nbsp;</p>
<p>So you want to sysprep your BizTalk Server 2013 Development Environment? Well this might just be your lucky day. This post will detail the steps required in order for you to create your sysprep BizTalk Server 2013 Development environment.</p>
<blockquote><p><strong>Update2! Due to several requests; I&#8217;ve created a zip file containing all required script files and a document highlighting the required steps in order to sysprep an BizTalk Server 2013 (Standalone) image. You can download it <a href="https://blog.brauwers.nl/?attachment_id=2982">here</a></strong></p>
<p>Update! I&#8217;ve made a small change to the sysprep.xml file; fixing an issue with regards that you had would be prompted to agree with the fact that you were changing the powershell execution policy; it now includes the additional switch -force which will take care of this prompt. Besides this I added an additional script which will remove the product key and will prompt you at then end to enter a new product key. Please note that this last functionality is by default uncommented in the sysprep.xml file</p></blockquote>
<h2>Ingredients</h2>
<p>There is actually only one pre-requisite and it consists of the that you have already ‘build’ your own (Single-Server) BizTalk Server 2013 Development Environment; in short a <b><i><span style="text-decoration: underline">clean, lean and mean</span> </i></b>VM which contains:</p>
<ol>
<li>Windows Server 2012 <b><span style="text-decoration: underline">installed and configured </span></b></li>
<li>SQL Server 2012 <b><span style="text-decoration: underline">installed and configured</span></b></li>
<li>Visual Studio 2012 <b><span style="text-decoration: underline">+ updates</span></b></li>
<li>BizTalk Server 2013 <b><span style="text-decoration: underline">installed and configured</span></b></li>
<li>Other tools you might find handy</li>
</ol>
<p>In case you don’t have an Environment available or need assistance creating one; I suggest to check out this <a href="http://sandroaspbiztalkblog.wordpress.com/2013/05/05/biztalk-2013-installation-and-configuration-important-considerations-before-set-up-the-server-part-1/">link</a> which will redirect you to Sandro Pereira BizTalk Blog.</p>
<p>&nbsp;</p>
<h2>BizTalk Server Un-configuration</h2>
<p>This chapter will explain how to export the current BizTalk Server Configuration settings; once these settings have been exported we will proceed with a complete un-configuration of BizTalk Server (Including dropping all BizTalk Databases). If you already know how to do this, feel free to skip this chapter, if not well read on…</p>
<p>&nbsp;</p>
<p>1. Start your VM and log on with your Administrator Account</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image002.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image002" alt="clip_image002" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image002_thumb.jpg" width="244" height="183" border="0" /></a></p>
<p>&nbsp;</p>
<p>2. Once logged on, press the windows key and once in the ‘ type “BizTalk Server Configuration”, select it and press enter</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image004.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image004" alt="clip_image004" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image004_thumb.jpg" width="244" height="113" border="0" /></a></p>
<p>&nbsp;</p>
<p>3. The BizTalk Server Configuration screen will pop up. Now choose the option Export Configuration and save the resulting xml file to your desktop (I named it: BizTalk_Standalone_Configuration.xml)</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image006.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image006" alt="clip_image006" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image006_thumb.jpg" width="244" height="175" border="0" /></a></p>
<p>&nbsp;</p>
<p>4. Next step is to unconfigure all installed BizTalk Features. In order to do so click on ‘Unconfigure Features’. Select all features and once done press ok and follow the subsequent instructions displayed</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image008.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image008" alt="clip_image008" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image008_thumb.jpg" width="244" height="175" border="0" /></a></p>
<p>&nbsp;</p>
<p>5. Now wait, and once completed press Finish and close the Configuration Wizard</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image009.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image009" alt="clip_image009" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image009_thumb.png" width="244" height="185" border="0" /></a></p>
<p>&nbsp;</p>
<p>6. Now press the windows key and type ‘SQL Server Management Studio’. Select it and press enter.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image011.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image011" alt="clip_image011" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image011_thumb.jpg" width="244" height="105" border="0" /></a></p>
<p>&nbsp;</p>
<p>7. Now connect to your <b><i>LOCAL</i></b><i> SQL Server using ‘Database Engine’ as Server-Type </i></p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image013.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image013" alt="clip_image013" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image013_thumb.jpg" width="244" height="149" border="0" /></a></p>
<p>&nbsp;</p>
<p>8. Expand databases, and notice the databases related to BizTalk</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image015.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image015" alt="clip_image015" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image015_thumb.jpg" width="240" height="244" border="0" /></a></p>
<p>&nbsp;</p>
<p>9. Now delete all the BizTalk related databases one by one. This is done by right-clicking on one of the databases</p>
<p>“(in my case: BAMArchive, BAMPrimaryImport, BAMStarSchema, BizTalkDTADb, BizTalkMgmtDb, BizTalkMsgBoxDb, BizTalkRuleEngineDb, SSODB ) “</p>
<p>&nbsp;</p>
<p>10. A popup will show ensure you’ve checked the ‘Close existing connections’ checkbox and then press OK which will result in the database removal.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image017.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image017" alt="clip_image017" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image017_thumb.jpg" width="244" height="192" border="0" /></a></p>
<p>&nbsp;</p>
<p>11. Now repeat step 9 and 10 for the other BizTalk databases</p>
<p>&nbsp;</p>
<p>12. Now that all databases have been deleted, open the Security folder and it’s Logins folder</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image018.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image018" alt="clip_image018" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image018_thumb.png" width="201" height="244" border="0" /></a></p>
<p>&nbsp;</p>
<p>13 Now delete all the BizTalk related user groups and accounts one by one. This is done by right-clicking on a group/(service)account and selecting delete</p>
<blockquote><p>in my case: BizTalk Application Users,BizTalk Isolated Host Users, BizTalk Server Administrators, BizTalk Server B2B Operators, BizTalk Server Operators, SSO Administrators, svc-bts-bam-ws and svc-bts-bre</p></blockquote>
<p>&nbsp;</p>
<p>14 A popup will show press OK which will result in the database removal of the database group or user</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image020.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image020" alt="clip_image020" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image020_thumb.jpg" width="244" height="193" border="0" /></a></p>
<p>&nbsp;</p>
<p>15 Now repeat step 13 and 14 for the other BizTalk Groups and Service Accounts</p>
<p>&nbsp;</p>
<p>16 Now that all groups and accounts have been deleted, open the SQL Server Agent node and it’s Jobs folder</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image021.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image021" alt="clip_image021" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image021_thumb.png" width="221" height="244" border="0" /></a></p>
<p>&nbsp;</p>
<p>17 Now delete all the BizTalk related jobs one by one. This is done by right-clicking on a job and selecting delete</p>
<blockquote><p>in my case: BizTalk Backup Server, CleanupBTFExpiredEntriesJob_BizTalkMgmtDb, DTA Purge and Archive, MessageBox_DeadProcesses_Cleanup_BizTalkMsgBoxDb, MessageBox_Message_Cleanup_BizTalkMsgBoxDb, MessageBox_Message_ManageRefCountLog_BizTalkMsgBoxDb, MessageBox_Parts_Cleanup_BizTalkMsgBoxDb, Monitor BizTalk Server, Operations_OperateOnInstances_OnMaster_BizTalkMsgBoxDb, PurgeSubscriptionsJob_BizTalkMsgBoxDb, Rules_Database_Cleanup_BizTalkRuleEngineDb, TrackedMessages_Copy_BizTalkMsgBoxDb</p>
<p>&nbsp;</p></blockquote>
<p>18 A popup will show press OK which will result in the database removal of the job in question</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image023.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image023" alt="clip_image023" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image023_thumb.jpg" width="244" height="194" border="0" /></a></p>
<p>&nbsp;</p>
<p>19 Now repeat step 17 and 18 for the other BizTalk Jobs</p>
<p>&nbsp;</p>
<p>20 Now disconnect</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image025.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image025" alt="clip_image025" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image025_thumb.jpg" width="244" height="98" border="0" /></a></p>
<p>&nbsp;</p>
<p>21 Now press the connect tab and select Analysis Services and logon.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image026.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image026" alt="clip_image026" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image026_thumb.png" width="244" height="59" border="0" /></a></p>
<p>&nbsp;</p>
<p>22. Expand the folder Databases and right click on the BizTalk related Analysis Dbs and select delete</p>
<p>“(in my case: BAMAnalysis)”</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image028.jpg"><img style="padding-top: 0px;padding-left: 0px;padding-right: 0px;border: 0px" title="clip_image028" alt="clip_image028" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image028_thumb.jpg" width="244" height="194" border="0" /></a></p>
<p>&nbsp;</p>
<p>23. A popup will show ensure you’ve checked the ‘Continue deleting objects after error’ option and then press OK which will result in the database removal.</p>
<p>&nbsp;</p>
<p>24. Now close SQL Server Management Studio</p>
<p>&nbsp;</p>
<h2>Hacking the BizTalk Export File</h2>
<p>This chapter will explain how to manually change the settings in the exported BizTalk configuration file. If you already know how to do this, feel free to skip this chapter, if not well read on…</p>
<p>&nbsp;</p>
<p>1. Before you un-configured your BizTalk Server you’ve exported the configuration settings, well now it’s time to open this file and make some modifications to it. Below you will see a ‘Large’ screenshot highlighting the pieces which should be modified and below the image the changes are highlighter per line number. (or alternatively download my exported <a title="https://blog.brauwers.nl/?attachment_id=2192" href="https://blog.brauwers.nl/?attachment_id=2192" target="_blank" rel="noopener noreferrer">configuration settings</a>)</p>
<hr align="left" size="1" width="33%" />
<p><a name="_msocom_1"></a></p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/image.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/image_thumb.png" width="65" height="244" border="0" /></a></p>
<p>&nbsp;</p>
<blockquote><p>Line 12: Set the SSO backup file location<br />
Line 15: Set the Password used to protect the secret backup file<br />
Line 18: Set the Password used to protect the secret backup file<br />
Line 21: Set a reminder for the password</p>
<p>line 27: Should be already filled in with the username used for the SSO Service<br />
line 28: Ensure this field is empty<br />
line 29: Enter the password belonging to the service account mentioned in line 27</p>
<p>line 32: Ensure the Server field contains the value .<br />
line 41: Ensure the Server field contains the value .</p>
<p>line 47: [CREATE OR JOIN BIZTALK GROUP] Ensure that the attribute Default has the value &#8220;Create&#8221;<br />
line 48: [CREATE OR JOIN BIZTALK GROUP] Ensure that the Answer element contains the attribute SELECTED with value YES</p>
<p>line 50: Ensure the Server field contains the value .<br />
line 57: Ensure the Server field contains the value .</p>
<p>line 73: [CREATE OR JOIN BIZTALK GROUP] Ensure that the Answer Element does not contain an attribute SELECTED</p>
<p>line 94: Should already be filled out with the username used for the NT Service for the In-process Host Instance<br />
line 95: Ensure this field is empty<br />
line 96: Enter the password belonging to the service account mentioned in line 94</p>
<p>line 118: Should already be filled out with the username used for the NT Service for the BizTalk Isolated Host Instance<br />
line 119: Ensure this field is empty<br />
line 120: Enter the password belonging to the service account mentioned in line 118</p>
<p>line 128: Ensure the Server field contains the value .</p>
<p>line 135: Should already be filled out with the username used for the NT Service for the BizTalk Rule Engine<br />
line 136: Ensure this field is empty<br />
line 137: Enter the password belonging to the service account mentioned in line 135</p>
<p>line 143: Ensure the Server field contains the value .<br />
line 150: Ensure the Server field contains the value .<br />
line 159: Ensure the Server field contains the value .<br />
line 166: Ensure the Server field contains the value .</p>
<p>line 175: [BAM ALERTS CONFIG] Ensure that the attribute Default has the value &#8220;No&#8221;<br />
line 176: [BAM ALERTS CONFIG] Ensure that the Answer Element does not contain an attribute SELECTED<br />
line 195: [BAM ALERTS CONFIG] Ensure that the Answer element contains the attribute SELECTED with value YES</p>
<p>line 197: [BAM TOOLS] Ensure that the attribute Default has the value &#8220;No&#8221;<br />
line 198: [BAM TOOLS] Ensure that the Answer element contains the attribute SELECTED with value YES<br />
line 199: [BAM TOOLS] Ensure that the Answer Element does not contain an attribute SELECTED</p>
<p>line 204: Should already be filled out with the username used for the NT Service for the BAM Management Web Service User<br />
line 205: Ensure this field is empty<br />
line 206: Enter the password belonging to the service account mentioned in line 204</p>
<p>line 209: Should already be filled out with the username used for the NT Service for the BAM Application Pool Account<br />
line 210: Ensure this field is empty<br />
line 211: Enter the password belonging to the service account mentioned in line 209</p></blockquote>
<p>&nbsp;</p>
<p>2. Once you’ve made the changes, save the file.</p>
<p>&nbsp;</p>
<p>3. Now open the BizTalk Configuration Tool</p>
<p>&nbsp;</p>
<p>4. Fill out the initial details (Caution: use custom configuration) and once done press Configure</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/image1.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/image_thumb1.png" width="244" height="196" border="0" /></a></p>
<p>&nbsp;</p>
<p>5. Now click on Import Configuration</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0024.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image002[4]" alt="clip_image002[4]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0024_thumb.jpg" width="244" height="176" border="0" /></a></p>
<p>&nbsp;</p>
<p>6. Select the ‘modified file’ and press ok. The configurations will now be imported, and once done you should see a message box stating that same fact</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0044.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image004[4]" alt="clip_image004[4]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0044_thumb.jpg" width="244" height="176" border="0" /></a></p>
<p>&nbsp;</p>
<p>7. Next step is to validate all settings (see screenshots)</p>
<p>&nbsp;</p>
<p>8. Now Press Apply Configuration to test if everything works</p>
<p>&nbsp;</p>
<p>9. If it worked than hurrah! Now go and un-configure everything once again J (see beginning of this blog-post, yes this includes SQL Server etc..)</p>
<p>&nbsp;</p>
<p>10. If it failed, bummer! Well go and un-configure and make modifications to the import file and retry steps 1 till 10</p>
<p>&nbsp;</p>
<h2>Golden Image</h2>
<p>Well we are almost done, however now we need to care of a mechanism which will ensure that our BizTalk can be automagically configured, and for this we will use the all so mighty and sometimes underrated Windows Task Scheduler!</p>
<p>&nbsp;</p>
<p>Well the following trick I picked up in this <span style="text-decoration: underline"><a href="http://jeremiedevillard.wordpress.com/2013/05/06/one-touch-biztalk-configuration-in-windows-azure-virtual-machine/" target="_blank" rel="noopener noreferrer">blog-post</a></span>, and I used the same trick in my last blog-post. This trick involves the following actions which you need to perform</p>
<p>&nbsp;</p>
<p>1. Download and unpack <a href="https://blog.brauwers.nl/?attachment_id=2172" target="_blank" rel="noopener noreferrer">this archive</a></p>
<p>&nbsp;</p>
<p>2. Copy the contents to c:Program Files (x86)Microsoft BizTalk Server 2013</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0026.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image002[6]" alt="clip_image002[6]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0026_thumb.jpg" width="244" height="115" border="0" /></a></p>
<p>&nbsp;</p>
<p>3. Create the following directory c:Scripts</p>
<p>&nbsp;</p>
<p>4. Download and unpack <a href="https://blog.brauwers.nl/?attachment_id=2842" target="_blank" rel="noopener noreferrer">this archive</a> and copy the contents to the newly created directory c:Scripts</p>
<p>&nbsp;</p>
<p>5. No press the windows key, type ‘Scheduled Tasks’ and double click the application</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0046.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image004[6]" alt="clip_image004[6]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0046_thumb.jpg" width="244" height="109" border="0" /></a></p>
<p>&nbsp;</p>
<p>6. The Task Scheduler will now open.</p>
<p>&nbsp;</p>
<p>7. From the menu-bar select Action -&gt; Import Task…</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image005.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image005" alt="clip_image005" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image005_thumb.png" width="244" height="150" border="0" /></a></p>
<p>&nbsp;</p>
<p>8. Now in the file-picker browse to c:Scripts and select the following file to import “BizTalk Configuration Scheduled Task”</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image006.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image006" alt="clip_image006" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image006_thumb.png" width="244" height="94" border="0" /></a></p>
<p>&nbsp;</p>
<p>9. The scheduled task will open, now click on the button “Change User or Group…:”</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0084.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image008[4]" alt="clip_image008[4]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0084_thumb.jpg" width="244" height="184" border="0" /></a></p>
<p>&nbsp;</p>
<p>10. Enter the administrator user name and press ok</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0094.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image009[4]" alt="clip_image009[4]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0094_thumb.png" width="244" height="146" border="0" /></a></p>
<p>&nbsp;</p>
<p>11. Ensure the checkbox “Run with highest privileges” is checked and the administrator user name is used when the task is run and press Ok</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0114.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image011[4]" alt="clip_image011[4]" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image0114_thumb.jpg" width="244" height="184" border="0" /></a></p>
<p>&nbsp;</p>
<p>12. Now enter the password for the administrator user.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image012.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image012" alt="clip_image012" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image012_thumb.png" width="244" height="197" border="0" /></a></p>
<p>&nbsp;</p>
<p>13. Close the Task Scheduler</p>
<p>&nbsp;</p>
<p>14. Congratz your ‘Golden Image’ is ready. Now might be a good time to shutdown the server and copy this VHD and store it in a safe place <img class="wlEmoticon wlEmoticon-smile" style="border-style: none" alt="Smile" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/wlEmoticon-smile.png" /></p>
<p>&nbsp;</p>
<h2>Sysprep</h2>
<p>Now we are ready for our last step, so after you have made a copy of your ‘Golden Image’ it is time to fire-up your VM and perform these last few steps.</p>
<p>1. Once the VM has started and you logged on as Administrator go to c:Scripts</p>
<p>&nbsp;</p>
<p>2. Open the sysprep.xml file and make a few changes. Once more I’ve highlighted the lines which require a change.</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/image2.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="image" alt="image" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/image_thumb2.png" width="121" height="244" border="0" /></a></p>
<p>&nbsp;</p>
<blockquote><p>Line 22: Replace with a valid Windows Server 2012 product Key!<br />
Line 23: Replace with your Organization name<br />
Line 24: Replace with the registered owner<br />
Line 29: Enter the name of your Time Zone<br />
Line 64: Replace with your Organization name<br />
Line 65: Replace with the registered owner<br />
Line 68: Replace with the administrator user password<br />
Line 74: Replace with the administrator user password<br />
Line 80: Replace with the administrator username<br />
Line 99: Replace with the current computer name<br />
Line 125: Enter the name of your Time Zone</p></blockquote>
<p>&nbsp;</p>
<p>3. Now save your file and go back to the c:Scripts directory and click on startSysprep.bat</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image001.png"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image001" alt="clip_image001" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image001_thumb.png" width="244" height="182" border="0" /></a></p>
<p>&nbsp;</p>
<p>4. Your image will be SYSPREPPED <img class="wlEmoticon wlEmoticon-smile" style="border-style: none" alt="Smile" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/wlEmoticon-smile.png" /> and once done your VM will shutdown</p>
<p><a href="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image003.jpg"><img style="padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px;border: 0px" title="clip_image003" alt="clip_image003" src="https://brauwers-nl.azureedge.net/images/blog/2013/07/clip_image003_thumb.jpg" width="244" height="138" border="0" /></a></p>
<p>&nbsp;</p>
<p>Enjoy!</p>
<ul>
<li><a href="http://stretchedclub.accountant/">generic levitra</a></li>
<li><a href="http://kamerakaufen.cricket/">website</a></li>
<li><a href="http://cilaisgenericcialisfordailyuse20mg.accountant">viagra legal buy</a></li>
<li><a href="http://sildenafilervaringen.men/">buy sildenafil online</a></li>
<li><a href="http://genericforviagra.accountant/">generic cialis pas cher</a></li>
<li><a href="http://sildenafil100mgblab.accountant/">viagra 50 mg 4 tabletten</a></li>
<li><a href="http://sexyfeelingtabletsnamesmedicine.accountant/">view more</a></li>
<li><a href="http://cheapestheretadalafil.org/">generic cialis 5mg daily</a></li>
</ul>