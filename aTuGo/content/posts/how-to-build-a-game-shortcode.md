---
title: "How to Build a Game Shortcode"
date: 2021-03-23T10:21:30+08:00
draft: false
comment: true
---



```html
<a href="" class="gamecover" style="float: left;
    ">
    <img src="https://images.igdb.com/igdb/image/upload/t_cover_big/{{ .Get 0 }}.jpg" style="border-radius: 5px; 
    padding: 10px 0 10px 10px; 
    width: 140px;">
</a>
<div class="info" style="float:left">
    <p class="palydate">游戏日期：{{ .Get 1}}</p>
    <p class="rate">评级：
        {{ if eq (.Get 2) 1}}🌝🌑🌑🌑🌑
            {{ else if eq (.Get 2) 2}}🌝🌝🌑🌑🌑
            {{ else if eq (.Get 2) 3}}🌝🌝🌝🌑🌑
            {{ else if eq (.Get 2) 4}}🌝🌝🌝🌝🌑
            {{ else if eq (.Get 2) 5}}🌝🌝🌝🌝🌝
        {{ end }}
    </p>
    <p class="describe">{{ .Get 3}}</p>
</div>
```