---
title: "如何在hugo中统计总访问人数"
date: 2021-03-30T21:44:20+08:00
draft: true
comment: true
description: "HUGO本身没有集成这个功能，那么我们就另辟蹊径的通过GOOGLE ANALYTIC的方式来实现这个目的。"
featured_image: 
tags: 
---

```html
{{ $url := "https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv&gid=1124904396"}}
{{ $csv := getCSV "," $url }}

<p>{{ index ( index $csv 11 ) 1 }}</p>
```
