---
layout: post
title: Linux 运维故障排查与系统调优技巧 
tags: [Linux]
index_img: https://th.wallhaven.cc/small/kw/kweqx6.jpg
banner_img: https://w.wallhaven.cc/full/3k/wallhaven-3k953v.jpg
categories: [Linux]
date: 2019-10-24 17:47:09
---

# 一、 排障

## 1.1 排障要点

1. 确定问题故障特征
2. 重现故障
3. 使用工具收集进一步信息
4. 排除不可能的原因
5. 定位故障：从简单的问题入手；一次尝试一中方式

## 1.2 常见故障

- 管理员密码忘记
- 系统无法正常启动
  - 1. grub 损坏 （MBR损坏、grub 配置文件丢失）
  - 2. 系统初始化故障（某文件系统无法正常挂载、驱动不兼容）
  - 3. 服务故障
  - 4. 用户无法登陆系统（密码是否正确、bash程序是否故障）
- 命令无法运行
- 编译过程无法继续（开发环境缺少基本组件）

### 1.2.1 MBR/grub 损坏

#### 实验

将 mbr 分区手动损坏

```bahs
#先备份 mbr 分区
[root@test ~]# dd if=/dev/sda of=/root/mbr.backup count=1 bs=512
1+0 records in
1+0 records out
512 bytes (512 B) copied, 0.000588373 s, 870 kB/s
#往 mbr 写入200 bytes 空数据
1+0 records in
1+0 records out
200 bytes (200 B) copied, 0.000351164 s, 570 kB/s
#同步到磁盘
[root@test ~]# sync
#重新启动，系统无法进入
```

**解决办法：**

使用紧急救援模式：使用完整的系统安装光盘

- boot:linux rescue
  - /mnt/sysimage
- chroot /mnt/sysroot

```bash
sh-4.2# chroot /mnt/sysimage
sh-4.2# grub2-install /dev/vda
Installing for i386-pc platform.
Installation finished. No error reported.
sh-4.2# exit
sh-4.2# reboot
```

TODO....

### 1.2.2 



































# 更改主机名（hostname）

在更改主机名时将主机名添加至 `/etc/hosts`下，做个映射

`127.0.0.1		hostname` 

这样能够避免后续安装应用所出现的问题

<!-- more -->

# SSH 设置

ssh 配置文件路径，修改配置文件前记得备份文件

`cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`

`/etc/ssh/sshd_config`



## 关闭 GSSAPI 验证，可以提高 ssh 连接速度

 `GSSAPIAuthentication no`

## 不使用 DNS 反查，可以提高 ssh 连接速度

 `UseDNS no`



# yum 设置

## 常用 yum 源

* epel源：https://fedoraproject.org/wiki/EPEL 
* repoforge 源：http://repoforge.org/use/ 


# 排障关注点

* /var/log/messages	\#应用日志查询
* /var/log/secure         \#登录日志查询
* dmesg                        \#系统日志查询
* /var/tmp 、/tmp       #容易攻击点查询
* crontab -l 、/etc/crontab    \#计划任务查询（经常攻击对象）

## dmesg 命令、文件

直接输出当前最新的系统日志。

系统每次开机将日志保存到`/var/log/dmesg`

旧的日志会保存到`/var/log/dmesg.old`

## secure 文件

ssh 登录日志

可以通过过滤查看登录成功的日志

`tail -1000 /var/log/secure |grep Accepted`

## temp 文件夹（/var/tmp 、/tmp ）

通过`ls -al`查看 temp 文件是否有异常文件

## crontab

通过`crontab -l`查看是否有异常的计划任务

系统计划任务配置文件在`/ect/crontab`，目录`/etc/cron.*/`里放的脚本科自动执行，也要检查