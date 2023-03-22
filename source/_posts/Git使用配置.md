---
title: Git使用笔记
tags: ["Git"]
categories: Git
date: 2018-03-17
grammar_cjkRuby: true
copyright: ture
---

### 设置配置

``` shell
$ git config --global user.name "Your Name"
$ git config --global user.email "email@example.com"
```
 <!-- more -->
**注意git config命令的--global参数，用了这个参数，表示你这台机器上所有的Git仓库都会使用这个配置，当然也可以对某个仓库指定不同的用户名和Email地址**

### 创建SSH Key

``` shell
$ ssh-keygen -t ed25519 -C "youremail@example.com"
```
你需要把邮件地址换成你自己的邮件地址，然后一路回车，使用默认值即可，由于这个Key也不是用于军事目的，所以也无需设置密码。
如果一切顺利的话，可以在用户主目录里找到.ssh目录，里面有id_rsa和id_rsa.pub两个文件，这两个就是SSH Key的秘钥对，id_rsa是私钥，不能泄露出去，id_rsa.pub是公钥，可以放心地告诉任何人。

###  测试是否可以正常连接

```shell
ssh -T git@git.coding.net  #Coding
```

```shell
ssh -T git@github.com  #Github
```
### 设置Git代理

```shell
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```
### 取消Git代理

```shell
git config --global --unset http.proxy
git config --global --unset https.proxy
```

### 忽略文件名大小写

```shell
git config core.ignorecase true
```

