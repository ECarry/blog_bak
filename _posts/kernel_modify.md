---
layout: post
title: Linux 简单内核优化
tags: [Linux, kernel]
index_img: https://th.wallhaven.cc/small/od/odqlol.jpg
banner_img: https://w.wallhaven.cc/full/4g/wallhaven-4gwk17.png
categories: [Linux]
date: 2020-05-13 09:15:18
---
# ulimit

ulimit 为 shell 内建指令，可用来控制shell执行程序的资源。

```
[root@localhost ~]# ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 7257
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 7257
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

<!-- more -->

## core file size

core 文件其实就是内存的映像，当程序崩溃时，存储内存的相应信息，主用用于对程序进行调试。当程序崩溃时便会产生 core 文件，其实准确的应该说是 core dump 文件,默认生成位置与可执行程序位于同一目录下，文件名为 core.\*\*\* ，其中\*\*\*为数字。

开启 core 文件生成

> 查看 core 文件生成是否打开，如果为0，则表示没有打开。
> [root@localhost ~]#  ulimit -c
> 0
>
> 临时设置（如下设置2G，单位为kbyte）
> 如果生成的信息超过此大小，将会被裁剪，最终生成一个不完整的core文件。在调试此core文件的时候，**gdb**会提示错误。
> ulimit -c 4194304

## file size

文本文件最大容量，可用 `ulimit -f` + 数字修改，需要限制日志文件大小时，可以修改此项

## open files

系统最大打开文件数量，可用`ulimit -n` + **数字** 更改，一般设置为65536

`ulimit -n 65536`

启动任何进程，都会加载文件。针对 Web 服务器、基于java的服务会加载更多的文件，对于这些服务器，需要修改最大打开文件数。

## max user processes

用户可使用最多的进程数，可用`ulimit -u` + **数字** 更改，一般改为65536

`ulimit -u 65536`



**<u>以上修改为临时修改，需要永久修改需要修改配置文件`/etc/security/limits.conf`</u>**

当修改`/etc/security/limits.d/*.conf` 下的配置文件是，会优先使用此配置文件




