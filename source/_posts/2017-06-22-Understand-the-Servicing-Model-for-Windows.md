---
title: Understand the Servicing Model for Windows
date: 2017-06-22 22:28:07
tags:
---
Recently, one of new IE patch bring a print issue to IE 11 on all version of Windows. Before getting the fix, the only solution is to uninstall the problematic KB. But many people are confused by one question: which KB should I uninstall To answer it, we need to figue out the patch releasing model of Windows.
<!-- more --> 
# Before Windows 10
From Oct 2016, a new patch releasing model, *Rollup Model*, was introduced to the family of Windows 7SP1/2008R2, Windows 8.1 and Windows 2012/R2 by Microsoft. Later, starting with February 2017, the Cumulative Security Update for Internet Explorer will not be included by Security Only updates. Meanwhile, there will be a standalone patch for IE 11. So now, normally, you should see 4 different types of patch in one month:
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