---
layout: post
title: TCP UDP 的区别
tags: [网络]
index_img: https://w.wallhaven.cc/full/42/wallhaven-42j6q6.jpg
banner_img: https://w.wallhaven.cc/full/42/wallhaven-42j6q6.jpg
categories: [网络]
date: 2020-07-24 10:07:12
---


# TCP 

可靠传输、面向连接：速度慢，但是准确性高 

## 可靠传输

客户端收到报文后，需要发送 TCP 的 ACK 确认包，并告诉服务端接下来要收到的报文的序号。同时该过程确定了两者传输的 “Windows 窗口”大小。

## 面向连接

如果某应用层协议的四层使用 TCP 端口，那么在正式的数据报文传输之前，需要先建立完连接后才可以传输数据。

建立连接：<font color=red>三次握手</font>---面向连接的高层协议在正式传输数据之前需要先建立连接，建立连接的过程需要来回交互三个报文（[SYN]-[SYN,ACK]-[ACK]）



![](/img/tcp_udp/tcp_udp_4.jpg) 

①次握手 客户端--SYN-->服务器

![](/img/tcp_udp/tcp_udp_1.jpg) 

②次握手 服务器--SYN+ACK-->客户端

![](/img/tcp_udp/tcp_udp_2.jpg) 

③次握手 客户端--ACK-->服务器

![](/img/tcp_udp/tcp_udp_3.jpg) 



# UDP

不可靠传输，非面向连接：速度快，但准确性差


# Wireshark 过滤规则

ip.addr == 59.36.202.3 过滤出包含 59.36.202.3 的报文

ip.src == x.x.x.x 源地址

ip.dst == x.x.x.x 目标

tcp.dstport == 80 过滤出目标端口为 80 端口

tcp.srcport == 80 过滤出源端口为 80 端口

eth.dst == 48:89:e7:ae:71:53 过滤出目标 MAC 地址

eth.src == 48:89:e7:ae:71:53 过滤出源 MAC 地址

http 过滤高层协议

and or not 同时过滤--> tcp or http and (not xxx)

# 常用协议端口号

HTTP : tcp 80 网页浏览

telnet : tcp 23 远程控制

FTP : tcp 20 21 文件传输

RDP : tcp 3389 远程桌面

# 用 telnet 测试某个端口是否开放

未打开：

```
[root@localhost ~]# telnet 192.168.2.89 3389
Trying 192.168.2.89...
telnet: connect to address 192.168.2.89: No route to host

```

打开状态：

```
[root@localhost ~]# telnet 192.168.2.98 3389
Trying 192.168.2.98...


```