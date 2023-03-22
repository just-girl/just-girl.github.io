---
title: 正确为Android Studio配置ssr代理
tags: ["Android"]
categories: Android
date: 2019-09-12
grammar_cjkRuby: true
copyright: ture
---

打开Android Studio，进入Preferences，搜索proxy关键字，进入proxy设置页面。

 <!-- more -->
打开本地代理软件SS，等其他socks代理软件

![](/images/4352353.png)

但是这样只能下载sdk，这样配置之后有没有发现我们使用Gradle下载某些依赖的时候还是下载不下来，因为gradle不支持socks代理，这里我们可以将socks转为http代理，或者给jvm设置代理

在AS中我们可以直接给JVM设置代理

```groovy
org.gradle.jvmargs=-DsocksProxyHost=127.0.0.1 -DsocksProxyPort=1080
```

这里的端口要根据你自己代理的端口变化，一般都是1080

在我们的跟目录的 gradle.properties 加上上面配置就可以下载了。