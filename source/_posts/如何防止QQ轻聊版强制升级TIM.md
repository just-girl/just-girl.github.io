---
title: 如何防止QQ轻聊版强制升级TIM
tags: ["Windows"]
categories: Windows
date: 2019-04-28
grammar_cjkRuby: true
copyright: ture
---

QQ主要使用以下2个升级程序来自动升级：

<!-- more -->

> txupd.exe
> QQProtectUpd.exe
> bugreport.exe

点击 Win 键，键盘输入 `gpedit.msc`, 回车，定位到 `用户配置` - `管理模版` - `系统` - `不要运行特定的Windows 应用` :

![插图1](/images/21342342.png)

双击 `不要运行特定的Windows 应用` ， 将其 `启用`，并且在下面的 `列表` 中添加上面提到的2个程序名称，然后保存即可。

![插图2](/images/3284653.png)

后续 QQ 将无法启动这两个程序，也就不会弹窗提醒，更不会给你悄悄的升级了。