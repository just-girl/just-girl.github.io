---
title: 网页版HTML5视频倍速播放方法
tags: ["HTML"]
categories: HTML
date: 2018-11-17
grammar_cjkRuby: true
copyright: ture
---

仅支持的浏览器有Chrome 43+、 Firefox 20+、 IE 9+、 Edge 12+等版本。

 <!-- more -->

```shell
document.querySelector('video').defaultPlaybackRate = 2.0;
document.querySelector('video').play();
document.querySelector('video').playbackRate = 16.0;
```


