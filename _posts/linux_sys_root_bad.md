---
layout: post
title: Linux 系统权限不正常导致系统无法启动
tags: [Linux]
index_img: https://th.wallhaven.cc/small/ox/ox2xdl.jpg
banner_img: https://w.wallhaven.cc/full/ox/wallhaven-ox2xdl.png
categories: [Linux]
date: 2019-08-17 10:51:56
---
# 问题描述

##  系统卡在登录界面，debug 模式显示多项 [FAILED]

![](/img/linux_sys_root_bad/linux_sys_root_bad_1.jpg)

<!-- more -->

# 告警信息
## 1 远程ssh普通用户能够登入，但是无法切换 root 用户

```
[user@location~]$ sudo su -
sudo: unknow user:root
sudo: unable to initialize policy plugin
```

## 2 普通用户下使用 ps -ef 查看用户进程 UID 显示为 ixdba

![](/img/linux_sys_root_bad/linux_sys_root_bad_2.jpg)

正常为：

![](/img/linux_sys_root_bad/linux_sys_root_bad_3.jpg)



# 处理过程

## 1 查看 /etc/passwd

发现 `root:x:0:0:root:/root:/bin/bash`-> `ixdba:x:0:0:root:/root:/bin/bash`

## 2 将 root 用户名改回

进单用户模式，修改  /etc/passwd `ixdba:x:0:0:root:/root:/bin/bash`-> `root:x:0:0:root:/root:/bin/bash`

# 根因

root 用户被改成其他名称，如本例中的 ixdba。

服务在启动过程中，需要 root 用户名及 root 组来授权（root，root），修改后，服务找不到 root 用户，无法得到授权，所以很多服务系统无法启动。

# 解决方案

将 root用户名称改回 root，避免修改 root 用户名。