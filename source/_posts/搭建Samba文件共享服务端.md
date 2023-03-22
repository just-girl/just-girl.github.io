---
title: 搭建Samba文件共享服务端
tags: ["Linux"]
categories: Linux
date: 2019-04-11
grammar_cjkRuby: true
copyright: ture
---

`Samba` 是著名的开源软件项目之一，它在 `Linux/UNIX` 系统中实现了微软的 `SMB/CIFS` 网络协议，从而使得跨平台的文件共享变得更加容易。在部署 Windows、Linux/UNIX 混合平台的企业环境时，选用 `Samba` 可以很好地解决不同系统之间的文件互访问题。

 <!-- more -->

# 开始准备

Windows电脑和服务器主机必须在相同的工作站域下，打开命令提示符检查Windows系统所在的工作站域

```powershell
net config workstation
```

![](/images/3456543.png)

在Windows系统中，修改 hosts 文件，以`管理员权限`打开命令提示符，并运行以下命令

```powershell
notepad C:\Windows\System32\drivers\etc\hosts
```

例如：

```shell
[...]
192.168.0.100 	server1.example.com	ubuntu
```

# 安装 `samba` 软件包

```shell
sudo apt install samba -y
```

备份samba的配置文件，方便出错的时候恢复初始设置

```shell
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

# 配置

## 编辑 `/etc/samba/smb.conf` 文件：

在 `[global]` 节点下添加以下键值对：

```shell
workgroup = WORKGROUP
security = user
```

## 在文件的最末尾添加新的节点：

```shell
[SambaShare]
    comment = Ubuntu File Server Share   #描述和介绍
    path = /srv/samba/share   #请添加需要的路径
    browsable = yes   #能够被 Windows Explorer 浏览文件或文件夹
    guest ok = yes    #任何用户都可以访问
    read only = no    #Windows Explorer 拥有读写权限
    create mask = 0777  #当文件被创建的时候，被赋予的默认权限
　　 valid users = tom       # 指定登录的用户，该项不写，则默认对所有人可见
　　 force user = nobody     # 指定的用户可以进行登录，其他用户没有权限登录，nobody不限制
　 　force group = nogroup   # 同上，指定用户组
　　 public = yes            # 是否对所有登录成功的用户可见
　　 writable = yes          # 写权限，目录的权限也要许可
　　 available = yes         # 同样是设置共享目录是否可见
```

## 创建待共享的文件夹路径，并赋予权限：

```shell
sudo mkdir -p /srv/samba/share  #如果父文件夹不存在，则创建父文件夹
sudo chown nobody:nogroup /srv/samba/share/
```

## 创建Samba用户

```shell
sudo smbpasswd -a tom
```

然后输入密码即可

## 重启 `samba` 系统服务让配置生效

```shell
sudo systemctl restart smbd.service nmbd.service
```

## 设置访问权限

再次强调，以上配置，在局域网内所有的访问客户端都被赋予了所有权限，如果需要配置访问权限，请按照以下说明修改：

- ### 编辑 `/etc/samba/smb.conf` 

- ### 添加以下网络配置：

```shell
# hosts allow = 127.0.0.1 192.168.1.0/24
hosts allow = 127.0.0.1 192.168.1.1 192.168.1.2
hosts deny = 0.0.0.0/0    #拒绝其他所有人
```

- ### 分享权限：

browseable = no ，代表当客户端访问的时候，无法浏览文件或文件夹

users = user1 user2 ，代表仅列出的用户可以访问此分享 

可以参考如下配置：

```shell
[private]
        comment = Private Share
        path = /path/to/share/point
        browseable = no
        read only = no
        valid users = user1 user2 user3  # user1 user2和user3 访问私有分享
```

## 设置防火墙

可以配置防火墙（如 `iptables` ）来限制访问你的服务器，`Samba` 使用以下端口

```shell
UDP : 137 和 138
TCP : 139 和 445
```

# 查看效果

在Windows下通过samba访问服务器内容

![](/images/1231232.png)