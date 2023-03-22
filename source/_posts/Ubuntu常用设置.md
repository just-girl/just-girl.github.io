---
title: Ubuntu常用设置
tags: ["Linux"]
categories: Linux
date: 2018-06-16
grammar_cjkRuby: true
copyright: ture
---

Ubuntu 18.10 安装之后需要做的 事，下面会列出系统安装之后需要做的一些事，这将使得 Ubuntu 的体验更加出色。

<!-- more -->

### 安装更新
```shell
sudo apt update
sudo apt upgrade
```

### 启用「点击Ubuntu Dock最小化」

在 Ubuntu 18.10 中点击应用程序「最小化」按钮可以隐藏到 Ubuntu Dock 栏图标，再点击 Ubuntu Dock 栏中的图标时可以还原界面。但在默认情况下，点击 Ubuntu Dock 图标却无法将应用程序界面最小化。

```shell
gsettings set org.gnome.shell.extensions.dash-to-dock click-action 'minimize'
```

### 显示电池百分比

```shell
gsettings set org.gnome.desktop.interface show-battery-percentage true
```

### 用TLP降低发热

TLP 是一个有助于「系统冷却」的应用程序，可以让 Ubuntu 运行得更快、更顺畅。安装完成后，运行命令启动它即可，而无需任何配置。

```shell
sudo add-apt-repository ppa:linrunner/tlp
sudo apt update
sudo apt install tlp tlp-rdw
sudo tlp start
```

### 设置软件更新镜像

Ubuntu 的软件源配置文件是 /etc/apt/sources.list，确保 Ubuntu 从最佳服务器获取更新，建议使用[清华大学开源镜像站的软件源镜像](https://mirrors.tuna.tsinghua.edu.cn/help/ubuntu/)

Ubuntu 18.10

```shell
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-updates main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-updates main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-backports main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-backports main restricted universe multiverse
deb https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-security main restricted universe multiverse
# deb-src https://mirrors.tuna.tsinghua.edu.cn/ubuntu/ cosmic-security main restricted universe multiverse
```

### 使用apt-fast取代apt-get

如果您希望下载速度更快，可以安装apt-fast并在使用apt-get命令的地方用apt-fast来替换。

```shell
sudo add-apt-repository ppa:apt-fast/stable
sudo apt-get update
sudo apt-get install apt-fast
```

### 清理Ubuntu

```shell
sudo apt clean #只删除过时的软件包
sudo apt autoremove #清理整个缓存
```

### 安装Nvidia显卡驱动

```shell
sudo add-apt-repository ppa:graphics-drivers/ppa
sudo apt-get update
sudo apt-fast install nvidia-390 nvidia-settings nvidia-prime
```

安装完成后需要重启。

### 安装Preload

Preload（预加载）会在后台工作，以「研究」您如何使用计算机并增强计算机的应用程序处理能力。安装好 Preload 后，您使用频率最高的应用程序的加载速度就会明显快于不经常使用的应用程序。

```shell
sudo apt install preload
```

### 清理缩略图缓存

```shell
du -sh ~/.cache/thumbnails
```

### 卸载不必要的应用程序

```shell
sudo apt remove 软件包名
```

### 获取WiFi无线密码

```shell
cd /etc/NetworkManager/system-connections  #此目录中存储了网络连接详细信息的配置文件，可以使用ls命令列出所有 WiFi 连接配置文件。
```

使用 cat 命令查看 Linux 中已保存的 WiFi 配置文件，在 WiFi-Security 段的 psk 位置可以查看到无线密码。