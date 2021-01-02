---
layout: post
title: Linux nmcli 使用及网络配置
tags: [Linux]
index_img: https://th.wallhaven.cc/small/0p/0pl2ej.jpg
banner_img: https://w.wallhaven.cc/full/0p/wallhaven-0pl2ej.png
categories: [Linux]
date: 2020-03-26 10:42:52
---

# 一、nmcli 命令和网络配置文件对应关系

|nmcli connection modify * | ifcfg-* 文件|
| :---: |:---: |
|ipv4.method manual	| BOOTPROTO=none|
|ipv4.method auto|BOOTPROTO=dhcp|
|ipv4.addresses 192.168.1.1/24|IPADDR=192.168.2.230,PREFIX=24|
|ipv4.dns 8.8.8.8|DNS1=8.8.8.8|
|ipv4.dns-seach example.com	 | DOMAIN=example.com|
|ipv4.ignore-auto-dns true |	PEERDNS=no|
|connection.autoconnect yes |	ONBOOT=yes|
|connection.id eth0 | NAME=eth0|
|connection.interface-name eth0	| DEVICE=eth0|
|802-3-ethernet.mac-address …	| HWADDR=…|

<font color=red>注 \*：网卡名称</font>

可以使用 `nmcli device` 查看网卡

## 1.1 应用

```bash
[root@test ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens18 
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=dhcp
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens18
DEVICE=ens18
[root@test ~]# nmcli connection modify ens18 ipv4.addresses 192.168.2.230/24     <---配置ip地址和子网掩码
[root@test ~]# nmcli connection modify ens18 ipv4.dns 114.114.114.114            <---添加dns
[root@test ~]# nmcli connection modify ens18 +ipv4.dns 8.8.8.8                   <---添加dns
[root@test ~]# nmcli connection modify ens18 ipv4.method manual                  <---将ip获取方式改为手动
[root@test ~]# nmcli connection modify ens18 ipv4.gateway 192.168.2.1            <---配置网关地址
[root@test ~]# cat /etc/sysconfig/network-scripts/ifcfg-ens18                   
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens18
DEVICE=ens18
ONBOOT=yes
IPADDR=192.168.2.230
PREFIX=24
DNS1=114.114.114.114
DNS2=8.8.8.8
GATEWAY=192.168.2.1
```

# 二、主机名

默认主机名一般为：localhost.localdomain

修改主机名：

```bash
[root@localhost ~]# hostnamectl set-hostname test
[root@localhost ~]# cat /etc/hostname
test
```

# 三、nmcli 命令的使用

## 3.1 显示网卡或某个网卡具体信息信息

```bash
[root@test ~]# nmcli connection
NAME   UUID                                  TYPE      DEVICE 
ens18  d43b7a46-0dff-9d53-1068-ccc58c977db3  ethernet  ens18  
[root@test ~]# nmcli con show
NAME   UUID                                  TYPE      DEVICE 
ens18  d43b7a46-0dff-9d53-1068-ccc58c977db3  ethernet  ens18  
[root@test ~]# nmcli con show ens18
connection.id:                          ens18
connection.uuid:                        d43b7a46-0dff-9d53-1068-ccc58c977db3
connection.stable-id:                   --
connection.type:                        802-3-ethernet
connection.interface-name:              ens18
```

## 3.2 显示所有设备状态

```bash
[root@test ~]# nmcli device status
DEVICE  TYPE      STATE      CONNECTION 
ens18   ethernet  connected  ens18      
lo      loopback  unmanaged  --  `
```

## 3.3 重启网络使配置文件生效

1. `systemctl restart network`
2. `nmcli connection reload`

## 3.4 显示所有活动连接

```bash
[root@test ~]# nmcli connection show --active
NAME   UUID                                  TYPE      DEVICE 
ens18  d43b7a46-0dff-9d53-1068-ccc58c977db3  ethernet  ens18
```

## 3.5 删除和添加一个网卡连接

```bash
[root@test ~]#  nmcli connection delete eth0 
[root@test ~]#  nmcli connection add type ethernet con-name eth0 ifname eno33554992
```

## 3.6 网络接口的停用和启用

```bash
[root@test ~]#  nmcli connection down eth0 
[root@test ~]#  nmcli connection up eth0
```