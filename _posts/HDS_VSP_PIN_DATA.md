---
layout: post
title: HDS VSP 双盘故障（存在 pin data）
tags: [运维]
index_img: https://th.wallhaven.cc/small/j8/j8xr7w.jpg
banner_img: https://w.wallhaven.cc/full/j8/wallhaven-j8xr7w.jpg
categories: [运维]
date: 2020-08-24 20:04:42
---

# 问题描述
接客户邮件，HDS VSP 有故障，到现场查看，有块硬盘故障，操作界面发现 Pinned Track 闪烁，点开如下图：

![](/img/HDS_VSP/HDS_VSP_PIN_DATA_pin.jpg)

未理睬，接下来正常换故障硬盘，故障硬盘为 HDD24-08：

![](/img/HDS_VSP/HDS_VSP_PIN_DATA_2.jpg)

换完硬盘后一切顺利，故障盘也有回拷进度条：

![](/img/HDS_VSP/HDS_VSP_PIN_DATA_3.jpg)


# 告警信息

第二天，客户电话联系，HDS VSP 有故障，到现场查看为昨天更换的硬盘位，硬盘状态为 **Blocked**：

![](/img/HDS_VSP/HDS_VSP_PIN_DATA_4.jpg)

首先以为是备件硬盘质量问题，相继更换不同批次备件5次，故障依旧，无法从热备盘 **HDD054-0F** 回拷数据，查看 SIM 信息，发现都是回拷不到一小时中断

# 分析

经原厂工程师分析故障原因：
7月23日存储硬盘 **HDD024-08** 完全故障，再通过RAID机制向热备盘 **HDD54-0F** 恢复数据的过程中发现同一RAID组中的 **HDD020-08** 硬盘故障，导致有部分数据无法恢复到 **HDD54-0F**，在00:27这个数据lun产生了pin data。此RAID组发生了两块磁盘同时故障的问题，导致RAID保护机制失效。

![](/img/HDS_VSP/HDS_VSP_PIN_DATA_5.jpg)

当时 **HDD20-08** SIM 信息：
![](/img/HDS_VSP/HDS_VSP_PIN_DATA_6.jpg)

**Pin data** 信息：

![](/img/HDS_VSP/HDS_VSP_PIN_DATA_7.jpg)

# 处理过程

## 准备工作
 
1. 备份 lun 00:27 所涉及的数据进行备份
2. 单独对 lun 00:27 进行备份

?
删除操作

## 处理步骤

1. 开启存储 MODE MODE 模式，开启方式 ?，密码为 mode
2. 进入 **install** ---> **change config？** ，将 **(SOMs)System option modes** 改为 Mode 22
3. 在存储中将 **HDD20-08** 硬盘上的数据拷到 Spare disk
    - 步骤：Mantenance --> 选中硬盘 HDD20-08 --> Other --> Spare Disk
    - 此时，提示 Pinned Track exists. Do you want to stop this process?
    - 点击 No，输入密码 exist-pintrack
4. 等待 Copy end，更换 HDD20-08 硬盘并回拷数据
5. 检查是否产生新的 pin data，如有则需要再次检查系统日志
6. 没有产生新的 pin data，则关闭 Mode 22 后格式化 lun 00:27。格式化完成后原本的 pin data 消失

# 根因
















