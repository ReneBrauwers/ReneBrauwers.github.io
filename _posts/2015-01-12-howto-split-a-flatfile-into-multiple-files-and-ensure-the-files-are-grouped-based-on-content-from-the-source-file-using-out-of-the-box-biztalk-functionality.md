---
ID: 4792
post_title: 'Howto: Split a FlatFile into multiple files and ensure the files are grouped based on content from the source file using out of the box BizTalk functionality'
post_name: >
  howto-split-a-flatfile-into-multiple-files-and-ensure-the-files-are-grouped-based-on-content-from-the-source-file-using-out-of-the-box-biztalk-functionality
author: Rene Brauwers
post_date: 2015-01-12 00:25:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2015/01/12/howto-split-a-flatfile-into-multiple-files-and-ensure-the-files-are-grouped-based-on-content-from-the-source-file-using-out-of-the-box-biztalk-functionality/
published: true
tags:
  - Assembler
  - CSV
  - Flatfile
  - Grouping
  - Promoted Properties
  - Sequentual Convoy
categories:
  - BizTalk
---
<h1>First of all…</h1>
<p>…a belated Happy New Year! I know it has been quiet on this blog for quiet some time, but I’ll clear this up in the near future once things are certain for a 100% <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none;border-left-style: none;border-bottom-style: none;border-right-style: none" alt="Winking smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/wlEmoticon-winkingsmile.png"></p>
<h1>&nbsp;</h1>
<h1>Well let’s get to it.</h1>
<p>Recently a colleague of mine, André&nbsp; Ruiter <a href="https://twitter.com/andreruiter67" target="_blank" rel="noopener noreferrer">@AndreRuiter67</a> , asked my view on a particular challenge involving flatfiles. This challenge in short consisted of:</p>
<p>How would one based on a flatfile output x files, where the&nbsp; files would contain grouped data based on a specific value in the original flatfile. </p>
<p>&nbsp;</p>
<p>My response was, as most of my response, as I enjoy using OOB functionality (thus no code): use a flatfile disassembler in a custom receive pipeline add a map on the receive port in which the inbound file is being transformed to an internal format. Within this internal document ensure to promote the field(s) one wants to group on (correlate). Then use an orchestration which subscribes to the internal message(s) and implement the sequential convoying pattern and aggregate the results to an output format and lastly store the end result to disk.</p>
<p>&nbsp;</p>
<p>As you’ve been reading the above you might go like; do what? So for the readers convenience I will walk through an example and explain the required steps. In case it makes sense, well you now know how to implement it, so go ahead move on… Nothing to see anymore <img class="wlEmoticon wlEmoticon-winkingsmile" style="border-top-style: none;border-left-style: none;border-bottom-style: none;border-right-style: none" alt="Winking smile" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/wlEmoticon-winkingsmile.png"></p>
<p>&nbsp;</p>
<h1>Walkthrough</h1>
<p>Before we start, I assume you have at least a basic understanding of BizTalk, as such I will not explain all things, although I will have a step by step instruction relating to the flat file generation as well as the sequential convoy. Having said this you ought to be able to follow all steps and reproduce the steps involved all by yourself and in case that doesn’t work out for you, I’ve added the source which you can download <a href="http://bit.ly/1AN6arB" target="_blank" rel="noopener noreferrer">here</a>. </p>
<p>&nbsp;</p>
<h2></h2>
<h2>The scenario</h2>
<p>In our example we will receive a comma delimited file containing data received from smart energy readers. Each line contains data like;</p>
<p>-customer id</p>
<p>-date of the energy reading</p>
<p>-energy consumption value since last reading</p>
<p>-name of the energy company (to which the reader belongs and sends out the bills)</p>
<p>&nbsp;</p>
<h6>Example file contents</h6>
<div id="scid:9D7513F9-C04C-4721-824A-2B34F0212519:5574375d-4cb1-411c-bfc5-6d17f35a2b55" class="wlWriterEditableSmartContent" style="float: none;padding-bottom: 0px;padding-top: 0px;padding-left: 0px;margin: 0px;padding-right: 0px">
<pre style="width: 400px;height: 300px;background-color:White;overflow: auto"><div><!--

Code highlighting produced by Actipro CodeHighlighter (freeware)
http://www.CodeHighlighter.com/

--><span style="color: #000000">customerId,readingDate,consumption,energyCompanyName
</span><span style="color: #800080">1</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">12</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Free Energy INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">2</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">8</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">3</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">23</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Windmills INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">4</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">5</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Sun Unlimited</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">5</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">6</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Free Energy INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">6</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">3</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Free Energy INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">7</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">12</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">8</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">8</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Windmills INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">9</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">9</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Windmills INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">10</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">26</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Sun Unlimited</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">11</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">24</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">12</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">17</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Go Nuclear</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">13</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">11</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">14</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">9</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Windmills INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">15</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">0</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Free Energy INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">16</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">5</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Go Nuclear</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">17</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">12</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Windmills INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">18</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">43</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Sun Unlimited</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">19</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">35</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">20</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">23</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Free Energy INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">21</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">2</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Sun Unlimited</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">22</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">14</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Free Energy INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">23</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">13</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">24</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">9</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Go Nuclear</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">25</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">26</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Windmills INC</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">26</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">27</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Sun Unlimited</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">27</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">25</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Go Nuclear</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">28</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">31</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">29</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">4</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Water Works LTD</span><span style="color: #800000">&quot;</span><span style="color: #000000">
</span><span style="color: #800080">30</span><span style="color: #000000">,</span><span style="color: #800080">2015</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">-</span><span style="color: #800080">01</span><span style="color: #000000">,</span><span style="color: #800080">7</span><span style="color: #000000">,</span><span style="color: #800000">&quot;</span><span style="color: #800000">Sun Unlimited</span><span style="color: #800000">&quot;</span></div></pre>
<p><!-- Code inserted with Steve Dunn's Windows Live Writer Code Formatter Plugin.  http://dunnhq.com --></div>
<p>based on this file we need to split the source file into separate files grouped by energy company. </p>
<p>&nbsp;</p>
<p>Sounds easy doesn’t it? Well let’s get to it! </p>
<p>&nbsp;</p>
<h2>Create the to use schemas [Flat file header ]</h2>
<p>First of we will start with creating an xml presentation of the source flat file <em>Header</em>. For this we will use the BizTalk Flat File Wizard. </p>
<p>&nbsp;</p>
<h3>Step 1</h3>
<p>In the solution explorer of Visual Studio, select your BizTalk Project and add a new item <strong>[ Right Click -&gt; Add -&gt; New Item -&gt; Flat File Schema Wizard ] </strong>and add a descriptive name for the flatfile schema header you are about to create and click on the <strong>[ Add button ]</strong></p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb.png" width="641" height="444"></a></p>
<p>&nbsp;</p>
<h3>Step 2</h3>
<p>The BizTalk Flat File Wizard will appear. Now press the <strong>[ Next button]</strong> untill you see <strong>[ Flat File Information Screen ]</strong>. On this screen, <strong>[ browse ]</strong> to the csv file in question. Enter a name for the record in the <strong>[ Record Name ]</strong> input field. Leave the other options in tact and press the <strong>[ Next button ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image1.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb1.png" width="639" height="503"></a></p>
<p>&nbsp;</p>
<h3>Step 3</h3>
<p>You should now be on the <strong>[ Select Document Screen ]</strong>. On this screen, select the header&nbsp; <strong>[ The first line ] </strong>and press the <strong>[ Next button ]</strong>.</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image2.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb2.png" width="647" height="511"></a></p>
<p>&nbsp;</p>
<h3>Step 4</h3>
<p>At this point you should be on the <strong>[ Select Record Format Screen ]</strong>. On this screen, ensure you select that the record is being by means of a <strong>[ Delimiter Symbol ]</strong>. Once you’ve selected this item press the <strong>[ Next button ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image3.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb3.png" width="650" height="516"></a></p>
<p>&nbsp;</p>
<h3>Step 5</h3>
<p>The next screen which pops up allows you the select the <strong>[ Child Delimiter ]</strong> ensure that for you select the <strong>[ {CR/LF} ]</strong> option. Now press the <strong>[ Next Button ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image4.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb4.png" width="652" height="515"></a></p>
<p>&nbsp;</p>
<h3>Step 6</h3>
<p>Now you will be presented with the <strong>[ Child Elements ]</strong> screen. On this screen ensure that you change the <strong>[ Element Type ]</strong> from <strong>[ Field Element ]</strong> to <strong>[ Record ]</strong>. Once done press the <strong>[ Next Button ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image5.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb5.png" width="656" height="518"></a></p>
<p>&nbsp;</p>
<h3>Step 7</h3>
<p>So far all we have done is defined our record definition, the next few steps will define our header elements (our columns if you prefer). The screen which you will be presented with at this stage is the start of this process.&nbsp; In order to start press the <strong>[ Next Button ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image6.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb6.png" width="658" height="519"></a></p>
<p>&nbsp;</p>
<h3>Step 8</h3>
<p>The sceen <strong>[ Select Document Data ]</strong> allows you to select the actual data (headers elements). If you followed up on all the steps so far it would suffice to select the <strong>[ Next Button ]</strong>. In case you’re not sure ensure that you only have selected the actual data excluding the <u><strong>[ New line characters ]</strong>.</u></p>
<p><u></u>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image7.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb7.png" width="658" height="519"></a></p>
<p><font color="#333333"></font>&nbsp;</p>
<h3>Step 9</h3>
<p>Once again you will be presented with the <strong> [ Select Record Format Screen ]</strong>. On this screen, ensure you select that the record is being by means of a <strong>[ Delimiter Symbol ]</strong>. Once you’ve selected this item press the <strong>[ Next button ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image8.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb8.png" width="662" height="522"></a></p>
<p>&nbsp;</p>
<h3>Step 10</h3>
<p>The next screen which pops up allows you the select the <strong>[ Child Delimiter ]</strong> ensure that for you select the <strong>[ , ] (Comma)</strong> option. Now press the <strong>[ Next Button ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image9.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb9.png" width="664" height="524"></a></p>
<p>&nbsp;</p>
<h3>Step 11</h3>
<p>You will now be presented with the <strong>[&nbsp; Childs Elements ]</strong> screen which actually allows us to define the columns of the header. In our example we will make a few modification relating to the <strong>[ Element Name ]</strong> we will not change the <strong>[ Data Type</strong> <strong>] </strong>as we are defining our header section and we are currently only defining the header (column) names. For brevity see the screenshot below which depicts all changes I’ve made. Once you have made the changes press the <strong>[ Next Button ]</strong></p>
<p>&nbsp;</p>
<h6>Before changes</h6>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image10.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb10.png" width="672" height="530"></a></p>
<p>&nbsp;</p>
<h6>After changes</h6>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image11.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb11.png" width="676" height="533"></a></p>
<p>&nbsp;</p>
<h3>Step 12</h3>
<p>Congratulations at this point you have created your header structure, the end result should look similar to the image as depicted below. (note I’ve selected the Flat File tab, to display the non-xsd view)</p>
<p>&nbsp;</p>
<p>&lt;a href=&quot;http://blog <a href="http://biturlz.com/Cr4uy3i">les pilules de viagra</a>.brauwers.nl/wp-content/uploads/2015/01/image12.png"&gt;<img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb12.png" width="684" height="565"></a></p>
<p>&nbsp;</p>
<h2>Create the to use schemas [Flat file non header data]</h2>
<p>Now that we have defined our xml representation of our flat file header is time to define an xml representation of the <em>non header data</em>. For this we will once again use the BizTalk Flat File Wizard. The steps 1 to 13 we went thought earlier will have to repeated with a few <strong>[ Changes in Configuration</strong>&nbsp;<strong>]</strong>. As such I will only list those steps which are different. Yeah you are allowed to call me lazy <img class="wlEmoticon wlEmoticon-smilewithtongueout" style="border-top-style: none;border-left-style: none;border-bottom-style: none;border-right-style: none" alt="Smile with tongue out" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/wlEmoticon-smilewithtongueout.png">&nbsp;</p>
<h3>Step 2</h3>
<p>The BizTalk Flat File Wizard will appear. Now press the <strong>[ Next button]</strong> until you see <strong>[ Flat File Information Screen ]</strong>. On this screen, <strong>[ browse ]</strong> to the csv file in question. Enter a name for the record in the <strong>[ Record Name ]</strong> input field. Leave the other options in tact and press the <strong>[ Next button ]</strong>. Note I’ve named the <strong>[ Record Name ]</strong> EnergyReadings</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image13.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb13.png" width="689" height="544"></a></p>
<p>&nbsp;</p>
<h3>Step 3</h3>
<p>You should now be on the <strong>[ Select Document Screen ]</strong>. On this screen, select the <strong>[ The second line ] </strong>which contains the (repeating) data <strong>&nbsp;</strong>and press the <strong>[ Next button ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image14.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb14.png" width="695" height="549"></a></p>
<p>&nbsp;</p>
<h3>Step 6</h3>
<p>Now you will be presented with the <strong>[ Child Elements ]</strong> screen. On this screen ensure that you change the <strong>[ Element Type ]</strong> from <strong>[ Field Element ]</strong> to <strong>[ Repeating Record ]</strong>. Once done press the <strong>[ Next Button ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image15.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb15.png" width="701" height="553"></a></p>
<p>&nbsp;</p>
<h3>Step 11</h3>
<p>You will now be presented with the <strong>[&nbsp; Childs Elements ]</strong> screen which actually allows us to define the columns value. In our example we will make a few modification relating to the <strong>[ Element Name ]</strong> and the <strong>[ Data Type</strong> <strong>]</strong>. For brevity see the screenshot below which depicts all changes I’ve made. Once you have made the changes press the <strong>[ Next Button ]</strong></p>
<p>&nbsp;</p>
<h6>After changes</h6>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image16.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb16.png" width="704" height="555"></a></p>
<p>&nbsp;</p>
<p>Congratulations at this point you have created your data structure, however we will need to make some manual changes to the generated schema. This changes will ensure that we will instruct BizTalk to<strong>[ Auto Debatch ] </strong>the inbound records to single records (in case there are multiple data lines.)</p>
<p>&nbsp;</p>
<h3>Step 12</h3>
<p>In order to ensure that <strong>[ Auto Debatching</strong> <strong>]</strong> will happen we will need to do the following. <strong>[ Select the Schema Element ] </strong>of the newly generated schema and then in the <strong>[ Properties ]</strong> window ensure to change the following setting: <strong>[ Allow Message Breakup at InFix Root ] </strong>from<strong>&nbsp; [ False ]</strong>&nbsp; to <strong>[ True ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image17.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb17.png" width="717" height="400"></a></p>
<p>&nbsp;</p>
<h3>Step 13</h3>
<p>The last step we need to perform to enable <strong>[ Auto Debatching ]</strong> consists of changing the <strong>[ Max Occurs ]</strong>&nbsp; <strong>[ Property</strong> <strong>] </strong>of the <strong>[ Repeating ‘Element’ ] </strong>from being <strong>[ Unbound</strong> <strong>]</strong>&nbsp; to <strong>[ 1 ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image18.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb18.png" width="722" height="399"></a></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<h2>Create the to use schemas [Other]</h2>
<p>Now that we’ve created our schemas which represent the flat file definition, we can move on to creating the other schema’s we need. I will not go over the details on how to create these ‘normal’&nbsp; schemas instead I’ll list the schema’s required.</p>
<p>&nbsp;</p>
<h4>Property schema</h4>
<p>We start of with a definition of a simple property schema, this schema will only hold one field and will be named EnergyCompany. </p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image19.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;margin: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb19.png" width="191" height="79"></a></p>
<p>&nbsp;</p>
<p>If you need more information with regards to property schemas please click on this <a href="http://msdn.microsoft.com/en-us/library/aa561059.aspx" target="_blank" rel="noopener noreferrer">link</a>.</p>
<p>&nbsp;</p>
<h4>Internal schema: Reading</h4>
<p>This schema is our internal representation of a energy reading, and looks as depicted below. Please note that the element named <strong>[ CompanyName ] </strong>has been promoted, as such we can use it later on when we are about to implement or sequential convoy.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image20.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;margin: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb20.png" width="231" height="141"></a></p>
<p>&nbsp;</p>
<h4>Internal schema: EnergyReading</h4>
<p>This schema is the actual representation of the xml we will output and contains multiple readings on a per energy ompany basis. It has to be noted that this schema is a composite schema and as such it <strong>[ Imports ]</strong> the schema <strong>[ Reading ] </strong>(see 1). The other thing which has to be noted is the fact that the <strong>[ Reading ]</strong> element has it’s <strong>[ Max Occurs ]</strong> value set to unbounded.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image21.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb21.png" width="738" height="383"></a></p>
<p>&nbsp;</p>
<h2>Creation of the Receive Pipeline</h2>
<p>Now that all schemas have been created we can go ahead with the creation of a receive pipeline. Once again I will not dive into the nitty gritty details, but if you require more information please click on this <a href="http://msdn.microsoft.com/en-us/library/ee267879(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a></p>
<p>&nbsp;</p>
<p>So create a <strong>[Receive Pipeline ]</strong> and give it a meaning name, drag a <strong>[ Flat File Disassembler Component ]</strong> to the <strong>[ Design Surface ]</strong> and drop it in the <strong>[ Disassemble stage (1) ]</strong>. Now <strong>[ Click ]</strong>on the just added component and go to the <strong>[ Properties Windows ]</strong>. In this window ensure to select the earlier on created <strong>[ Flat File Header Schema ]</strong> for the <strong>[ Header Schema Property (2) ]</strong> and select the <strong>[Flat File Schema ]</strong> for the <strong>[ Document Schema Property (2) ]</strong>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image22.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb22.png" width="743" height="361"></a></p>
<p>&nbsp;</p>
<h2>Transformations</h2>
<p>At this point we can start with the required mappings we need. In total we will need 3 maps. The required maps are listed below. </p>
<p>&nbsp;</p>
<blockquote>
<p>Please note if you want to learn more with regards to mappings and advanced patterns (In my example everything is kept quit basic), I can only recommend that you download and start reading an ebook titled “BizTalk Mapping Patterns and Best Practices” which a friend of mine, Sandro Pereira <a href="https://www.twitter.com/sandro_asp" target="_blank" rel="noopener noreferrer">@sandro_asp</a>,&nbsp; and Microsoft Integration MVP put together for free. Go <a href="http://sandroaspbiztalkblog.wordpress.com/2014/09/28/biztalk-mapping-patterns-and-best-practices-book-free-released/" target="_blank" rel="noopener noreferrer">here</a> to download it</p>
</blockquote>
<p>&nbsp;</p>
<h4>EnergyReadingFF_TO_Reading</h4>
<p>This mapping will be used on the receive port and will map the generated inbound flat file xml structure to our single reading file.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image23.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb23.png" width="750" height="209"></a></p>
<p>&nbsp;</p>
<h4>Reading_TO_EnergyReadings</h4>
<p>This mapping will be used in our orchestration, which implements a sequential convoy, and maps the single reading file to the energy readings </p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image24.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb24.png" width="760" height="209"></a></p>
<h4>&nbsp;</h4>
<h4>Reading_Readings_TO_AggregatedEnergyReadings</h4>
<p>This mapping will be used in our orchestration which implements a sequential convoy as well, and maps all results together.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image25.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb25.png" width="762" height="349"></a></p>
<p>&nbsp;</p>
<h2>Sequential Convoy</h2>
<p>Before we can deploy our BizTalk Application there is one more thing we need to implement, and that’s a mechanism to output the grouped files. The way to implement this is using an orchestration and implement the <strong>[ Sequential Convoy ]</strong> pattern. Below a screenshot of the end result and I’ll go into the basic details using steps which refer to the screenshot below. In case you want to now more about the <strong>[ Sequential Convoy]</strong> pattern please click on this <a href="http://msdn.microsoft.com/en-us/library/aa561843.aspx" target="_blank" rel="noopener noreferrer">link</a>.</p>
<p>&nbsp;</p>
<p><a href="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image26.png"><img title="image" style="border-top: 0px;border-right: 0px;border-bottom: 0px;padding-top: 0px;padding-left: 0px;border-left: 0px;padding-right: 0px" border="0" alt="image" src="https://brauwersnl.blob.core.windows.net/images/uploads/2015/01/image_thumb26.png" width="760" height="816"></a></p>
<p>&nbsp;</p>
<h4>Step 1: rcvReading</h4>
<p>This receive shape ensures that messages with the message type <a href="http://FlatFileGrouping.Reading#Reading">http://FlatFileGrouping.Reading#Reading</a> are being subscribed to. These are the single reading messages as stated earlier. It has to be noted that we initialize a <strong>[ correlation Set ]</strong> this set will ensure that we actually will create a single process (Singleton) which subscribes not only to messages of the aforementioned messagetypes but to messages which contain the same value for the element CompanyName contained with the reading message.</p>
<p>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-US/library/ee253479(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on the <strong>[ Receive shape ]</strong></p>
<p>Click on this <a href="http://msdn.microsoft.com/en-us/library/aa560163.aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on <strong>[ Correlation Sets ]</strong> </p>
<p>&nbsp;</p>
<h4>Step 2: Init Timeout boolean</h4>
<p>This expression shape is used to initialize a boolean which is used later on in the process to indicate if the convoying process should be ended. The initial value here is set <strong>[ False ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-us/library/ee253557(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on the <strong>[ Expression Shape ]</strong></p>
<p>&nbsp;</p>
<h4>Step 3: Construct Energy Readings</h4>
<p>This construction block is used to to host the <strong>[ Reading_TO_EnergyReadings ]</strong> transformation, and as such initializes the actual message we will send out to disk containing the grouped contents with regards to the energy readings on a per company base</p>
<p>&nbsp;</p>
<p>Click on <a href="http://msdn.microsoft.com/en-us/library/ee253554(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">this</a> link for more information on the <strong>[ Construct Message Shape ]</strong></p>
<p>&nbsp;</p>
<h4>Step 4: Loop until timeout</h4>
<p>This loop ensures that the contained logic is being repeated as long as the previous initialized boolean is False. In our specific case the boolean is set to true once we have not received any more reading messages for 30 seconds.</p>
<p>&nbsp;</p>
<p>Click on <a href="http://msdn.microsoft.com/en-us/library/ee268264(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">this</a> link for more information on the <strong>[ Looping Shape ]</strong></p>
<p>&nbsp;</p>
<h4>Step 5: Listen</h4>
<p>This shape will enable us to receive other messages for a given time window. </p>
<p>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-US/library/ee253551(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on the <strong>[ Listen Shape ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<h4>Step 6: rcvSubsequentReadings</h4>
<p>This receive shape ensures that messages with the message type <a href="http://FlatFileGrouping.Reading#Reading">http://FlatFileGrouping.Reading#Reading</a> are being subscribed to. These are the single reading messages as stated earlier. It has to be noted that we follow a <strong>[ correlation Set ]</strong> this will ensure that we will receive any follow up messages without starting up a new service instance of this orchestration. Ie; if an instance of this orchestration is initiated and a message with has the value <em>Company Y</em> for the element CompanyName contained with the reading message is received it will enter the process at this point (and be further processed)</p>
<p>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-us/library/aa560163.aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on <strong>[ Correlation Sets ]</strong> </p>
<p>&nbsp;</p>
<h4>Step 7: Aggregate following reading to initial reading</h4>
<p>This construction block is used to to host the composite transformation <strong>[ Reading_Readings_TO_AggregatedEnergyReadings ]</strong>, and as such this map takes both the follow up reading message as well as the in step 3 constructed Energy Reading message and combines these messages to a temp message called AggregatedEnergyReadings.</p>
<p>&nbsp;</p>
<p>Click on <a href="http://msdn.microsoft.com/en-us/library/ee253554(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">this</a> link for more information on the <strong>[ Construct Message Shape ]</strong></p>
<p>Click on <a href="http://geekswithblogs.net/sthomas/archive/2005/02/04/21969.aspx" target="_blank" rel="noopener noreferrer">this</a> link for more information on<strong> [ Multi Part Mappings ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<h4>Step 8: Copy to EnergyReadings</h4>
<p>This message assignment shape is used to copy over output of the previous mapping (step 7) to the original Energy readings document.</p>
<p>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-us/library/ee253499(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on the <strong>[ Message Assignment Shape ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<h4>Step 9: Wait 30 seconds </h4>
<p>This delay shape will be activated once the listen shape has not received any messages for 30 seconds.</p>
<p>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-us/library/ee253483(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on the <strong>[ Delay Shape ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<h4>Step 10: Set bHasTimeout </h4>
<p>This expression shape is used to set the bHasTimeout&nbsp; boolean to <strong>[ True ]</strong> ensuring that we will exit the loop and are able to end to process eventually after sending out the energy readings message</p>
<p>&nbsp;</p>
<p><strong><font color="#333333"></font></strong> Click on this <a href="http://msdn.microsoft.com/en-us/library/ee253557(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information on the <strong>[ Expression Shape ]</strong></p>
<p><strong><font color="#333333"></font></strong>&nbsp;</p>
<h4>Step 11: sndAggregation</h4>
<p>This shape will actually send out the energy readings message, which at this time only contains data relating to a specific company,</p>
<p>&nbsp;</p>
<p>Click on this <a href="http://msdn.microsoft.com/en-us/library/ee253504(v=bts.10).aspx" target="_blank" rel="noopener noreferrer">link</a> for more information in the <strong>[ Send Shape ]</strong></p>
<p>&nbsp;</p>
<h3>Final Configuration</h3>
<p>At this point you will have created all the required artifacts and as such you could deploy the application and configure it. Below I’ve listed the items which need to be configured</p>
<p>&nbsp;</p>
<h4>Receive Port and Location</h4>
<p>In order to start processing the inbound Flat File we need to set up a receive port and receive location. Once this has been configured using the File Adapter we can simply start the processing of a readings flat file by dropping such a file in the folder to which the file adapter listens. The initial processing includes debatching the inbound flat file structure to separate files using the earlier defined <strong>[ Receive Pipeline ]</strong> and the <strong>[ Transformation ]</strong> of the xml presentation of the flat file energy reading to the internal reading format.</p>
<p>&nbsp;</p>
<p>Below the settings I used for configuring the receive port and location</p>
<p>&nbsp;</p>
<p>Receive Port Type <strong>[ One Way ]</strong></p>
<p>Receive Port Inbound Map <strong>[ EnergyReadingFF_TO_Reading]</strong></p>
<p><font color="#333333">Receive Location Transport Type <strong>[ FILE Adapter ]</strong></font></p>
<p><font color="#333333">Receive Location Receive Pipeline <strong>[Flat File Pipeline created earlier]</strong></font><br />Inbound Map:&nbsp;&nbsp;&nbsp; EnergyReadingFF_TO_Reading</p>
<p>&nbsp;</p>
<p>Send port:<br />Transport Type: File<br />Filters: BTS.Operation = name of the send port operation name in the orchestration</p>
<p>&nbsp;</p>
<h4>Send Port</h4>
<p>The send port which needs to be configured will subscribe to messages which are send out by the orchestration and ensures that this message is written to disk. </p>
<p>&nbsp;</p>
<p>Below the settings I used for configuring the receive port and location</p>
<p>Send Port Trabs port Type <strong>[ FILE Adapter ]</strong></p>
<p>Send Port Filter <strong>[BTS.Operation = name of the send port operation name in the orchestration]</strong></p>
<p><font color="#333333"></font>&nbsp;</p>
<p><font color="#333333"></font>&nbsp;</p>
<h2>Et voila</h2>
<p>So I hope you enjoyed this post, and feel free to give me a shout on twitter <a href="https://www.twitter.com/ReneBrauwers" target="_blank" rel="noopener noreferrer">@ReneBrauwers</a> or in the comments below, and as a reminder you can download the source <a href="http://bit.ly/1AN6arB" target="_blank" rel="noopener noreferrer">here</a> (including bindings)</p>
<p>&nbsp;</p>
<p>Please note; the bindings might not import this is most likely due to the fact that I use different Host Instance names (Processing_Host for the orchestration, Receive_Host and Send_Host for receiving and sending the files)</p>
<p>&nbsp;</p>
<p>&nbsp;</p>
<p>Cheerio</p>
<p>&nbsp;</p>
<p>René</p>
