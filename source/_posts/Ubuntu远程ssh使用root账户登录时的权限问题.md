---
title: Ubuntu远程ssh使用root账户登录时的权限问题
tags: ["Linux"]
categories: Linux
date: 2019-05-05
grammar_cjkRuby: true
copyright: ture
---

Ubuntu使用ssh进行远程登陆，账户为 `root` 时，提示出现 Permission denied 错误。

 <!-- more -->

解决办法，编辑服务器中的 `/etc/ssh/sshd_config` 配置文件：

搜索找到 `PermitRootLogin without-password` 这行，注释掉，然后添加代码 `PermitRootLogin yes`

然后重启 ssh

```sh
sudo service ssh restart
```

另外，用 `ssh密钥 `也可以，使用 `ssh-keygen` 生成后，追加在 `/root/.ssh/authorized_keys` 文件中即可.

