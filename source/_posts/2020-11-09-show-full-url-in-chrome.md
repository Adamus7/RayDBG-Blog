---
title: Chrome 地址栏开启显示完整 URL 模式
date: 2020-11-09 10:59:20
tags:
- Chrome
- Trick
---
升级到 Chrome 76 之后，地址栏中将不在显示完整的 URL。Google 从安全性角度的考虑启用了这项功能，但是对于个人的日常工作来说，可能带来了更多的不便，比如经常需要根据瞄一眼 URL 中的 Query String 进行页面情况的确认。
为了关闭这个特性做了一番搜索，记录一下。
<!--more-->
1. 访问 chrome://flags/
2. 搜索 `#omnibox-ui-hide-steady-state-url-path-query-and-ref-on-interaction`
3. 禁用 "Omnibox UI Hide Steady-State URL Path, Query, and Ref On Interactio" 选项

{% asset_img disable-flags.jpg %}

效果：
{% asset_img before.jpg %}
{% asset_img after.jpg %}
