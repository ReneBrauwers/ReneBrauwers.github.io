---
ID: 5417
post_title: 'Tips &#038; Tricks: Cost savings using Logic Apps'
post_name: >
  tips-tricks-cost-savings-using-logic-apps
author: Rene Brauwers
post_date: 2017-03-30 20:25:00
layout: post
link: >
  https://brauwers.azurewebsites.net/2017/03/30/tips-tricks-cost-savings-using-logic-apps/
published: true
tags:
  - Azure DB
  - Cost Savings
categories:
  - Azure
  - Logic Apps
  - Tips and Tricks
---
<h1>We all love Logic Apps</h1>
Logic apps literally allows us to solve an integration challenge within minutes. This however does not prevent us from making unintentional expensive decisions when it comes to determining the total costs of running our solution.

The decisions which we, integration specialists as well as integration architects have to deal with no longer merely focus on implementing patterns, ensuring re-usability, allowing for scalability, ensuring resilience and rigidity, loos coupling and so on. The biggest challenge we are facing nowadays in my humble opinion consist of delivering awesome integration solutions whilst keeping the total operational costs within reason without sacrificing on quality

This post will by means of an example showcase three different scenario's how one can implement a logic app which main functionality is based around; a daily data synchronisation process to SQL Server.

If you are looking for the short version of this article (aka the tip and the trick), scroll down :-)

Most of you might think, hey mate; not so fast. Are you trying to implement a data movement (or ETL like) process using Logic Apps, shouldn't you be using Azure Data Factory for these kinds of scenario's? Well to be honest, ideally you should, but the overall price point and the fact that it is not available as of yet in all Azure Regions, I haven't taken this into account in the sample showcases. Nevertheless at the end of this post I will include a sample approximate TCO costing for having this scenario running using Azure Data Factory.

Well let's get to it, and start off with explaining the different scenario's…
<h2>Scenario’s</h2>
Note; the below mentioned Json is an example of the request which will kick of the process (for brevity reasons, I only list the first two items)

Sample json request

{
"ProductCatalogue": [{
"Name": "Blue Goose Seed Potatoes, Irish Cobbler",
"Pack": "50 LB",
"SoldBy": "BAG",
"SubCategory": "Seed Potatoes",
"Category": "Garden Supplies"
}, {
"Name": "Blue Goose Seed Potatoes, Kennebec",
"Pack": "50 LB",
"SoldBy": "BAG",
"SubCategory": "Seed Potatoes",
"Category": "Garden Supplies"
}]
}
<h3>Scenario 1: Logic App (ProductCatalogue-SYNC-OOB) leveraging out of the box connectors and not leveraging stored procedures.</h3>
<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image001.png"><img class="alignnone" style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image001" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image001_thumb.png" alt="clip_image001" width="588" height="616" border="0" /></a>

Our first scenario is triggered by means of an HTTP Post Request (1). Once received we will perform a for each over all [ProductCatalogue] arrays elements contained within the received JSON (2).

The next steps include, determining if we need to do an update or an insert; so before we can determine this we need to check in our database to see if the current item we are looping over already exists. We do this by means of utilising the "SQL Server - Get Rows" action. (3) This action utilises the configuration (displayed below) which leverages an OData Filter Query and will take only the first record returned (in our scenario, the OData filter should already cater for this).

So why not use the 'Get Row' action, you might ask; well this one requires us to pass in the Row Id (PK) and well in our scenario this value is not being passed in by the request as such we cannot leverage this action.

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image0014.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image001[4]" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image0014_thumb.png" alt="clip_image001[4]" width="420" height="407" border="0" /></a>

Once we have execute the 'Get Rows' action, our next action will need to check if a record was returned (4). Based on this outcome we will either perform an Insert or an Update (5)
<h3>Scenario 2: Logic App (ProductCatalogue-SYNC-SPROD) leveraging out of the box connectors, leveraging standard upsert stored procedure to store data.</h3>
<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image0017.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image001[7]" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image0017_thumb.png" alt="clip_image001[7]" width="386" height="460" border="0" /></a>

Our second scenario is triggered by means of an HTTP Post Request (1). Once received we will perform a for each over all [ProductCatalogue] arrays elements contained within the received JSON (2).

The subsequent action consist of the Execute Stored Procedure Action and passes in the data to be inserted.

Note: The stored procedure which is being invoked contains the logic to either insert or update the data as such we don't need to perform the 'existence' check as we did in scenario 1.
<h3>Scenario 3: Logic App (ProductCatalogue-SYNC-SPROD) leveraging out of the box connectors, leveraging upsert stored procedure accepting JSON to perform bulk upserts into store data.</h3>
<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image002.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image002" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image002_thumb.png" alt="clip_image002" width="366" height="250" border="0" /></a>

The last scenario is triggered by means of an HTTP Post Request (1). Once received we will execute a stored procedure which takes the request Json message as a parameter as well as a parameter used to indicate the name of the array node (2)

Note: this scenario is pretty basic and all the logic which is required is handled within the stored procedure itself.
<h2>Performance</h2>
Now that we have the scenario's out of our way, let's dive into the performance metrics. Which I inferred after invoking the logic apps in question. It has to be noted I performed 3 tests for each Azure DB Tier/Scenario and took the lowest run-duration (exception to this is scenario 1, as it would have taken 'ages'). It also has to be noted that I looked at the run duration time.

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/image.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/image_thumb.png" alt="image" width="660" height="603" border="0" /></a>
<h3>Conclusion</h3>
So looking at the raw data, we want to focus on those scenario's which; take a long time to process and which contain a lot of action executions. Reason being is the scenario's in question are pristine candidates to optimise both from a performance point of few as well as from a costing point of view. More on the costing later…

Having said the above, the conclusion (per scenario) we can draw is:
<h4>Scenario 1: ProductCatalogue-SYNC-OOB</h4>
is taking a lot of time to finish and the total number of monthly and yearly action executions are high. (see below). Looking at SQL performance, there is no need to scale beyond S2 as the duration between S2 and P1 is minimal. Reason being; that looking at performance statistics on the dashboard we notice that the max DTU used at all times is just above 30 DTU, as such scaling up any larger will not drastically improve our performance).

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image00110.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image001[10]" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image00110_thumb.png" alt="clip_image001[10]" width="486" height="360" border="0" /></a>

<em>ProductCatalogue-SYNC-OOB</em><a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/image-1.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/image_thumb-1.png" alt="image" width="660" height="130" border="0" /></a>
<h4>Scenario 2: ProductCatalogue-SYNC-SPROC</h4>
is doing much better (performance, and number of action executions) than scenario 1, main reason being is that we are utilising a stored procedure and no longer are relying on 'existence check and manually determining if we need to do an insert or update; we are actually using 2998 action executions less per run.

Looking at SQL performance, there is no need to scale beyond the Basic Tier as the duration with more powerful SQL DB's is not worth the investment. Reason being; that looking at performance statistics on the dashboard we notice that the max DTU used at all times is below 5 DTU, as such scaling up any larger will not drastically improve our performance).

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image00112.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image001[12]" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image00112_thumb.png" alt="clip_image001[12]" width="481" height="350" border="0" /></a>

<em>ProductCatalogue-SYNC-SPROD</em><a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/image-2.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/image_thumb-2.png" alt="image" width="660" height="127" border="0" /></a>
<h4>Scenario 3: ProductCatalogue-SYNC-SPROC-JSON</h4>
is our top performer, which should not be that surprising considering we do not require to loop over all the product catalogue items as we are able to do a bulk insert by means of passing in the complete received Json.

Looking at SQL performance, there is no need to scale beyond the Basic Tier as the duration with more powerful SQL DB's is not worth the investment. Reason being; that looking at performance statistics on the dashboard we notice that the max DTU used at all times is below 5 DTU, as such scaling up any larger will not drastically improve our performance).

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image00114.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="clip_image001[14]" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/clip_image00114_thumb.png" alt="clip_image001[14]" width="475" height="355" border="0" /></a>

<em>ProductCatalogue-SYNC-SPROD-SJON</em><a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/image-3.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/image_thumb-3.png" alt="image" width="660" height="127" border="0" /></a>
<h2>Pricing</h2>
Looking at performance metrics is fun, but it is even more fun if we look at how much each scenario would cost us on a monthly and yearly basis.

Please note; all amounts mentioned are in Australian Dollars and they were calculated using the <a href="https://azure.microsoft.com/en-us/pricing/calculator/">Azure Pricing Calculator</a>

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/image-4.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/image_thumb-4.png" alt="image" width="660" height="515" border="0" /></a>

From a pure price point perspective, one would most definitely choose scenario 3, with the Basic Azure DB Tier: ProductCatalogue-SYNC-SPROC-JSON which would be saving us $1,698.96 when compared to Scenario 1: ProductCatalogue-SYNC-OOB and $566.40 when compared to Scenario 2:ProductCatalogue-SYNC-SPROC.

However if we would look at it from a Production point of view, one would not choose the Basic SQL DB tier, but at least the Standard S0 DB tier; even then the costs savings would be in the hundreds of dollars on a yearly basis.
<h2>Final Conclusion</h2>
In general when your solution is leveraging the awesomeness power of Logic Apps there are multiple valid way of solving the integration challenge at hand, however one should keep in mind the number of total actions you will be executing and refactor whenever possible (if it makes sense!, last thing you want to do is have a logic app which only calls azure functions; it will work and there might be valid scenario's to do it this way; but would it be the best option, that's the question you should ask yourself.

Anyways, just keep in mind missing out on an obvious refactor step within Logic Apps, could increase the number of action execution by a factor which goes in the hundreds, if not thousands. (see the scenario's above). So be smart, it could save you hundreds if not thousands of dollars! And we all, know our customer love quality especially if it comes at a reasonable (low) price ;-)
<h2>And here is the tip &amp; trick</h2>
So here, as promised the tip &amp; trick (well it's contained in the blogpost title, isn't it? ;-)

Whenever you have to deal with inserting/updating a batch of data in SQL Server using Azure Logic Apps, it would be best to leverage a stored procedure which takes the whole of the batch (either Json or XML). This way you will be able to reduce the number of actions contained within your logic-app, which subsequently will keep your total execution costs to a minimum and save you some bucks.

Note: At the time when I wrote this article, there was no build in SQL Connector for bulk inserts; this however is most likely to change over the course of the next months, as the MS Integration team is doing an amazing job and releasing new connectors and enhancing existing connectors all the time
<h2>Before we leave</h2>
As promised see below an estimated cost when leveraging Azure Data Factory for the above scenario. I am assuming it would require at least 2 Low Frequency Activities (Load the data from a storage account and executing a stored procedure which allows for bulk inserting using Json or XML).

<a href="https://brauwers-nl.azureedge.net/images/blog/2017/04/image-5.png"><img style="background-image: none; padding-top: 0px; padding-left: 0px; display: inline; padding-right: 0px; border: 0px;" title="image" src="https://brauwers-nl.azureedge.net/images/blog/2017/04/image_thumb-5.png" alt="image" width="660" height="81" border="0" /></a>

So; the logic-apps route would still be $522.01 less on a yearly basis.

Well I hoped you enjoyed the article, until next time, stay tuned!

René