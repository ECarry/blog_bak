---
layout: post
title: Redhat/CentOS 7 单用户修改 root 密码
tags: [Linux]
index_img: https://th.wallhaven.cc/small/83/83r3k2.jpg
banner_img: https://w.wallhaven.cc/full/p2/wallhaven-p2yjz3.png
categories: [Linux]
date: 2019-11-01 21:28:42
---
# 修改步骤

1. 重启系统，在 Grub Boot Loader 倒计时读秒结束前，按下任意键

2. 选择第一项，并按下 e 键，编辑

![](/img/linux_change_pw/linux_change_pw_1.jpg)

3. 进入编辑界面后，找到 linux16 开头的行，通过 Ctrl + e 可以快速定位到行末，空格键后输入 rd.break

![](/img/linux_change_pw/linux_change_pw_2.jpg)

4. 编辑好后，按下 Ctrl + x，引导系统，系统将启动到临时内核 shell 界面，输入以下指令即可修改密码

![](/img/linux_change_pw/linux_change_pw_3.jpg)

**<font color=red>注意：修改密码后，首次重启时间将会比较长，因为系统将对所有文件进行 SeLinux 打标，请勿在打标过程中手动强制重启，否则系统将会永久性损坏导致无法开机。</font>**



