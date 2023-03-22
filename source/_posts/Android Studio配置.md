---
title: Android Studio配置
tags: ["Android"]
categories: Android
date: 2023-01-11T23:56:53+08:00
copyright: ture
draft: false
---

当我首次使用Android Studio的时候，会采用如下配置，下面的一些配置技巧或许对你有一定的帮助。

<!-- more -->

### 显示行号
1. File | Settings 打开设置
2. 选择 Editor | General | Appearance
3. 勾选 Show line numbers

### 命名前缀
1. File | Settings 打开设置
2. 选择 Editor | Code Style | Java
3. 选择 Code Generation 标签
4. 给普通 Field 添加一个 `m` 前缀，给 Static filed 添加一个 `s` 前缀

### 快速导包
1. File | Settings 打开设置
2. 选择 Editor | General | Auto Import
3. 勾选 Optimize imports on the fly
4. 勾选 Add unambiguous imports on the fly

### Logcat 颜色
1. File | Settings 打开设置
2. 选择 Editor | Color & Fonts | Android Logcat
3. 点击 Click on Save As…按钮创建一个新的配色 Scheme
4. 按照下面的表格修改对应的颜色(修改之前需要取消勾选 Use inherited attributes)

| Logcat级别 | 颜色    |
| ---------- | ------- |
| Assert     | #AA66CC |
| Debug      | #33B5E5 |
| Error      | #FF4444 |
| Info       | #99CC00 |
| Verbose    | #BBBBBB |
| Warning    | #FFBB33 |

### 取消代码注释中的<p>显示
1. File | Settings 打开设置
2. 选择 Editor | Code Style | Java | JavaDoc
3. 打开 Other 节点
4. 取消勾选 Generate "<p>" on empty lines

### 代码配色
1. File | Settings 打开设置
2. 选择 Editor | Color & Fonts | Java
3. 点击 Click on Save As…按钮创建一个新的配色 Scheme
4. 展开下方的 Variables 选择 Local variable
5. 设置右侧的 Foreground 颜色 `#68B5EE`

### 字体设置
1. File | Settings 打开设置
2. 选择 Editor | Font
3. 设置右侧的 Font 选择`Consolas`，Size选择`14`

### 文件编码设置
1. File | Settings 打开设置
2. 选择 Editor | File Encodings
3. 设置右侧的 Global Encoding、Project Encoding、Default encoding for properties files 为 `UTF-8`

### 显示快速文档
1. File | Settings 打开设置
2. 选择 Editor | General
3. 勾选 Show quick documentation on mouse move

### 设置启动项
1. File | Settings 打开设置
2. 选择 Appearance & Behavior | System Settings
3. 取消勾选 Reopen last project on startup 和 Confirm application exit
4. 选择 Open project in new window