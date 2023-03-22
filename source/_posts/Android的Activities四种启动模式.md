---
title: Android的Activities四种启动模式
tags: ["Android"]
categories: Android
date: 2018-07-25
grammar_cjkRuby: true
copyright: ture
---

Android的Activities四种启动模式：`Standerd`、`SingleTop`、`SingleTask`、`SingleInstance`。

<!-- more -->

### Standard(默认标准启动模式)

每次启动都重新创建一个新的实例，不管它是否存在。且谁启动了这个Acitivity,那么这个Acitivity就运行在启动它的那个Acitivity的任务栈中。

### SingleTop(栈顶复用模式)

如果新的Activity已经位于任务栈的栈顶，那么不会被重新创建，而是回调onNewIntent()方法，通过此方法的参数可以取出当前请求的信息。

### SingleTask(栈内复用模式)

这是一种单例模式，在这种模式下，只要Acitivity在一个栈中存在，那么多次启动此Acitivity都不会重建实例，而是回调onNewIntent方法。同时由于SingleTask模式有ClearTop功能，因此会导致所要求的Acitivity上方的Acitivity全部销毁。

### SingleInstance(单实例模式)

和栈内复用类似，此种模式的Acitivity只能单独位于一个任务栈中。全局唯一性。单例实例，不是创建，而是重用。独占性，一个Acitivity单独运行在一个工作栈中。