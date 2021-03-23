---
title: "创建游戏卡片Shortcode"
date: 2021-03-23T10:21:30+08:00
draft: false
comment: true
description: "利用Shortcode快速构建一张最近玩过的游戏的卡片"
---









### 前言

最近这前段业余时间主要就在做两件事，疯狂肝《戴森球计划》和建博客改主题，我想他们的共通之处就在于都能够通过自己一点一滴的努力构建出想要看到的结果，前者是通过传送带这样的微观元素逐步构建出戴森球这样宇宙尺度的巨构奇观，后者是一点点的代码构建出可视化的博客啦。本身就是相对重度的PC游戏玩家，刚好也有用到[豆瓣看过的电影](https://mufeng.me/post/have-seen-the-film)，就想着是不是能够通过hugo的短代码实现类似的卡片功能，当然人家是通过自带读取豆瓣API来实现的，我这不能比~话不多说，开搞

### HUGO SHORTCODE

> Shortcodes are simple snippets inside your content files calling built-in or custom templates.

官方对于短代码的定义是：在内容区域使用的内建或自定义创建的片段（ ~~祖传机翻，手动狗头~~）。上网简单的搜了下一些案例，例如怎么在内容区域快速插入B站视频。

```html
<iframe src="//player.bilibili.com/player.html?aid=587006475&bvid=BV15z4y117Eq&cid=309758932&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
```

以上是B站分享出来的支持内嵌到其他网站的代码如下，其中地址内的bvid就是每个视频特有的识别码，那么要做的就是将其变为shortcode里的param。

hugo shortcode的写法是`{{ shortcodename parameters }}`，其中shortcodename就是你的段代码名称，其实就是你在layouts/shortcode文件夹下建立的html文件名，paramaters是你传递给shortcode的参数，基于此，将B站的段代码替换为param就好了，因此在HUGO的layouts/shortcode目录下创建Bilibili.html文件输入以下代码。

```html
<iframe src="//player.bilibili.com/player.html?aid=587006475&bvid={{ .Get 0 }}&cid=309758932&page=1" scrolling="no" border="0" frameborder="no" framespacing="0" allowfullscreen="true"> </iframe>
```

其中`{{ . Get 0 }}`就是后续在段代码使用时传递的参数变量，接下来就是使用段代码的时候，直接在文章内输入{< bilibili BV11y4y187sR >}（golang标准语法是左右有俩大括号...我这里为了示意没法放）就会出现以下视频

{{< bilibili BV11y4y187sR >}}

小是小破站的锅，要改大还是可以对段代码进行更深入的定制的。这里就不说了。

### Gamecard

其实是想做一个专门的游戏页，来摆放自己玩过的游戏，因此需要有以下内容：

+ 游戏海报
+ 游戏名
+ 游戏日期
+ 游戏评分
+ 游戏体验描述

海报从[igdb](https://www.igdb.com/)搞，通过观察可以发现海报的地址只有jpg之前的一段字符是不一样的，因此将该部分替换成参数

```bash
https://images.igdb.com/igdb/image/upload/t_cover_big/co2jxw.jpg
https://images.igdb.com/igdb/image/upload/t_cover_big/co1zv7.jpg
```

游戏评分只是分值，未免有些单调，因此加入了基本的判断输入，如下：

```html
            <p>评级：{{ if eq (.Get 3) 1}}🌝🌑🌑🌑🌑{{ else if eq (.Get 3) 2}}🌝🌝🌑🌑🌑{{ else if eq (.Get 3) 3}}🌝🌝🌝🌑🌑{{ else if eq (.Get 3) 4}}🌝🌝🌝🌝🌑{{ else if eq (.Get 3) 5}}🌝🌝🌝🌝🌝{{ end }}</p>
```

其他的手动输入，只是省去了构建样式这一步，本质上只是个解耦的过程。最终短代码如下。

```html
<div class="gamecard">
    <div class="cover">
        <img src="https://images.igdb.com/igdb/image/upload/t_cover_big/{{ .Get 0 }}.jpg">
    </div>  
    <div class="info">
        <p class="name">{{ .Get 1 }}</p>
        <div class="meta">
            <p>游戏日期：{{ .Get 2}}</p>
            <p>评级：{{ if eq (.Get 3) 1}}🌝🌑🌑🌑🌑{{ else if eq (.Get 3) 2}}🌝🌝🌑🌑🌑{{ else if eq (.Get 3) 3}}🌝🌝🌝🌑🌑{{ else if eq (.Get 3) 4}}🌝🌝🌝🌝🌑{{ else if eq (.Get 3) 5}}🌝🌝🌝🌝🌝{{ end }}</p>
        </div>
        <p class="describe">{{ .Get 4}}</p>
    </div>
</div>
```

最后再加上CSS样式。无视我混乱冗余的css代码...

```css
.gamecard {
  max-width: 90%;
  background-color: rgba(128, 128, 128, 0.138);
  margin: 30px auto;
  border-radius: 5px;
  overflow: hidden;
  line-height: 0;
  padding: 10px;
}
.gamecard .cover {
  float: left;
}
.gamecard .cover img {
  border-radius: 5px;
  width: 120px;
}
.gamecard .info {
  margin-left: 140px;
  max-width: 80%;
}
.gamecard .info p {
  line-height: 20px;
  margin: 0;
  font-size: 14px;
}
.gamecard .info .meta {
  margin-top: 10px;
}
.gamecard .info .meta p {
  display: inline-block;
  margin-right: 10px;
}
.gamecard .info .name {
  font-size: 18px;
  font-weight: 700;
}
.gamecard .info .describe {
  margin-top: 20px;
  font-size: 16px;
}

```

最后用这样一段代码就可以构建出下面这样的游戏卡片啦。

```go
{< game co2h4q "戴森球计划" 2021-03-20 5 "这游戏也太尼玛肝了，浑身长满肝也不够玩啊。">}
```



{{< game co2h4q "戴森球计划" 2021-03-20 5 "这游戏也太尼玛肝了，浑身长满肝也不够玩啊。">}}

### 最后

其实[igdb](https://www.igdb.com/)是由开放API的，但是无奈公司被Twitch收购了，然后使用API要开放两步验证，但是操蛋的Twich账户不支持大陆手机的验证，就直接卡在这部没法继续了。不过也好，就自己现在这技术，给我API也啃不动。

另外记录几点做的过程中遇到的简单的坑和可以额外做到的事...

> + HUGO的条件判断是用`{{ if  eq param1 param2}}`这样的方式来输出布尔值
> + 如果param不相连要用()包起来
> + 以上的短代码里链接还没加上，igdb上的游戏详情页都是以https://www.igdb.com/games/为开头的，后续是加了连字符-的游戏名称，如戴森球计划就是https://www.igdb.com/games/dyson-sphere-program，那么在短代码里就可以通过正则表达式来将输入的游戏名自动加上连字符，来达到将游戏名转变为跳转链接的效果。可是重点是我不会正则...

以上，结束。