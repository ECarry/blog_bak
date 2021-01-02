---
layout: post
title: IP 子网划分
tags: [网络]
index_img: https://w.wallhaven.cc/full/0j/wallhaven-0jz7gm.jpg
banner_img: https://w.wallhaven.cc/full/0j/wallhaven-0jz7gm.jpg
categories: [网络]
date: 2020-07-22 12:21:42
---

## IP地址（Internet Protocol）
在互联网漫游的计算机的身份ID。唯一标识一台网络设备的身份ID。

IP 地址为：网络位 + 主机位 -> 子网掩码下详解

IPV4地址：点分十进制 32bit

192.168.1.1->11000000.10101000.00000001.00000001

|         1         | 1                | 1                | 1                | 1               | 1               | 1               | 1               |
| :---------------: | ---------------- | ---------------- | ---------------- | --------------- | --------------- | --------------- | --------------- |
| 2<sup>7</sup>=128 | 2<sup>6</sup>=64 | 2<sup>5</sup>=32 | 2<sup>4</sup>=16 | 2<sup>3</sup>=8 | 2<sup>2</sup>=4 | 2<sup>1</sup>=2 | 2<sup>0</sup>=1 |
|        128        | 192              | 224              | 240              | 248             | 252             | 254             | 255             |

<!-- more -->

## 子网掩码（Netmask）

子网掩码是一个32位地址，用于屏蔽IP地址的一部分以区别网络标识和主机标识，并说明该IP地址是在局域网上，还是在广域网上。

**例子：**  

ip：192.168.1.1

二进制：11000000.10101000.00000001.00000001

netmask：255.255.255.0

二进制：<font color=green > 11111111.11111111.11111111</font>.<font color=blue > 00000000</font>

根据二进制子网掩码，绿色为网络位，蓝色为主机位，即：

网络位：子网掩码1 bit 对应的位

主机位：子网掩码0 bit 对应的位

网络位为：192.168.1->11000000.10101000.00000001

主机位：00000001



主机位全0：子网地址

主机位全1：子网广播地址

<font color=red >注：主机位全 0 全1 的 ip 地址和掩码的组合是无效的。</font>

**例子：**

192.168.1.127 255.255.255.128

192.168.1.0---1111111

255.255.255.1---0000000



## 网段

具有<font color=red >相同网络位</font>的ip和掩码的组合称为同一个网段（局域网、子网）

<font color=red >注：同一网段的PC互通不需要网关。</font>

**例子：**

已知某个网络的掩码是 255.255.248.0，下面属于同一网段的是:

A. 10.110.16.1 和 10.110.25.1

B. 10.76.129.21 和 10.76.137.1

C.  10.52.57.34 和 10.52.62.2

D. 10.33.23.2 和 10.33.31.1

解：

子网掩码 255.255.248.0 -> 

->二进制 <font color=green >255.255.11111</font>   <font color=blue >000.0</font>

<font color=green >绿色：网络位</font>         <font color=blue >蓝色：主机位</font>

A.

<font color=green >10.110.  00010</font>   <font color=blue >000.1</font>
<font color=green >10.110.  00011</font>   <font color=blue >001.1</font>

B.

<font color=green >10.76.    10000</font>   <font color=blue >001.21</font>
<font color=green >10.76.    10001</font>   <font color=blue >001.1</font>  

C.

<font color=green >10.52.    00111</font>   <font color=blue >001.34</font>
<font color=green >10.52.    00111</font>   <font color=blue >110.2</font>

D.

<font color=green >10.22.    00010</font>   <font color=blue >111.2</font>
<font color=green >10.33.    00011</font>   <font color=blue >111.1</font>

**例子：**

172.16.0.0/16 分成6个子网段：

2^n>=6 n=3 需要3个 bit 的子网位 

172.16.<font color=red >**000**</font>00000.0

1. 172.16.<font color=red >**000**</font>00000.0 --> 172.16.0.0/19
2. 172.16.<font color=red >**001**</font>00000.0 --> 172.16.32.0/19
3. 172.16.<font color=red >**010**</font>00000.0 --> 172.16.64.0/19
4. 172.16.<font color=red >**011**</font>00000.0 
5. 172.16.<font color=red >**100**</font>00000.0 
6. 172.16.<font color=red >**101**</font>00000.0 
7. 172.16.<font color=red >**110**</font>00000.0 
8. 172.16.<font color=red >**111**</font>00000.0 



## 网关（Gateway）

当 PC 访问不同网段的服务时，需要将数据交给网关处理

## TTL 

time to live：生命周期，指定IP包被路由器丢弃之前允许通过的最大网段数量。

## tracert

测试本地到达目标所经过的三层设备

```
C:\Users\ecarry>tracert www.baidu.com

通过最多 30 个跃点跟踪
到 www.a.shifen.com [14.215.177.39] 的路由:

  1     *        *       <1 毫秒 OPENWRT [192.168.2.3]
  2     1 ms    <1 毫秒   <1 毫秒 RT-AC1200G+-3220 [192.168.2.1]
  3     1 ms     1 ms    <1 毫秒 SMBSHARE [192.168.1.1]
  4    15 ms    22 ms     4 ms  100.64.0.1
  5     *        *        *     请求超时。
  6     7 ms     7 ms     6 ms  120.36.57.29
  7     *        *        *     请求超时。
  8    22 ms    22 ms    23 ms  113.96.4.126
  9    23 ms    23 ms     *     98.96.135.219.broad.fs.gd.dynamic.163data.com.cn [219.135.96.98]
 10    20 ms    20 ms    20 ms  14.29.117.246
 11     *        *        *     请求超时。
 12     *        *        *     请求超时。
 13    19 ms    19 ms    18 ms  14.215.177.39
```

## ARP(Address Resolution解析 Protocol)

通过目的 IP 地址，请求对方 MAC 地址的过程

![](/img/ip_netmask/ip_netmask_0.jpg)

一台主机向另外一台主机发送 ARP Request 的目的 MAC 地址为<font color=red >广播 MAC 地址</font>（FFFF.FFFF.FFFF），其请求包（request）结构为

![](/img/ip_netmask/ip_netmask_1.jpg)

查看 APR 缓存表，通过命令`arp -a`查看，通过`arp -d`删除缓存表

```
PC>arp -a
  Internet Address      Physical Address      Type
  192.168.1.30          00d0.582e.0b9b        dynamic
```

ARP 回应包（reply）结构：

![](/img/ip_netmask/ip_netmask_2.jpg)