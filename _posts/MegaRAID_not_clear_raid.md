---
layout: post
title: MegaRAID卡无法清除VD ( Discard Cache) 创建raid
tags: [运维]
index_img: https://th.wallhaven.cc/small/95/95wryk.jpg
banner_img: https://w.wallhaven.cc/full/95/wallhaven-95wryk.jpg
categories: [运维]
date: 2019-06-21 16:23:17
---

# 问题描述

## 1.开机提示VD丢失
![](/img/megaraid/VD_lost_1.png)

## 2.点击discard cache无效，无法清除VD，点清除后还是提示有丢失的vd
![](/img/megaraid/VD_lost_2.png)![](/img/megaraid/VD_lost_3.png)
<!-- more -->

## 3.创建raid报错，无法正常创建raid配置
![](/img/megaraid/VD_lost_4.png)

# 告警信息

you cannot perform that operation because the contronller has preserved cache.you must either discard the preserved cache by going to the manage preserved cache operation or import the virtual drives that have preserved cache

# 处理过程

硬核处理过程：
服务器下电，把阵列卡电池拔出放电

或者
1、下载tookit工具引导进去tookit命令行

linux: ~ #./storcli64 -help | grep -i preserved
storcli /cx show preservedcache
storcli /cx/vx delete preservedcache[force]

1../storcli64 /c0 show preservedcache

可以看到是哪个VD有preservedcache

2.根据上面的命令查到的VD编号，例如VD：0，执行：

./storcli64 /c0/v0 delete preservedcache force

清除完成后到raid卡的webbios里把boot error handling恢复原来的设置

# 根因

RAID卡Cache中存在数据，重启服务器或者更换硬盘后，Cache中的数据无法写到硬盘中，导致出现上述问题。

# 解决方案

Cache 数据的保护，一般都依赖于镜像与电池 ( 或者是 UPS)，要是discard cache无效，可以试着拔出阵列卡电池试试。