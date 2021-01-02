---
layout: post
title: Linux 系统启动详细流程
tags: [Linux]
index_img: https://th.wallhaven.cc/small/xl/xldd5z.jpg
banner_img: https://w.wallhaven.cc/full/xl/wallhaven-xldd5z.png
categories: [Linux]
date: 2020-06-01 20:50:21
---

POST --> BIOS(Boot Sequence 启动顺序) --> [MBR](https://ecarry.cc/2020/06/09/Linux_part/)(Bootloader,446) --> Kernel --> initrd(init ram disk,Linux初始RAM磁盘) --> /sbin/init 

