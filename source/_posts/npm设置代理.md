---
title: npm设置代理
tags: ["npm"]
categories: npm
date: 2019-03-14
grammar_cjkRuby: true
copyright: ture
---

## 背景

 通常情况下当 npm 无法下载依赖时，我们使用 cnpm 就够了。但是总是存在特殊情况的：

1. 公司内网，无法访问 npm，但是有一个代理可以访问外网

2. 网络质量很差，连 cnpm 都无法正常使用（比如长城坑带，跨市就出现大量的丢包）

那就需要给 npm 设置代理了

<!-- more -->

## 设置代理

### http 代理

npm 原生支持 http 代理，直接设置即可

```shell
# 假设本地代理端口为1080
npm config set proxy "http://localhost:1080"
npm config set https-proxy "http://localhost:1080"

# 有用户密码的代理
npm config set proxy "http://username:password@localhost:1080"
npm confit set https-proxy "http://username:password@localhost:1080"
```

### socks5 代理

npm 不支持 socks 代理，但是我们可以用一个工具将 http 代理转成 socks 代理，然后将 npm 代理地址设置到这个工具的地址。

```shell
# 假设本地socks5代理端口为1081
# 首先安装转换工具
npm install -g http-proxy-to-socks
# 然后使用这个工具监听1080端口,支持http代理，然后所有1080的http代理数据都将转换成socks的代理数据发送到1081上
hpts -s localhost:1081 -p 1080
# 最后设置npm代理为1080
npm config set proxy "http://localhost:1080"
npm config set https-proxy "http://localhost:1080"
```

相当于又加了一个中间层，将 http 转成 socks。

## 删除代理

```shell
npm config delete proxy
npm config delete https-proxy
```

