---
title: "如何在hugo中统计总访问人数"
date: 2021-03-30T21:44:20+08:00
draft: false
comment: true
description: "HUGO本身没有集成这个功能，那么我们就另辟蹊径的通过GOOGLE ANALYTIC的方式来实现这个目的。"
featured_image: 
tags: [hugo,折腾]
toc: true
---

>本文基于hugo已经接入Google Analytics（下文统称GA）以及能够访问Google Spreadsheet为前提。大部分HUGO主题都已经默认集成了GA，至于怎么访问谷歌表格，这里就不细说啦。

### begin
经常看别人博客，总是能看到访问数，有的还能区分出PV和UV，就想着是不是也能在自己这也实现，就去搜了下解决方案，在HUGO社区的[这篇帖子](https://discourse.gohugo.io/t/total-number-of-views/9729/4)中提到了实现这个结果的几种方法：

+ 接入服务器日志（因为我是架设在VERCEL上的，所以PASS）
+ 接入第三方服务（提到了[reliablecounter](https://www.reliablecounter.com/)，一键就能生成统计代码接入站点，当时没细看，所以也PASS了）
+ 通过GA统计，但是GA屏蔽了外部的链接可能，并且准确性越来越差（貌似不大靠谱）

在这三种办法里，在我看来，方法一可能需要自己定义UV和PV的标准，然后需要通过公式计算出所要的结果，对于我来说太复杂了；方法二应该说是标准性的问题，并没有提到统计的口径是什么。这样看来就第三种方法最适合自己了。

在帖子的最后，有个用户提到了一种讨巧的读取GA数据的办法：

> You can set up Google Analytics to store data in a Google Calc document (stored in your Google Drive account), and that spreadsheet document can be made available online for people who know the special link. Then with Hugo’s `getCSV` function you can download and read that file.

**意思就是通过谷歌表格的GA插件创建表格报表，然后再通过HUGO的`getCSV`方法调用表格里特定单元格的数据，以此方法来达成统计访问用户数量的目的。**话不多说，开搞。

### google spreadsheet config

### setup addon

> 完整安装过程，[这篇文章](https://developers.google.com/analytics/solutions/google-analytics-spreadsheet-add-on)里有进行介绍。

1. 创建一个新表格或使用一个现存表格；
2. 打开菜单上的【插件】 - 【获取插件】；
3. 搜索Google Analytics并将其添加到自己的插件中；
4. 接下来就可以在菜单的【插件】下看到Google Analytics了。

![image-20210331123858145](https://30924398.xyz:6001/images/2021/03/31/google-spreadsheet-addon.png#center)

### get data

> 如果你网站已经接入GA，并且在GA中也已经进行了相关配置，这个插件中会进行自动关联的，毕竟都是自家的产品。

接下来就是通过插件来定义你要获取的数据字段，因为只是要统计PV和UV，量度（Metrics）分别取`Users`和`Session`，而维度（Dimensions）都取日期`Date`（更多的数据我也看不懂了LOL，可以在[GA的文档](https://ga-dev-tools.appspot.com/dimensions-metrics-explorer/)里进行查看）。需要注意的是一个报告只能对应一个量度和维度，所以统计PV和UV需要创建两个报表。

![image-20210331125049198](https://30924398.xyz:6001/images/2021/03/31/ga-sets.png#center)

创建完报表后，插件会自动在表格里生成带有一些字段以及对应的信息。

![image-20210331130104900](https://30924398.xyz:6001/images/2021/03/31/20210331130104.png#center)



需要注意的事，**在创建完报表后需要在插件中点击`Run reports`才会真正生成报表**，这样就会在谷歌表格里生成两个基于你自定义的报表名称的两个子表格。

![image-20210331125736724](https://30924398.xyz:6001/images/2021/03/31/20210331125748.png#center)

### transcode gs url

### HUGO config



### total







```html
{{ $url := "https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv&gid=1124904396"}}
{{ $csv := getCSV "," $url }}

<p>{{ index ( index $csv 11 ) 1 }}</p>
```
