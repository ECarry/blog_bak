---
layout: post
title: Linux 系统资源 ulimit 设置不正确导致无法登录
tags: [Linux]
index_img: https://th.wallhaven.cc/small/47/47pwoo.jpg
banner_img: https://w.wallhaven.cc/full/47/wallhaven-47pwoo.jpg
categories: [Linux]
date: 2019-07-15 10:56:56
---

# 问题描述

## 本地及远程 ssh 无法登录

ssh登录有显示 last login，但是会闪断，重连。

本地登录为一直重复输入用户名密码。

<!-- more -->

# 告警信息

进入单用户模式，查看登录日志 `tail -f /var/log/secure`

![](/img/linux_ulimit_bad/linux_ulimit_bad_1.jpg)

关键信息

error: PAM: pam_open_session(): Permission denied

pam_limits(sshd:session): Could not set limit for 'nofile': Operation not permitted

# 处理过程

查看 ulimit 配置是否异常：`ulimit -a`
```
[user@localhost~]# cat /etc/security/limits.conf |grep -v "#"

```
查看 ulimit 配置` /etc/security/limits.conf` `/etc/security/limits.d/*.conf`

```
[user@localhost~]# cat /etc/security/limits.d/20-nproc.conf 
# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.

*          soft    nproc     4096
root       soft    nproc     unlimited
*          soft    nofile    65536000
*          hard    nofile    65536000
```
将 nofile 数值改小到 65536


# 根因
nofile 数值设置太大导致溢出，无权限设置过大  Could not set limit for 'nofile': Operation not permitted

# 解决方案
无法登录情况，需要进入单用户模式查看系统配置是否正确：
如：/root/.bash_profile 
本例中，设置资源配置文件 ` /etc/security/limits.conf` `/etc/security/limits.d/*.conf` nofile 设置过大溢出，导致系统无法分配 open files 权限，无法登陆。