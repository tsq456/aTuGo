---
title: "使用Google Spreadsheet统计PV和UV"
date: 2021-03-30T21:44:20+08:00
draft: false
comment: true
description: "内心OS：没什么人看你搞这功能干嘛？..."
featured_image: 
tags: [hugo,折腾]
toc: true
---
>21年4月2日：我是沙雕，原来有现成的服务可以使用，[不蒜子](https://busuanzi.ibruce.info/)就提供很好的网站计数服务...而且谷歌表格由于统计周期的问题，只适合用来统计PV统计。

>本文基于hugo已接入Google Analytics（下文统称GA）以及能够访问Google Spreadsheet为前提。大部分HUGO主题都已经默认集成了GA，至于怎么访问谷歌表格，你懂得。

## 源起
经常看别人博客，总是能看到访问数，有的还能区分出PV和UV，就想着是不是也能在自己这也实现，就去搜了下解决方案，在HUGO社区的[这篇帖子](https://discourse.gohugo.io/t/total-number-of-views/9729/4)中提到了实现这个结果的几种方法：

+ 接入服务器日志（因为我是架设在VERCEL上的，所以PASS）
+ 接入第三方服务（提到了[reliablecounter](https://www.reliablecounter.com/)，一键就能生成统计代码接入站点，当时没细看，所以也PASS了）
+ 通过GA统计，但是GA屏蔽了外部的链接可能，并且准确性越来越差（貌似不大靠谱）

在这三种办法里，在我看来，方法一可能需要自己定义UV和PV的标准，然后需要通过公式计算出所要的结果，对于我来说太复杂了；方法二应该说是标准性的问题，并没有提到统计的口径是什么。这样看来就第三种方法最适合自己了。

在帖子的最后，有个用户提到了一种讨巧的读取GA数据的办法：

> You can set up Google Analytics to store data in a Google Calc document (stored in your Google Drive account), and that spreadsheet document can be made available online for people who know the special link. Then with Hugo’s `getCSV` function you can download and read that file.

**意思就是通过谷歌表格的GA插件创建表格报表，然后再通过HUGO的`getCSV`方法调用表格里特定单元格的数据，以此方法来达成统计访问用户数量的目的。**话不多说，开搞。

## 谷歌表格设置

### 安装插件

> 完整安装过程，[这篇文章](https://developers.google.com/analytics/solutions/google-analytics-spreadsheet-add-on)里有进行介绍。

1. 创建一个新表格或使用一个现存表格；
2. 打开菜单上的【插件】 - 【获取插件】；
3. 搜索Google Analytics并将其添加到自己的插件中；
4. 接下来就可以在菜单的【插件】下看到Google Analytics了。

![image-20210331123858145](https://30924398.xyz:6001/images/2021/03/31/google-spreadsheet-addon.png#center)

### 建立报表并获取数据

> 如果你网站已经接入GA，并且在GA中也已经进行了相关配置，这个插件中会进行自动关联的，毕竟都是自家的产品。

接下来就是通过插件来定义你要获取的数据字段，因为只是要统计PV和UV，量度（Metrics）分别取`Users`和`Session`，而维度（Dimensions）都取日期`Date`（更多的数据我也看不懂了LOL，可以在[GA的文档](https://ga-dev-tools.appspot.com/dimensions-metrics-explorer/)里进行查看）。需要注意的是一个报告只能对应一个量度和维度，所以统计PV和UV需要创建两个报表。

![image-20210331125049198](https://30924398.xyz:6001/images/2021/03/31/ga-sets.png#center)

创建完报表后，插件会自动在表格里生成一列字段及对应的之前在右侧边栏里配置的量度和维度。（无视我把UV达成了CV...）

![image-20210331130104900](https://30924398.xyz:6001/images/2021/03/31/20210331130104.png#center)



需要注意的事，**在创建完报表后需要在插件中点击`Run reports`才会真正生成报表**，这样就会在谷歌表格里生成两个基于你自定义的报表名称的两个子表格。

![image-20210331125736724](https://30924398.xyz:6001/images/2021/03/31/20210331125748.png#center)

最后因为要让这个报表定期运行，你需要在插件的`Schedule Reports`中设置好自动运行报表的间隔时间，最短间隔时间是小时。

![image-20210401123939960](https://30924398.xyz:6001/images/2021/04/01/20210401123940.png#center)

至此，所有谷歌表格中的插件配置完成。

### 转换谷歌表格链接

谷歌表格默认格式不是CSV，因此需要把表格链接转换为CSV的分享格式。转换的方式在[这篇文章](https://www.highviewapps.com/blog/how-to-create-a-csv-or-excel-direct-download-link-in-google-sheets/)中有进行介绍。

谷歌链接的标准格式是这样：

``` plaintext
https://docs.google.com/spreadsheets/d/1Gztm9o8JEPibPWEDwH54qBG5kwj51ILDOk_dxK6uTSY/edit#gid=1330027783
```

其中 `1Gztm9o8JEPibPWEDwH54qBG5kwj51ILDOk_dxK6uTSY`是你的表格ID，gid=`1330027783`是子表的ID，因此需要获取具体一个子表的CSV链接就是下面这样（不要子表就把&gid以及之后的字符去掉）：

``` plaintext
https://docs.google.com/spreadsheets/d/1Gztm9o8JEPibPWEDwH54qBG5kwj51ILDOk_dxK6uTSY/export?format=tsv&gid=1330027783
```

**最后的最后，记得把谷歌表格的分享形式设置为共享，否则就会提示下面这段错误信息（不要问我怎么知道的）。**

```plaintext
ERROR 2021/03/30 21:05:28 Failed to get CSV resource “https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv”: failed to parse CSV file https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv: parse error on line 3, column 11: bare " in non-quoted-field
If you feel that this should not be logged as an ERROR, you can ignore it by adding this to your site config:
ignoreErrors = [“error-remote-getcsv”]
ERROR 2021/03/30 21:05:28 render of “page” failed: execute of template failed: template: _default/single.html:9:6: executing “main” at <index (index $csv 1) 1>: error calling index: index of untyped nil
ERROR 2021/03/30 21:05:28 render of “page” failed: execute of template failed: template: _default/single.html:9:6: executing “main” at <index (index $csv 1) 1>: error calling index: index of untyped nil
ERROR 2021/03/30 21:05:28 render of “page” failed: execute of template failed: template: _default/single.html:9:6: executing “main” at <index (index $csv 1) 1>: error calling index: index of untyped nil
ERROR 2021/03/30 21:05:28 render of “page” failed: execute of template failed: template: _default/single.html:9:6: executing “main” at <index (index $csv 1) 1>: error calling index: index of untyped nil
Built in 32175 ms
Error: Error building site: failed to render pages: render of “page” failed: execute of template failed: template: _default/single.html:9:6: executing “main” at <index (index $csv 1) 1>: error calling index: index of untyped nil
```



## HUGO getCSV

[HUGO官方文档](https://gohugo.io/templates/data-templates/#example-for-csv-files)对于`getCSV`方法的使用示例是这样的。

```html
 <table>
    <thead>
      <tr>
      <th>Name</th>
      <th>Position</th>
      <th>Salary</th>
      </tr>
    </thead>
    <tbody>
    {{ $url := "https://example.com/finance/employee-salaries.csv" }}
    {{ $sep := "," }}
    {{ range $i, $r := getCSV $sep $url }}
      <tr>
        <td>{{ index $r 0 }}</td>
        <td>{{ index $r 1 }}</td>
        <td>{{ index $r 2 }}</td>
      </tr>
    {{ end }}
    </tbody>
  </table>
```

*这里没搞太清楚的是，为什么`$i`这个变量没有声明，也能用在这段代码中...*

所以我当时就依葫芦画瓢也用了`range`套在外面，像下面这样，结果死活不输出内容。

```html
{{ $url := "https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv&gid=1124904396" }}
{{ $csv := getCSV "," $url }}
{{ $r := 12 }}
{{ $c := 2 }}
<i>
  {{ range $i,$csv }}
    {{ index ( index $csv $r ) $c }}   
  {{ end }}
</i>
```

后来到[官方社区](https://discourse.gohugo.io/t/getcsv-output-nothing/32052/2)一问，有个大佬回复说了下面这些内容，才发现原来不用加`range`，大概意思就是如果加了range就指向了具体某一行内容了，如果想要指向任一单元格，就要把`range`去掉。另外`getCSV`是把csv转换成数组，因此是从0开始计算单元格的。

![image-20210401125535223](https://30924398.xyz:6001/images/2021/04/01/image-20210401125535223.png#center)

所以把代码换成下面这样就OK了。

```html
{{ $url := "https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv&gid=1124904396"}}
{{ $csv := getCSV "," $url }}

<p>{{ index ( index $csv 11 ) 1 }}</p>
```

`{{ index ( index $csv 11 ) 1 }}`表示指向表格的第12行，第2列内容，即表格中的B12单元格。

## 总结

因此完成的步骤应该如下：

1. 谷歌表格中创建GA报表并开启自动运行，其中Users量度表示新用户数，Sessions量度表示访问总数；
2. 将谷歌表格链接转化为CSV格式；
3. 在HUGO中使用`getCSV`方法引入CSV链接并读取表格下的具体单元格。

最后代码如下。

```html
<p>
{{ $uvURL := "https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv&gid=1124904396" }}
{{ $pvURL := "https://docs.google.com/spreadsheets/d/1N8480w1hPyilUS4qK1rnSY6RiE-OX-2hy1glQCTRh_w/export?format=csv&gid=1100827646" }}
{{ $uvCSV := getCSV "," $uvURL }}
{{ $pvCSV := getCSV "," $pvURL }}

本站访问量：{{ index ( index $pvCSV 11 ) 1 }}&nbsp;&nbsp;您是第{{ index ( index $uvCSV 11 ) 1 }}位访问者
</p>
```


