---
layout: post
title: CentOS 7 更改网卡名称
tags: [Linux]
index_img: https://th.wallhaven.cc/small/5d/5dl5m1.jpg
banner_img: https://w.wallhaven.cc/full/5d/wallhaven-5dl5m1.jpg
categories: [Linux]
date: 2020-03-2 20:09:52
---

Redhat 7 官方文档介绍命名规则

CentOS 7 默认网卡名大多为：ens，已经不是我们熟悉的 eth ，从centos7开始，网卡命名会根据固件，拓扑结构和位置信息来确定。

------------------------------------

# 一、修改步骤

## 1.1 编辑网卡的配置文件

```bash
[root@test ~]# vi /etc/sysconfig/network-scripts/ifcfg-ens18
NAME=eth0           <--- 改为想要的名称，如 eth0，下同
DEVICE=eth0
```

## 1.2 重命名网卡配置文件

```bash
[root@localhost network-scripts]# mv ifcfg-ens18 ifcfg-eth0
```

## 1.3 修改内核参数配置文件

`vi /etc/default/grub` ，添加红色框内内容，禁用 CentOS7 上的命名规则

![](/img/linux_change_ifname/linux_change_ifname_1.jpg)

## 1.4 执行命令

`grub2-mkconfig -o /boot/grub2/grub.cfg` 来重新生成 grub 配置并更新内核参数。重启设备可生效

## 1.5 创建网卡接口命名规则

`vim /etc/udev/rules.d/70-persistent-ipoib.rules`

添加内容如下一行：

`UBSYSTEM=="net",ACTION=="add",DRIVERS=="?",ATTR{address}=="",ATTR{type}=="1",KERNEL=="eth*",NAME="eth0"`

这时重启再添加网卡其名称就会自动变成eth1、eth2 … 依次类推。