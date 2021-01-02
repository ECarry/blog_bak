---
layout: post
title: Linux 系统性能分析标准
tags: [Linux]
index_img: https://th.wallhaven.cc/small/k9/k9y23m.jpg
banner_img: https://w.wallhaven.cc/full/k9/wallhaven-k9y23m.jpg
categories: [Linux]
date: 2019-09-17 09:26:15
---

| 影响性能因素 | 评判标准  | 评判标准  | 评判标准  |
| :----:| :----: | :----: | :----:|
|  | 好 | 坏 | 糟糕 |
| CPU | user% + sys% < 70% | user% + sys% = 85% | user% + sys% >= 90% |
| 内存 | Swap In(si) & Out(so)=0 | Per CPU with 10 page/s | more Swap In &Swap Out |
| 磁盘 | iowait% < 20% | iowait% = 35% | iowait% >= 50% |

### 其中
%user：表示 CPU 处在用户模式下的时间百分比。
%sys：表示 CPU 处在系统模式下的时间百分比。
%iowait：表示 CPU 等待输入输出完成时间的百分比。
swap in：即si，表示虚拟内存的页导入，即从SWAP DISK 交换到 RAM。
