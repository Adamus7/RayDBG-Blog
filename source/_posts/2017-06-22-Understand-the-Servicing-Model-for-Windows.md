---
title: Understand the Servicing Model for Windows
date: 2017-06-22 22:28:07
tags:
---

From Oct 2016,  the family of Windows 7, Windows 8.1 and Windows 2012/R2 introduced a new patching releasing model called *Rollup Model*. Starting with February 2017, the Cumulative Security Update for Internet Explorer will not be included by Security Only updates. Therefor, normally, you should see 4 different types of patch in one month:
* Security Monthly Quality Update (aka the *Monthly Rollup*)
* Security Only Quality Update (aka the *Security Only updates*)
* Preview of Monthly Quality Rollup (aka the *Preview Rollup*)
* Cumulative Security Update for Internet Explorer (aka the *CU for IE*)

Should we install all of them?
Before answer this question, let us see the relationship of above patches.

| Type                  | Contents                                                                             |
|-----------------------|--------------------------------------------------------------------------------------|
| Monthly Rollup        | IE CU + security and non-security fixes in this month + all fixes in previous months |
| Security Only updates | security fix in this month                                                           |
| Preview Rollup        | non-security fixes in this month +  all fixes in previous months                     |
| CU for IE             | Fixes for IE                                                                         |

{% asset_img image1.png %}