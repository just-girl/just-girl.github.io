---
title: Ubuntu系统出现无法连接adb问题的解决方案
tags: ["Linux"]
categories: Linux
date: 2019-04-12
grammar_cjkRuby: true
copyright: ture
---

如果以下情况：

```shell
$ adb devices
List of devices attached
xxxxxxxx    no permissions (user in plugdev group; are your udev rules wrong?);
see [http://developer.android.com/tools/device.html]
```

<!-- more -->

检查设备的ID和产品ID：

```shell
$ lsusb
Bus 001 Device 002: ID 8087:8000 Intel Corp. 
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 003 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 002 Device 078: ID 138a:0011 Validity Sensors, Inc. VFS5011 Fingerprint Reader
Bus 002 Device 003: ID 8087:07dc Intel Corp. 
Bus 002 Device 002: ID 5986:0652 Acer, Inc 
Bus 002 Device 081: ID 22b8:2e81 Motorola PCS  #示例设备
Bus 002 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

在这里，示例设备为 Motorola PCS ，所以我的 `vid=22b8`  和 `pid=2e81`

在路径  `/etc/udev/rules.d` 下，新建 `51-android.rules`

```
$ sudo vi /etc/udev/rules.d/51-android.rules
SUBSYSTEM=="usb", ATTR{idVendor}=="22b8", ATTR{idProduct}=="2e81", MODE="0666", GROUP="plugdev"
```

现在重启 `udev` 规则

```shell
sudo udevadm control --reload-rules
```

现在，设备可以被识别了：

```shell
$ adb devices
List of devices attached
ZF6222Q9D9  device
```

> 如果依然无法识别：
> 1、重新拔插手机设备；
> 2、重启系统。