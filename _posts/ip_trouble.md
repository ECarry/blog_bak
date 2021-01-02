---
layout: post
title: IP 地址发生冲突导致网络故障，排查解决
tags: [网络]
index_img: https://th.wallhaven.cc/small/ey/eymzjk.jpg
banner_img: https://w.wallhaven.cc/full/ey/wallhaven-eymzjk.jpg
categories: [网络]
date: 2020-08-18 23:38:56
---
# **1、故障摘要**
一般 IP 冲突（大多为自动分配地址的 PC 入网，与现有固定 IP 发生冲突）造成设备无法上线或者不断上线和下线，网络推流死机等现象
# **2、故障具体情况**
大多为自动分配地址的 PC 入网，与现有固定 IP 发生冲突

# **3、故障分析及处理**
## 3.1 
将电脑接入局域网，在终端输入：`arp -a` ，查看局域网内所有的 IP，当看到有两个 IP 一样的时候，就可以确定冲突的 IP
## 3.2
当知道冲突 IP 后，用`nbtstat -a ip`(ip 为冲突 ip)命令，查看冲突 IP 的设备信息，找到该设备
## 3.3 
修改该设备 IP 信息
# **4、总结**



