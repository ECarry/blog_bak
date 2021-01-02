---
layout: post
title: 规划和安装ESXI
tags: [VMware]
index_img: https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1596917923997&di=618debf5d6fc18c50916aee5fd51670b&imgtype=0&src=http%3A%2F%2F2a.zol-img.com.cn%2Fproduct%2F105_500x2000%2F378%2FceIUwQIvzRtDY.png
banner_img: https://w.wallhaven.cc/full/45/wallhaven-4583r3.jpg
categories: [VMware]
date: 2020-04-07 09:16:37
---

# 1. 规划 vSphere 部署
## 1.1 服务器平台
检查设备兼容性：存储控制器或者网络适配器

可使用 VNware 网上支持搜索的兼容指南（Compatibility Guide）：www.vmware.com/resources/compatibility/
<!-- more -->
## 1.2 存储架构
* 基于光纤通道和以太网光纤通道（FCoE）存储
* 基于iSCSI的存储
* 通过网络文件系统（NFS）访问的存储
* 支持在一种解决方案中使用多种存储协议

## 1.3 网络基础架构
* ESXI 管理网络至少需要一个NIC（网络接口卡），推荐增加1个 NIC 冗余
* vMotion 需要使用一个NIC ，推荐增加1个 NIC 冗余
* 使用 vSphere FT 需要至少一个NIC ，推荐增加1个 NIC 冗余
* 在使用 iSCSI 、 NFS 或 VSAN 的部署环境中，至少还需增加一个NIC，最好为2个
* 需要2个 NIC 来处理来自虚拟机本身的流量



# 2. 部署 ESXI
## 2.1 部署方法
* 交互式安装 ESXI
* 无人干预（脚本化）安装 ESXI
* 自动化分配（vSphere Auto Deploy） ESXI

## 2.2 无人干预 ESXI 安装过程的启动项选择

|  启动选项   |  简要说明  |
| :----: | :----: |
|  ks=cdrom:/path  | 使用CD-ROM 中指定路径的安装脚本。安装脚本会检查所有CD-ROM驱动器，直到发现与所指定路径相匹配的文件  |
| ks=usb  | USB设备中根目录下名称为ks.cfg的安装脚本。安装程序会搜索文件格式为FAT16或FAT32的USB设备 |
| ks=usb:/path  | 使用USB设备上指定路径的安装脚本 |
| ks=protocol:/serverpath  | 使用指定网络位置的安装脚本。支持的协议NFS、HTTP、HTTPS、FTP |

使用脚本安装ESXI，能够提高安装速度，有利于保证所有ESXI主机都有统一的配置。

## 2.3 自动化分配（vSphere Auto Deploy） ESXI

# 知识点
## vSphere 设计需考虑的网络问题
* VLAN支持
* 链路聚合、网络速度（1Gbps或10Gbps）、负载均衡、NIC接口

## 使用 vSphere Auto Deploy 部署 ESXI 的优缺点
### 优点
可以快速分配、快速重分配及在分配过程中快速增加新 ESXI 映像或更新。
### 缺点
增加复杂性及需要额外的配置，解决部署无状态问题

