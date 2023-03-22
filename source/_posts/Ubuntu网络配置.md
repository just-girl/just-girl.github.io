---
title: Ubuntu网络配置
tags: ["Linux"]
categories: Linux
date: 2019-03-15
grammar_cjkRuby: true
copyright: ture
---

## 以太网接口

### 识别以太网接口的两种方法：

 <!-- more -->

```shell
justme@justserver:$ ifconfig -a
eno1np0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.125  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::4ed9:8fff:fe2c:cb98  prefixlen 64  scopeid 0x20<link>
        ether 4c:d9:8f:2c:cb:98  txqueuelen 1000  (Ethernet)
        RX packets 24440107  bytes 35187073438 (35.1 GB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 11198489  bytes 800420673 (800.4 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

eno2np1: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether 4c:d9:8f:2c:cb:99  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 7094  bytes 793776 (793.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 7094  bytes 793776 (793.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

```shell
justme@justserver:~/AndroidSource/lineage_17.0$ sudo lshw -class network
[sudo] password for justme:
  *-network:0
       description: Ethernet interface
       product: BCM57416 NetXtreme-E Dual-Media 10G RDMA Ethernet Controller
       vendor: Broadcom Inc. and subsidiaries
       physical id: 0
       bus info: pci@0000:18:00.0
       logical name: eno1np0
       version: 01
       serial: 4c:d9:8f:2c:cb:98
       size: 1Gbit/s
       capacity: 10Gbit/s
       width: 64 bits
       clock: 33MHz
       capabilities: pm vpd msi msix pciexpress bus_master cap_list rom ethernet physical tp 1000bt-fd 10000bt-fd autonegotiation
       configuration: autonegotiation=on broadcast=yes driver=bnxt_en driverversion=1.10.0 duplex=full firmware=214.0.222.1/pkg 21.40.22.21 ip=192.168.1.125 latency=0 link=yes multicast=yes port=twisted pair speed=1Gbit/s
       resources: irq:72 memory:9da10000-9da1ffff memory:9d900000-9d9fffff memory:9da22000-9da23fff memory:9de00000-9de7ffff
  *-network:1 DISABLED
       description: Ethernet interface
       product: BCM57416 NetXtreme-E Dual-Media 10G RDMA Ethernet Controller
       vendor: Broadcom Inc. and subsidiaries
       physical id: 0.1
       bus info: pci@0000:18:00.1
       logical name: eno2np1
       version: 01
       serial: 4c:d9:8f:2c:cb:99
       capacity: 10Gbit/s
       width: 64 bits
       clock: 33MHz
       capabilities: pm vpd msi msix pciexpress bus_master cap_list rom ethernet physical tp 1000bt-fd 10000bt-fd autonegotiation
       configuration: autonegotiation=on broadcast=yes driver=bnxt_en driverversion=1.10.0 duplex=half firmware=214.0.222.1/pkg 21.40.22.21 latency=0 link=no multicast=yes port=twisted pair
       resources: irq:81 memory:9da00000-9da0ffff memory:9d800000-9d8fffff memory:9da20000-9da21fff memory:9de80000-9defffff
```

### 以太网接口逻辑名称

接口逻辑名称也可以通过 `netplan` 配置进行配置。如果要控制哪个接口收到特定的逻辑名称，请使用 `match` 和 `set-name` 键。`match` 键用于根据某些条件（例如MAC地址，驱动程序等）查找适配器。然后，`set-name` 键可用于将设备更改为所需的逻辑名称。

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
    eth_lan0:
      dhcp4: true
	  match:
	    macaddress: 00:11:22:33:44:55
	  set-name: eth_lan0
```

### 以太网接口设置

ethtool 是一个程序，用于显示和更改以太网卡设置，例如自动协商，端口速度，双工模式和局域网唤醒。以下是如何查看以太网接口的受支持功能和配置设置：

```shell
sudo ethtool eth4
Settings for eth4:
	Supported ports: [ FIBRE ]
	Supported link modes:   10000baseT/Full
	Supported pause frame use: No
	Supports auto-negotiation: No
	Supported FEC modes: Not reported
	Advertised link modes:  10000baseT/Full
	Advertised pause frame use: No
	Advertised auto-negotiation: No
	Advertised FEC modes: Not reported
	Speed: 10000Mb/s
	Duplex: Full
	Port: FIBRE
	PHYAD: 0
	Transceiver: internal
	Auto-negotiation: off
	Supports Wake-on: d
	Wake-on: d
	Current message level: 0x00000014 (20)
			               link ifdown
	Link detected: yes
```

## IP 地址

### 临时IP地址分配

对于临时网络配置，可以使用 `ip` 命令，该命令在大多数其他 GNU / Linux 操作系统上也可以找到。 ip 命令使您可以配置立即生效的设置，但是这些设置不是永久性的，并且在重启后将丢失。

要临时配置 IP 地址，可以按以下方式使用 `ip` 命令。修改 IP 地址和子网掩码以符合您的网络要求。

```shell
sudo ip addr add 10.102.66.200/24 dev enp0s25
```

然后可以使用 `ip` 来设置链接 up 或 down

```shell
ip link set dev enp0s25 up
ip link set dev enp0s25 down
```

要验证 enp0s25 的 IP 地址配置，可以按以下方式使用 `ip` 命令。

```shell
ip address show dev enp0s25
10: enp0s25: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 00:16:3e:e2:52:42 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 10.102.66.200/24 brd 10.102.66.255 scope global dynamic eth0
       valid_lft 2857sec preferred_lft 2857sec
    inet6 fe80::216:3eff:fee2:5242/64 scope link
       valid_lft forever preferred_lft forever6
```

要**配置默认网关**，可以按以下方式使用ip命令。修改默认网关地址以符合您的网络要求。

```shell
sudo ip route add default via 10.102.66.1
```

要验证默认网关配置，可以按以下方式使用 `ip` 命令。

```shell
ip route show
default via 10.102.66.1 dev eth0 proto dhcp src 10.102.66.200 metric 100
10.102.66.0/24 dev eth0 proto kernel scope link src 10.102.66.200
10.102.66.1 dev eth0 proto dhcp scope link src 10.102.66.200 metric 100 
```

如果您需要 DNS 进行临时网络配置，则可以在文件 `/etc/resolv.con`f 中添加 DNS 服务器 IP 地址。通常，不建议直接编辑 `/etc/resolv.conf` ，但这是一个**临时且非永久的配置**。下例显示了如何在 `/etc/resolv.conf` 中输入两个 DNS 服务器，应将其更改为适合您网络的服务器。下一节将更详细地描述进行 DNS 客户端配置的正确的持久方法。

```shell
nameserver 8.8.8.8
nameserver 8.8.4.4
```

如果您不再需要此配置，并且希望从接口清除所有 IP 配置，则可以将 `ip` 命令与 `flush` 选项一起使用，如下所示。

```shell
ip addr flush eth0
```

> 使用 `ip` 命令刷新IP配置不会清除 `/etc/resolv.conf` 的内容。您必须手动删除或修改这些条目，或者重新引导，这也将导致 `/etc/resolv.conf`（它是 `/run/systemd/resolve/stub-resolv.conf`的符号链接）被重写。

## 动态IP地址分配（DHCP客户端）

要将服务器配置为使用 DHCP 进行动态地址分配，请在文件 `/etc/netplan/99_config.yaml` 中创建一个 `netplan` 配置。下面的示例假定您正在配置标识为 `enp3s0` 的第一个以太网接口。

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: true
```

然后可以使用 `netplan` 命令应用该配置。

```shell
sudo netplan apply
```

### 静态IP地址分配

要将系统配置为使用静态地址分配，请在文件 `/etc/netplan/99_config.yaml` 中创建一个 `netplan` 配置。下面的示例假定您正在配置标识为 `eth0` 的第一个以太网接口。更改地址，`gateway4` 和名称服务器值，以满足您的网络要求。

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
     eno1np0:
        dhcp4: no
        dhcp6: no
        addresses:
           - 192.168.1.125/24
        gateway4: 192.168.1.1
        nameservers:
           search: []
           addresses: [192.168.1.1]
```

> eno1np0：指定需配置网络接口的名称。
>
> dhcp4：是否打开 IPv4 的 dhcp。
>
> dhcp6：是否打开 IPv6 的 dhcp。
>
> addresses：定义网络接口的静态 IP 地址。
>
> gateway4：指定默认网关的 IPv4 地址。
>
> nameservers：指定域名服务器的 IP 地址。

然后可以使用 `netplan` 命令应用该配置。

```shell
sudo netplan apply
```

## 环回接口

系统将环回接口标识为 `lo` ，其默认 IP 地址为 127.0.0.1。可以使用 `ip` 命令查看它。

```shell
ip address show lo
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host
       valid_lft forever preferred_lft forever
```

## 名称解析

与 IP 网络相关的名称解析是将 IP 地址映射到主机名的过程，从而更容易识别网络上的资源。下一节将说明如何使用 DNS 和静态主机名记录正确配置系统以进行名称解析。

### DNS客户端配置

传统上，文件 `/etc/resolv.conf` 是静态配置文件，很少需要通过 DCHP 客户端挂接进行更改或自动更改。 `Systemd-resolved` 处理名称服务器配置，并且应该通过 `systemd-resolve` 命令与之交互。 `Netplan` 配置`systemd-resolved` 生成一个名称服务器和域的列表，以放入 `/etc/resolv.conf` 中，这是一个符号链接：

```shell
/etc/resolv.conf -> ../run/systemd/resolve/stub-resolv.conf
```

要配置解析器，请将适合您的网络的名称服务器的 IP 地址添加到 `netplan` 配置文件中。您还可以添加可选的 DNS 后缀搜索列表以匹配您的网络域名。生成的文件可能如下所示：

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s25:
      addresses:
        - 192.168.0.100/24
      gateway4: 192.168.0.1
      nameservers:
          search: [mydomain, otherdomain]
          addresses: [1.1.1.1, 8.8.8.8, 4.4.4.4]
```

搜索选项也可以与多个域名一起使用，以便将 DNS 查询按输入顺序附加。例如，您的网络可能有多个子域可供搜索； example.com 的父域，以及两个子域 sales.example.com 和 dev.example.com 。

如果您要搜索多个域，则配置可能如下所示：

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s25:
      addresses:
        - 192.168.0.100/24
      gateway4: 192.168.0.1
      nameservers:
          search: [example.com, sales.example.com, dev.example.com]
          addresses: [1.1.1.1, 8.8.8.8, 4.4.4.4]
```

如果尝试对名称为 `server1` 的主机执行 `ping` 操作，系统将按照以下顺序自动查询DNS的完全合格域名（FQDN）：

1. server1.example.com

2. server1.sales.example.com

3. server1.dev.example.com

如果找不到匹配项，则 DNS 服务器将提供 `notfound` 的结果，并且 DNS 查询将失败。

#### 静态主机名

静态主机名是本地定义的主机名到 IP 的映射，位于文件 `/etc/hosts` 中。默认情况下，`hosts` 文件中的条目将优先于 DNS 。这意味着，如果您的系统尝试解析主机名并且与 `/etc/hosts` 中的条目匹配，它将不会尝试在 DNS 中查找记录。在某些配置中，尤其是当不需要 Internet 访问时，可以方便地将与有限数量的资源进行通信的服务器设置为使用静态主机名而不是 DNS 。

以下是主机文件的示例，其中已通过简单的主机名，别名及其等效的完全限定域名（FQDN）标识了许多本地服务器。

```shell
127.0.0.1	localhost
127.0.1.1	ubuntu-server
10.0.0.11	server1 server1.example.com vpn
10.0.0.12	server2 server2.example.com mail
10.0.0.13	server3 server3.example.com www
10.0.0.14	server4 server4.example.com file
```

> 在上面的示例中，请注意，除了它们的专有名称和 FQDN 之外，还为每个服务器指定了别名。 Server1 已映射为名称 vpn ，server2 被称为 mail ，server3 被称为 www ， server4 被称为 file 。

#### 名称服务交换机配置

系统选择将主机名解析为 IP 地址的方法的顺序由名称服务交换机（NSS）配置文件 `/etc/nsswitch.conf` 控制。如上一节所述，通常，在系统 `/etc/hosts` 文件中定义的静态主机名优先于从 DNS 解析的名称。以下是文件 `/etc/nsswitch.conf` 中负责主机名查找此顺序的行的示例。

```shell
hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4
```

1. **files** ：首先尝试解析 `/etc/hosts` 中的静态主机名。
2. **mdns4_minimal** ：尝试使用多播 DNS 解析名称。
3. **[NOTFOUND=return]** ：意味着前面的 `mdns4_minimal` 进程未找到的任何响应都应视为权威，并且系统不应尝试继续寻找答案。
4. **dns** ：表示旧式单播 DNS 查询。
5. **mdns4**  ：表示多播 DNS 查询。

要修改上述名称解析方法的顺序，只需将 hosts：字符串更改为您选择的值即可。例如，如果您更喜欢使用传统的单播 DNS 而不是多播 DNS ，则可以在 `/etc/nsswitch.conf` 中更改字符串，如下所示。

```shell
hosts:          files dns [NOTFOUND=return] mdns4_minimal mdns4
```

## 桥接

桥接多个接口是一种更高级的配置，但在多种情况下非常有用。一种情况是建立具有多个网络接口的网桥，然后使用防火墙过滤两个网段之间的流量。另一种情况是在具有一个接口的系统上使用网桥，以允许虚拟机直接访问外部网络。以下示例涵盖了后一种情况。

通过编辑 `/etc/netplan/` 中的 `netplan` 配置来配置网桥：

```shell
network:
  version: 2
  renderer: networkd
  ethernets:
    enp3s0:
      dhcp4: no
  bridges:
    br0:
      dhcp4: yes
      interfaces:
        - enp3s0
```

为您的物理接口和网络输入适当的值。

现在应用配置以启用网桥：

```shell
sudo netplan apply
```

现在，新的网桥接口应已启动并正在运行。 `brctl` 提供有关网桥状态的有用信息，控制哪些接口是网桥的一部分，等等。有关更多信息，请参见 `man brctl` 。

## YAML 语言基本语法规则

YAML 语言的设计目标，就是方便人类读写。它实质上是一种通用的数据串行化格式。 YAML 基本语法规则如下:

> 大小写敏感
> 使用缩进表示层级关系
> 缩进时不允许使用Tab键，只允许使用空格。
> 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
> #表示注释，从这个字符一直到行尾，都会被解析器忽略。

### 根据 Netplan 的描述文件手动创建网络守护程序的配置信息

```shell
sudo netplan generate
```

执行后会使用  `/etc/netplan/*.yaml` 生成对应网络守护程序的配置信息。例如：

```shell
$ cat /run/systemd/network/10-netplan-enp0s5.network
[Match]
Name=enp0s5

[Link]
RequiredForOnline=no

[Network]
Address=192.168.100.211/23
Gateway=192.168.100.1
DNS=8.8.8.8
DNS=8.8.4.4
```

### 查看当前系统的 DNS Servers

```shell
$ systemd-resolve --status
Global
          DNSSEC NTA: 10.in-addr.arpa
                      16.172.in-addr.arpa
                      168.192.in-addr.arpa
                      17.172.in-addr.arpa
                      18.172.in-addr.arpa
                      19.172.in-addr.arpa
                      20.172.in-addr.arpa
                      21.172.in-addr.arpa
                      22.172.in-addr.arpa
                      23.172.in-addr.arpa
                      24.172.in-addr.arpa
                      25.172.in-addr.arpa
                      26.172.in-addr.arpa
                      27.172.in-addr.arpa
                      28.172.in-addr.arpa
                      29.172.in-addr.arpa
                      30.172.in-addr.arpa
                      31.172.in-addr.arpa
                      corp
                      d.f.ip6.arpa
                      home
                      internal
                      intranet
                      lan
                      local
                      private
                      test

Link 2 (enp0s5)
      Current Scopes: DNS
       LLMNR setting: yes
MulticastDNS setting: no
      DNSSEC setting: no
    DNSSEC supported: no
         DNS Servers: 8.8.8.8
                      8.8.4.4
```

