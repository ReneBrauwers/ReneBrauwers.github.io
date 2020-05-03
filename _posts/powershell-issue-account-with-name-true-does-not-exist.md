---
ID: 4872
post_title: 'PowerShell issue: Account with name ‘True’ does not exist'
post_name: >
  powershell-issue-account-with-name-true-does-not-exist
author: Rene Brauwers
post_date: 2016-07-30 11:14:06
layout: post
link: >
  https://brauwers.azurewebsites.net/2016/07/30/powershell-issue-account-with-name-true-does-not-exist/
published: true
tags: [ ]
categories:
  - Azure
  - Powershell
---
Whilst I was scripting out some azure deployment stuff, I suddenly was getting this error whenever I tried to invoke

Get-AzureSqlDatabaseServer or Get-AzureSqlDatabaseServerFirewallRule

<b><i>Account with name 'True' does not exist</i></b>

So I ended up looking at my different subscriptions and I noticed the following (see screenshot)

<a href="https://menetazure.azurewebsites.net/wp-content/uploads/2016/08/powershell_screenshot1.jpg"><img class="alignnone size-medium wp-image-4873" src="https://menetazure.azurewebsites.net/wp-content/uploads/2016/08/powershell_screenshot1-300x264.jpg" alt="powershell_screenshot1" width="300" height="264" /></a>

As you can see, 2 subscriptions contain the value True for DefaultAccount, so obviously this was causing the issue (as the account I was querying against was my Default one)

Once I noticed the above, I only had to get my account information for my subscriptions and update my subscriptions which had the DefaultAccount set to true.

So I performed the following steps to resolve the issue

1. In powershell execute: Get-AzureAccount and match the correct ID to Subscription (lookup the value in subscriptions (1) and compare them to the subscriptions id obtained using Get-Subscriptions). Once found, copy the corresponding Id (2)

<img class="alignnone size-medium wp-image-4874" src="https://menetazure.azurewebsites.net/wp-content/uploads/2016/08/powersgell_screenshot2-300x78.png" alt="powersgell_screenshot2" width="300" height="78" />

2. Now execute the following

Select-AzureSubscription -SubscriptionName "YOUR SUBSCRIPTION NAME" -Account "VALUE OBTAINED IN PREVIOUS STEP"

Et voila; it works again

Cheers