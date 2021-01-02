---
layout: post
title: Unexpected inconsistency run fsck 处理报告
tags: [Linux]
index_img: https://th.wallhaven.cc/small/xl/xlpl7v.jpg
banner_img: https://w.wallhaven.cc/full/xl/wallhaven-xlpl7v.png
categories: [Linux]
date: 2018-10-20
---

# 1、故障摘要

机器意外宕机重启，出现报错，无法进入系统，如图

![](/img/linux_fsck/Linux_1.png)

# 2、故障具体情况

提示按CTRL+D继续，按了CTRL+D会重启，再次回到这个界面。
![](/img/linux_fsck/Linux_2.png)

    Give root passwd for maintenance
根据提示输入密码进行维护。

# 3、故障分析及处理

正确输入密码后，进入到维护模式。
![](/img/linux_fsck/Linux_3.png)
在修复模式下，键入命令`fsck –y`（后面可加其报错目录，每个报错目录都不相同）
![](/img/linux_fsck/Linux_4.png)
输入后，系统会检测修复硬盘，检测修复时间根据硬盘里数据多少而定，最后检测完毕。输入reboot重启。
一般情况下重启完毕就可以进入登陆界面。

# 4、总结

fsck的命令的几个使用方法

指令：fsck

使用权限 : 超级使用者

使用方式 : fsck [-sACVRP] [-t fstype] [–] [fsck-options] filesys […]

说明 ： 检查与修复 Linux 档案系统，可以同时检查一个或多个 Linux 档案系统

 

参数 ：

filesys ： device 名称(eg./dev/sda1)，mount 点 (eg. / 或 /usr)

 

-t : 给定档案系统的型式，若在 /etc/fstab 中已有定义或 kernel 本身已支援的则不需加上此参数

-s : 依序一个一个地执行 fsck 的指令来检查

-A : 对/etc/fstab 中所有列出来的 partition 做检查

-C : 显示完整的检查进度

-d : 列印 e2fsck 的 debug 结果

-p : 同时有 -A 条件时，同时有多个 fsck 的检查一起执行

-R : 同时有 -A 条件时，省略 / 不检查

-V : 详细显示模式

-a : 如果检查有错则自动修复

-r : 如果检查有错则由使用者回答是否修复
