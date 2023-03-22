---
title: Ubuntu UFW 防火墙设置
tags: ["Linux"]
categories: Linux
date: 2019-04-17
grammar_cjkRuby: true
copyright: ture
---

### 安装
Ubuntu 默认是有 `ufw` 的。如果因为种种原因没有的话：
``` shell
sudo apt-get install ufw
```
 <!-- more -->

### 初步配置

```shell
sudo nano /etc/default/ufw
```

#### 开启 ipv6 (optional)

```shell
IPV6=yes
```

#### 设置默认规则

```shell
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

### 设置 incoming 规则

### 允许 ssh

```shell
sudo ufw allow 22/tcp
```

#### 允许 http、https

```shell
sudo ufw allow 80
sudo ufw allow 443
```

#### 允许端口段

```shell
sudo ufw allow 1000:2000/tcp
```

#### 允许 ip

```shell
sudo ufw allow from 192.168.1.125
```

#### 设置允许 ip / subnet 访问特定端口

```shell
sudo ufw allow from 192.168.100.0/24 to any port 3306
```

#### 设置禁止规则

修改 `allow` 为 `deny` 即可。

### 开启 UFW

```shell
sudo ufw enable
```

### 查看 UFW 状态

```shell
sudo ufw status [verbose|numbered]
```

### 删除规则

```shell
sudo ufw delete allow 1000:2000/tcp
```

或

```shell
sudo ufw status numbered
sudo ufw delete [number]
```

`[number]` 为上一条返回的规则号码

### 其他

#### 关闭 UFW

```shell
sudo ufw disable
```

#### 重设 UFW

```shell
sudo ufw reset
```

#### 开启关闭 Logging

```shell
sudo ufw logging [on|off]
```