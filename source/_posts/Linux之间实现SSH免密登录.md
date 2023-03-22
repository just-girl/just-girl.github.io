---
title: Linux之间实现SSH免密登录
tags: ["Linux"]
categories: Linux
date: 2021-08-07
grammar_cjkRuby: true
copyright: ture
---

为了让两个 Linux 机器之间使用 ssh 不需要用户名和密码。

<!-- more -->

## 本地生成公钥私钥对
```shell
# 注：ED25519 算法的安全性和性能优于 RSA，DSA已经被淘汰。
# 通过 ED25519 算法生成
ssh-keygen -t ed25519 -C "<comment>"
# 通过 RSA 算法生成
ssh-keygen -t rsa -b 4096 -C "<comment>"
```

## 复制本机的公钥到远程服务器中

`ssh-copy-id` 命令可以把本地的 ssh 公钥文件安装到远程主机对应的账户下，会将本地主机的公钥追加到远程主机的 `authorized_keys` 文件中，也会给远程主机的用户主目录（home）和 ~/.ssh，和 ~/.ssh/authorized_keys 设置合适的权限。

使用方法：

```shell
ssh-copy-id [-f] [-n] [-i identity file] [-p port] [-o ssh_option] [user@]hostname
```

> **-f** Don't check if the key is already configured as an authorized key on the server. Just add it. This can result in multiple copies of the key in authorized_keys files.
>
> **-i** Specifies the identity file that is to be copied (default is ~/.ssh/id_rsa). If this option is not provided, this adds all keys listed by ssh-add -L. Note: it can be multiple keys and adding extra authorized keys can easily happen accidentally! If ssh-add -L returns no keys, then the most recently modified key matching ~/.ssh/id*.pub, excluding those matching ~/.ssh/*-cert.pub, will be used.
>
> **-n** Just print the key(s) that would be installed, without actually installing them.
>
> **-o** ssh_option Pass -o ssh_option to the SSH client when making the connection. This can be used for overriding configuration settings for the client. See ssh command line options and the possible configuration options in ssh_config.
>
> **-p** port Connect to the specifed SSH port on the server, instead of the default port 22.
>
> **-h** or **-?** Print usage summary.

## 实现免密登录

```shell
ssh-copy-id [user@]hostname
ssh-copy-id -i ~/.ssh/id_ed25519.pub [user@]hostname
```

## 参考链接

[ssh-copy-id for copying SSH keys to servers](https://www.ssh.com/academy/ssh/copy-id#setting-up-public-key-authentication)

