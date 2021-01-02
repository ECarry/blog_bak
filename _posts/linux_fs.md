---
layout: post
title: Linux 文件系统的选择
tags: [Linux]
index_img: https://th.wallhaven.cc/small/2e/2e3z7x.jpg
banner_img: https://w.wallhaven.cc/full/d5/wallhaven-d5697m.jpg
categories: [Linux]
date: 2020-02-11 18:07:22
---

# 文件系统类型

* ext2：linux 下标准文件系统，无日志记录功能。
* ext3：在 ext2 基础上增加了日志记录功能，仅支持32000个子目录。
* exit4：ext4 的后续版本，linux 2.6.28 内核开始支持。无限子目录支持，快速 fsck。
* xfs：高性能文件系统，linux3.10 内核开始默认支持。

![](/img/linux_fs/linux.jpg)


## 查看 linux 内核版本

```bash
[root@blog ~]# uname -a
Linux blog 3.10.0-1127.el7.x86_64 #1 SMP Tue Mar 31 23:36:51 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
```

### redhat/centos 查看具体版本号

```bash
[root@blog ~]# cat /etc/redhat-release 
CentOS Linux release 7.8.2003 (Core)
```

## 文件系统选择

ext4：适用于**读**操作频繁，同时**小文件**众多的业务

xfs：适用于**写**操作频繁业务

ext3：适用于对性能要求不高、数据安全要求不高的业务