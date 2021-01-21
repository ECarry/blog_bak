---
layout: post
title: Zabbix 5.0 LTS 添加主机及监控
tags: [Zabbix]
index_img: /img/zabbix/zabbix_logo.jpg
banner_img: https://w.wallhaven.cc/full/5w/wallhaven-5w26x1.jpg
categories: [Zabbix]
date: 2020-10-10 09:18:28
---

服务端及客户端安装配置文档：[在 CentOS 7 下 搭建 Zabbix 5.0 LTS 日志](https://ecarry.cc/2020/08/27/zabbix_install/)

---------------------------------------------------

# 一、 添加主机

首先创建**主机群组**，如 web组、DB组 等等

![](/img/zabbix_host/zabbix_host_3.jpg)

打开 WEB UI，在 **配置**--> **主机** 选项

![](/img/zabbix_host/zabbix_host.png)

点击添加主机

![](/img/zabbix_host/zabbix_host_4.jpg)

添加完主机，如图所示：

![](/img/zabbix_host/zabbix_host_5.jpg)

刚添加完主机，`Availability` 的 ZBX 可能还是灰色，需要等待几分钟或者可以通过添加监控项，可以让 server 端快速连上 agent

# 二、 添加监控

## 2.1 应用集

可以将一系列的监控项作为一个集合，比如监控系统状态，可创建 system status 应用集：

- system status
  - CPU
  - MEM
  - DISK

## 2.2 监控项

![](/img/zabbix_host/zabbix_host_6.jpg)

点击 **监控项**，右上角点击 创建监控项：

![](/img/zabbix_host/zabbix_host_7.jpg)

**示例：**

添加一个监控项，监控 CPU 中断数：

![](/img/zabbix_host/zabbix_host_8.jpg)

![](/img/zabbix_host/zabbix_host_9.jpg)

数据展示在：

![](/img/zabbix_host/zabbix_host_10.jpg)
![](/img/zabbix_host/zabbix_host_11.jpg)

**参数在中文显示下会乱码**

## 2.3 解决中文乱码问题

![](/img/zabbix_host/zabbix_host_12.jpg)

英文下正常显示：

![](/img/zabbix_host/zabbix_host_13.jpg)

可将 windows 下的带中文的字体文件上传到 `/usr/share/zabbix/assets/fonts`，删除原字体的软链 `rm -f /etc/alternatives/zabbix-web-font`，创建新字体的软链

```shell
ln -s /usr/share/zabbix/assets/fonts/SIMSUN.TTC /etc/alternatives/zabbix-web-font
ll
lrwxrwxrwx. 1 root root 41 1月  21 10:55 /etc/alternatives/zabbix-web-font -> /usr/share/zabbix/assets/fonts/SIMSUN.TTC
```

重启 `systemctl restart zabbix-server`，重新打开网页，正常显示：

![](/img/zabbix_host/zabbix_host_14.jpg)