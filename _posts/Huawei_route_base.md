---
layout: post
title: 华为 HCNA 学习笔记
tags: [HCNA]
index_img: /img/Huawei_route/Huawei_route_base.jpg
banner_img: https://w.wallhaven.cc/full/45/wallhaven-45zdx1.png
categories: [网络]
date: 2020-07-26 22:21:42
---

华为认证网络工程师是由华为公司认证与采购部推出的独立认证体系，与之前的华为认证不同，简称HCNA。

# 一、基本操作

## 1.1 华为路由/交换视图切换

\<Huawei> 用户视图 ----> [Huawei] 系统视图

```
<Huawei>system-view 
Enter system view, return user view with Ctrl+Z.
[Huawei]
```

## 1.2 查看路由信息

版本：

```
<Huawei>dis version 
Huawei Versatile Routing Platform Software
VRP (R) software, Version 5.110 (eNSP V100R001C00)
Copyright (c) 2000-2011 HUAWEI TECH CO., LTD

```

操作系统 <font color=red>VRP</font>---> Huawei Versatile Routing Platform Software

查看 flash（操作系统及配置文件都放在 flash 里面）：

```
<Huawei>dir				#显示所有文件
Directory of flash:/

  Idx  Attr     Size(Byte)  Date        Time       FileName 
    0  drw-              -  Aug 07 2015 13:51:14   src
    1  drw-              -  Jul 31 2020 14:34:44   pmdata
    2  drw-              -  Jul 31 2020 14:34:50   dhcp
    3  -rw-             28  Jul 31 2020 14:34:50   private-data.txt
    4  -rw-            452  Jul 31 2020 14:39:27   vrpcfg.zip			#路由配置信息，配置完路由器 save 后生成的文件
32,004 KB total (31,995 KB free)
```

文件操作命令：

```
delete vrpcfg.zip		#删除配置文件
rename vrpcfg.zip vrpcfg.zip.bak		#重命名配置文件
copy vrpcfg.zip vrpcfg.zip.bak	#复制文件
mkdir	demo	#创建文件夹
cd demo			#进入demo文件夹
unzip xx.zip 	#解压
more xx			#查看文件内容
带 [ ] 表示位于回收站里的文件
reset recycle-bin  #清空回收站
```



# 二、配置端口

```
[Huawei]int e0/0/0			#进入端口
[Huawei-Ethernet0/0/0]ip add 192.168.1.1 24	#配置ip和24位子网掩码
[Huawei-Ethernet0/0/0]dis this		#显示接口信息
#
interface Ethernet0/0/0
 ip address 192.168.1.1 255.255.255.0
#
return
[Huawei-Ethernet0/0/0]undo ip address #删除接口信息
```



## 2.1 显示接口摘要

```
[Huawei]display ip int brief 
*down: administratively down
!down: FIB overload down
^down: standby
(l): loopback
(s): spoofing
(d): Dampening Suppressed
The number of interface that is UP in Physical is 3
The number of interface that is DOWN in Physical is 8
The number of interface that is UP in Protocol is 3
The number of interface that is DOWN in Protocol is 8

Interface                         IP Address/Mask      Physical   Protocol  
Ethernet0/0/0                     192.168.1.1/24       up         up        
Ethernet0/0/1                     172.16.1.1/24        up         up        
GigabitEthernet0/0/0              unassigned           down       down      
GigabitEthernet0/0/1              unassigned           down       down      
GigabitEthernet0/0/2              unassigned           down       down      
GigabitEthernet0/0/3              unassigned           down       down      
NULL0                             unassigned           up         up(s)     
Serial0/0/0                       unassigned           down       down      
Serial0/0/1                       unassigned           down       down      
Serial0/0/2                       unassigned           down       down      
Serial0/0/3                       unassigned           down       down
```



## 2.2 显示路由表

> 路由器转发数据包的唯一依据，是路由器转发数据包的一张“地图”

```
[Huawei]display ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 6        Routes : 6        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  Direct  0    0           D   172.16.1.1      Ethernet0/0/1
     172.16.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/1
    192.168.1.0/24  Direct  0    0           D   192.168.1.1     Ethernet0/0/0
    192.168.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0
```



## 2.3 关闭信息中心防止弹出日志

```
[Huawei]undo info-center enable 
Info: Information center is disabled.
```



# 三、配置路由

![](/img/Huawei_route/Huawei_route_base_route.jpg)



## 3.1 直连路由（Direct routing）

> 路由器接口所连接的子网的路由方式称为直连路由

直接连接的路由，当路由器的接口配置好 ip 地址并且 up 后，会自动创建该路由。路由器默认情况下，只能到达直连的网段。

列：路由器 R2 ，只能在直连网段 12.1.1.x 和 23.1.1.x 下通信

![](/img/Huawei_route/Huawei_route_base_1.jpg)

其路由表：

![](/img/Huawei_route/Huawei_route_base_2.jpg)



## 3.2 静态路由（Static routing）

> 一种路由的方式，路由项（routing entry）由手动配置，而非动态决定。

PC1 与 PC2 通信，需要配置静态路由：

![](/img/Huawei_route/Huawei_route_base_3.jpg)

PC1 ---> PC2

 <font color=pink>src:192.168.1.2 ---> dst:172.16.1.2</font>

R1：

[R1]ip route-static <font color=red>172.16.1.0 24</font> <font color=blue>12.1.1.2</font>

<font color=red>红色</font>：目标网段

<font color=blue>蓝色</font>：下一跳（next-hop），下一个传递者、承接者

路由表：

```
[R1]display ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 7        Routes : 7        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       12.1.1.0/24  Direct  0    0           D   12.1.1.1        Ethernet0/0/1
       12.1.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/1
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  Static  60   0          RD   12.1.1.2        Ethernet0/0/1
    192.168.1.0/24  Direct  0    0           D   192.168.1.1     Ethernet0/0/0
    192.168.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0
```



R2：

[R2]ip route-static <font color=red>172.16.1.0 24</font> <font color=blue>23.1.1.3</font>

回包路由配置：

PC2 ---> PC1

<font color=pink>dst:192.168.1.2 \<--- src:172.16.1.2</font>

R3:

[R3]ip route-static 192.168.1.0 24 23.1.1.2

R2:

[R2]ip route-static 192.168.1.0 24 12.1.1.1



## 3.3 路由优先级（Preference）

> 思科：管理距离。衡量路由的优先程度，到达同款同一目标有两种协议，此时优选路由优先级较小的路由协议。

路由优先级范围：0-255

常见路由协议默认的优先级（数字越小，优先级越高）：

* 直连路由 0
* 静态路由 60
* Rip（路由信息协议） 100
* Ospf（Open Shortest Path First开放式最短路径优先） 10

禁用端口：

```
[Huawei]int e0/0/1 
[Huawei-Ethernet0/0/1]shutdown		#关闭端口
[Huawei-Ethernet0/0/1]undo shutdown #开启端口
```



![](/img/Huawei_route/Huawei_route_base_4.jpg)

目标：实现千兆 Ge 0/0/0 作为主链路，百兆 E 0/0/0 作为备份链路

R1:

```
[R1]ip route-static 210.1.1.0 24 21.1.1.2 preference 50
```

CT：

```
[ct]ip route-static 192.168.1.0 24 21.1.1.1 preference 50
```




调整优先级：

```
[Huawei]ip route-static 210.1.1.0 24 21.1.1.2 preference 50
```

<font color=red>达到某相同目标网段，路由表始终放置最优路由</font>

```
[Huawei]dis ip rou
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 9        Routes : 9        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

       12.1.1.0/24  Direct  0    0           D   12.1.1.1        Ethernet0/0/1
       12.1.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/1
       21.1.1.0/24  Direct  0    0           D   21.1.1.1        GigabitEthernet0/0/0
       21.1.1.1/32  Direct  0    0           D   127.0.0.1       GigabitEthernet0/0/0
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
    192.168.1.0/24  Direct  0    0           D   192.168.1.1     Ethernet0/0/0
    192.168.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0
      210.1.1.0/24  Static  50   0          RD   21.1.1.2        GigabitEthernet0/0/0
```

## 3.4 负载均衡

数据（负载）被均分到两条链路传输



## 3.5 路由度量

> 度量值，路由开销，metric，cost。到达某目标所花费的开销（代价）的总和，用来衡量路径的优劣



## 3.6 缺省路由（Default route）

> 默认路由，default route，pre 60

```
[Huawei]ip route-static 0.0.0.0 0 12.1.1.2 		#访问任何网段都将数据包交给 12.1.1.2
```

<font color=red>注：缺省路由属于特殊的静态路由</font>

<font color=red>注：PC 的网关其实就是一种缺省路由</font>

<font color=red>缺省路由属于“替补路由”，只有当其他路由不可达时才会使用缺省路由。</font>

缺省路由适用于<font color=red>边缘节点</font>及<font color=red>企业出口</font>



## 3.7 环回接口（Loopback）

> 逻辑接口，模拟网段、PC、服务器、后期用于动态路由选举 Router-ID

配置环回接口：

```
[Huawei]int LoopBack 0
[Huawei-LoopBack0]ip add 220.1.1.2 24
```



## 3.8 DHCP（Dynamic Host Configuration Protocol）

> 动态主机配置协议是一个局域网的网络协议。指的是由服务器控制一段IP地址范围，客户机登录服务器时就可以自动获得服务器分配的IP地址和子网掩码。

<font color=red>BOOTP（Bootstrap Protocol，引导程序协议）是一种引导协议，基于IP/UDP协议，也称自举协议，是DHCP协议的前身。</font>

![](/img/Huawei_route/Huawei_route_base_5.jpg)

开启 DHCP 功能：

```
[Huawei]dhcp enable
```

创建地址池：

```
[Huawei]ip pool add_pool
[Huawei-ip-pool-add_pool]gateway-list 192.168.1.1
[Huawei-ip-pool-add_pool]network 192.168.1.0 mask 24	
[Huawei-ip-pool-add_pool]dns-list 192.168.1.1 8.8.8.8
[Huawei-ip-pool-add_pool]dis this
#
ip pool add_pool
 gateway-list 192.168.1.1
 network 192.168.1.0 mask 255.255.255.0
 dns-list 192.168.1.1 8.8.8.8
#
return
[Huawei-ip-pool-add_pool]q
[Huawei]int e0/0/0	#和用户相连端口
[Huawei-Ethernet0/0/0]dhcp select global #使用本地全局配置的地址池分配 IP 地址
```

客户端 PC6 从本地服务地址池 add_pool 获取 DHCP 地址：

![](/img/Huawei_route/Huawei_route_base_6.jpg)

获取地址包：

![](/img/Huawei_route/Huawei_route_base_7.jpg)

用户发送的第一个 DHCP 请求包源是 0.0.0.0 目标为 255.255.255.255 的广播报文，该报文称为 dhcp discover

![](/img/Huawei_route/Huawei_route_base_8.jpg)



显示 dhcp 分配记录：

```
[Huawei]dis ip pool name add_pool used  
  Pool-name      : add_pool
  Pool-No        : 0
  Lease          : 1 Days 0 Hours 0 Minutes
  Domain-name    : -
  DNS-server0    : 192.168.1.1     
  DNS-server1    : 8.8.8.8         
  NBNS-server0   : -               
  Netbios-type   : -               
  Position       : Local           Status           : Unlocked
  Gateway-0      : 192.168.1.1     
  Mask           : 255.255.255.0
  VPN instance   : --
 -----------------------------------------------------------------------------
         Start           End     Total  Used  Idle(Expired)  Conflict  Disable
 -----------------------------------------------------------------------------
     192.168.1.1   192.168.1.254   253     3        250(0)         0        0
 -----------------------------------------------------------------------------
 
  Network section : 
  --------------------------------------------------------------------------
  Index              IP               MAC      Lease   Status  
  --------------------------------------------------------------------------
    251   192.168.1.252    5489-98c2-3d98       3018   Used       
    252   192.168.1.253    5489-98e4-1b88       3360   Used       
    253   192.168.1.254    5489-9856-795c       3364   Used       
  --------------------------------------------------------------------------
```



## 3.9 RIP(Routing Information Protocol)

> RIP（路由信息协议）协议基于距离矢量算法（DistanceVectorAlgorithms），使用“跳数”(即metric)来衡量到达目标地址的路由距离。这种协议的路由器只关心自己周围的世界，只与自己相邻的路由器交换信息，范围限制在15跳(15度)之内，再远，它就不关心了。RIP应用于OSI网络七层模型的应用层。各厂家定义的管理距离（AD，即优先级）如下：华为定义的优先级是100，思科定义的优先级是120。缺点：古老，收敛速度很慢！

![](/img/Huawei_route/Huawei_route_base_9.jpg)

配置RIP：

![](/img/Huawei_route/Huawei_route_base_10.jpg)

R1:

```
[R1-rip-1]dis this
#
rip 1
 undo summary			#关闭自动汇总
 version 2				#版本2
 network 192.168.1.0	#宣告直连主类网络
 network 12.0.0.0		#宣告直连主类网络
#
return
```

R2:

```
[R2-rip-1]dis this
#
rip 1
 undo summary
 version 2
 network 12.0.0.0
 network 23.0.0.0
 network 172.16.0.0
#
return
```

R3:

```
[R3-rip-1]dis this
#
rip 1
 undo summary
 version 2
 network 23.0.0.0
 network 10.0.0.0
#
return
```



Rip 报文：

![](/img/Huawei_route/Huawei_route_base_11.jpg)

每隔 30s 发送一次报文，目标地址是：224.0.0.9 组播地址，报文里存放的是路由信息

![](/img/Huawei_route/Huawei_route_base_12.jpg)

Rip 只看距离远近，距离是以跳数（Metric，经过路由器的个数）

## 3.10 静默接口（Silent-interface）

<font color=red>PC/客户端 不需要 Rip 报文，可以将路由连接客户端接口配置为抑制接口（silent-interface，静默接口）：</font>

```
[Huawei-rip-1]silent-interface e0/0/0	#将 e0/0/0 口配置为静默接口，Rip的路由更新将不再从该接口发送
```

此时，R1 的路由表：

```
[R1]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 9        Routes : 9        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

     10.10.10.0/24  RIP     100  2           D   12.1.1.2        Ethernet0/0/1			#Rip优先级为100
       12.1.1.0/24  Direct  0    0           D   12.1.1.1        Ethernet0/0/1
       12.1.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/1
       23.1.1.0/24  RIP     100  1           D   12.1.1.2        Ethernet0/0/1
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  RIP     100  1           D   12.1.1.2        Ethernet0/0/1
    192.168.1.0/24  Direct  0    0           D   192.168.1.1     Ethernet0/0/0
    192.168.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0
```



## 3.11 OSPF(Open Shortest Path First)

> 开放式最短路径优先协议是 IETF 定义的一种基于链路状态的内部网关路由协议

ospf 优先级为：10

![](/img/Huawei_route/Huawei_route_base_13.jpg)

Area 0：骨干区域、核心区域

Area 1、2...：常规区域

- 每个区域都维护一个独立的LSDB
- <font color=red>Area 0是骨干区域，其他区域都必须与此区域相连</font>

### 3.11.1 配置 OSPF：

![](/img/Huawei_route/Huawei_route_base_14.jpg)

R1：

```
[Huawei]ospf 1
[Huawei-ospf-1]area 0	
[Huawei-ospf-1-area-0.0.0.0]network 192.168.1.0 ?
  X.X.X.X  OSPF wild card bits

[Huawei-ospf-1-area-0.0.0.0]network 192.168.1.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]network 12.1.1.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]dis this
#
 area 0.0.0.0
  network 192.168.1.0 0.0.0.255
  network 12.1.1.0 0.0.0.255
#
return
```

R2：

```
[Huawei]ospf 1
[Huawei-ospf-1]area 0
[Huawei-ospf-1-area-0.0.0.0]network 12.1.1.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]network 23.1.1.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]network 172.16.1.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]dis this
#
 area 0.0.0.0
  network 12.1.1.0 0.0.0.255
  network 23.1.1.0 0.0.0.255
  network 172.16.1.0 0.0.0.255
#
return
```

R3:

```
[Huawei]ospf 1
[Huawei-ospf-1]area 0	
[Huawei-ospf-1-area-0.0.0.0]network 23.1.1.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]network 10.10.10.0 0.0.0.255
[Huawei-ospf-1-area-0.0.0.0]dis this
#
 area 0.0.0.0
  network 23.1.1.0 0.0.0.255
  network 10.10.10.0 0.0.0.255
#
return
```

配置完 OSPF 的 R1 路由信息：

```
[Huawei]dis ip routing-table
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 9        Routes : 9        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

     10.10.10.0/24  OSPF    10   3           D   12.1.1.2        Ethernet0/0/1
       12.1.1.0/24  Direct  0    0           D   12.1.1.1        Ethernet0/0/1
       12.1.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/1
       23.1.1.0/24  OSPF    10   2           D   12.1.1.2        Ethernet0/0/1
      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
     172.16.1.0/24  OSPF    10   2           D   12.1.1.2        Ethernet0/0/1
    192.168.1.0/24  Direct  0    0           D   192.168.1.1     Ethernet0/0/0
    192.168.1.1/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0
```



### 3.11.2 OSPF 的报文：常见5种

hello：小巧，用来建立和维持邻居关系

![](/img/Huawei_route/Huawei_route_base_15.jpg)

DBD 、LSR、LSU、LSack：用于传递路由信息，刚开始时发送

重置 OSPF 进程来抓取上面4个包：

```
<Huawei>reset ospf process 				#重置（reset）大多需要在\<Huawei> 用户视图 下操作
Warning: The OSPF process will be reset. Continue? [Y/N]:y
```

![](/img/Huawei_route/Huawei_route_base_16.jpg)

### 3.11.3 OSPF 邻居表

```
[R2]dis ospf peer brief 

	 OSPF Process 1 with Router ID 12.1.1.2
		  Peer Statistic Information
 ----------------------------------------------------------------------------
 Area Id          Interface                        Neighbor id      State    
 0.0.0.0          Ethernet0/0/0                    192.168.1.1      Full        
 0.0.0.0          Ethernet0/0/1                    23.1.1.1         Full        
 ----------------------------------------------------------------------------
```

### 3.11.4 OSPF 静默接口（silent-interface）

```
[R1]ospf 1
[R1-ospf-1]silent-interface e0/0/0
[R1-ospf-1]dis this
#
ospf 1
 silent-interface Ethernet0/0/0
 area 0.0.0.0
  network 192.168.1.0 0.0.0.255
  network 12.1.1.0 0.0.0.255
#
return
```



## 3.12 Telnet

![](/img/Huawei_route/Huawei_route_base_17.jpg)

开启 telnet 服务（华为默认开启）：

```
[R4]telnet server enable 
Info: The Telnet server has been enabled.
```

R4 创建用户名和密码：

```
[R4]aaa				#三 A 认证
[R4-aaa]local-user ?
  STRING<1-64>   User name, in form of 'user@domain'. Can use wildcard '*',
                 while displaying and modifying, such as *@isp,user@*,*@*.Can
                 not include invalid character / \ : * ? " < > | @ '

[R4-aaa]local-user telnetuser ?
  access-limit   Set access limit of user(s)
  ftp-directory  Set user(s) FTP directory permitted
  idle-timeout   Set the timeout period for terminal user(s)
  password       Set password 
  privilege      Set admin user(s) level
  service-type   Service types for authorized user(s)
  state          Activate/Block the user(s)
  user-group     User group
[R4-aaa]local-user telnetuser password cipher telnetuser privilege level 3 	#配置用户名密码，权限为3
[R4-aaa]local-user telnetuser service-type telnet  		#指定账户的类型为 telnet
[R4]user-interface 	
[R4]user-interface ?
  INTEGER<0,34-48,50-54>   The first user terminal interface to be configured
  console                  Primary user terminal interface
  current                  The current user terminal interface
  maximum-vty              The maximum number of VTY users, the default value
                           is 5
  vty                      The virtual user terminal interface 

	
[R4]user-interface vty 0 4
[R4-ui-vty0-4]au	
[R4-ui-vty0-4]authe	
[R4-ui-vty0-4]authentication-mode ?
  aaa       AAA authentication
  none      Login without checking
  password  Authentication through the password of a user terminal interface

[R4-ui-vty0-4]authentication-mode aaa
[R4-ui-vty0-4]dis this
#
user-interface con 0
user-interface vty 0 4
 authentication-mode aaa
user-interface vty 16 20
#
return
[R4]user-interface ?
  INTEGER<0,34-48,50-54>   The first user terminal interface to be configured
  console                  Primary user terminal interface
  current                  The current user terminal interface
  maximum-vty              The maximum number of VTY users, the default value
                           is 5
  vty                      The virtual user terminal interface 
	
[R4]user-interface vty 0 4					# 同时允许5个人远程控制
[R4-ui-vty0-4]authentication-mode ?
  aaa       AAA authentication
  none      Login without checking
  password  Authentication through the password of a user terminal interface

[R4-ui-vty0-4]authentication-mode aaa			#认证模式为 aaa 
[R4-ui-vty0-4]dis this
#
user-interface con 0
user-interface vty 0 4
 authentication-mode aaa
user-interface vty 16 20
#
return
```

R1 连接 telnet 服务端：

```
<R1>telnet 34.1.1.2
Trying 34.1.1.2 ...
Press CTRL+K to abort
Connected to 34.1.1.2 ...


Login authentication


Username:admin
Password:
Info: The max number of VTY users is 10, and the number
      of current VTY users on line is 1.
      The current login time is 2020-07-31 10:14:55.
<R4>
```

### 3.12.1 PC 客户端连接 telnet 服务器：

PC 开启 telnet 服务：

控制面板-->程序-->启动或关闭 Windows 功能 --> Telnet Client

![](/img/Huawei_route/Huawei_route_base_19.jpg)

本机 PC 连接 eNSP Cloud 设置：

![](/img/Huawei_route/Huawei_route_base_18.jpg)

本机需要增加静态路由从192.168.1.1 通信：

![](/img/Huawei_route/Huawei_route_base_20.jpg)

显示本地 PC 路由表：

![](/img/Huawei_route/Huawei_route_base_21.jpg)



## 3.13 FTP（File Transfer Protocol)

使用 FTP 实现远程文件传输的同时，还可以保证数据传输的可靠性和高效性。

### 3.13.1 开启 FTP 服务

```
[R4]ftp server enable 
Info: Succeeded in starting the FTP server.
```

![](/img/Huawei_route/Huawei_route_base_22.jpg)

创建 FTP  账户：

```
[R4]aaa
[R4-aaa]local-user ftpuser password cipher ftpuser privilege level 3 ftp-directory flash:
Info: Add a new user.
[R4-aaa]local-user ftpuser service-type ftp				#此账户用于 FTP 服务
```

连接 FTP 服务器：

```
<R1>ftp 34.1.1.2
Trying 34.1.1.2 ...
Press CTRL+K to abort
Connected to 34.1.1.2.
220 FTP service ready.
User(34.1.1.2:(none)):ftpuser
331 Password required for ftpuser.
Enter password:
230 User logged in.

[ftp]dir
200 Port command okay.
150 Opening ASCII mode data connection for *.
drwxrwxrwx   1 noone    nogroup         0 Aug 07  2015 src
drwxrwxrwx   1 noone    nogroup         0 Jul 31 08:39 pmdata
drwxrwxrwx   1 noone    nogroup         0 Jul 31 08:39 dhcp
-rwxrwxrwx   1 noone    nogroup       603 Jul 31 15:09 private-data.txt
drwxrwxrwx   1 noone    nogroup         0 Jul 31 08:54 mplstpoam
-rwxrwxrwx   1 noone    nogroup       507 Jul 31 15:19 ftp.demo
-rwxrwxrwx   1 noone    nogroup       507 Jul 31 15:09 vrpcfg.zip
226 Transfer complete.
```

# 四、构建冗余型企业网络

VLAN、Trunk、STP、VRRP、链路聚合、ACL

## 4.1 VLAN（Virtual Local Area Network）

> 虚拟局域网是将一个物理的局域网在逻辑上划分成多个广播域的技术。通过在交换机上配置 VLAN ，可以实现在同一个 VLAN 内的用户可以进行二层互访，而不同 VLAN 间的用户被二层隔离。这样既能够隔离广播域，又能够提升网络的安全性。

### 4.1.1 配置

![](/img/Huawei_route/Huawei_route_base_23.jpg)

创建 VLAN：

```
[SW1]vlan batch 10 20			#同时创建多个 VLAN ： VLAN 10 、VLAN 20
[SW1]int gi0/0/1				#进入 GE 0/0/1 口
[SW1-GigabitEthernet0/0/1]port link-type access #将接口类型配置为 access
[SW1-GigabitEthernet0/0/1]port default vlan 10 	#将接口划分到 VLAN 10 里
[SW1-GigabitEthernet0/0/1]dis this
#
interface GigabitEthernet0/0/1
 port link-type access
 port default vlan 10
#
return

#查看 VLAN 配置
[SW1]dis vlan
The total number of vlans is : 3
--------------------------------------------------------------------------------
U: Up;         D: Down;         TG: Tagged;         UT: Untagged;
MP: Vlan-mapping;               ST: Vlan-stacking;
#: ProtocolTransparent-vlan;    *: Management-vlan;
--------------------------------------------------------------------------------

VID  Type    Ports                                                          
--------------------------------------------------------------------------------
1    common  UT:GE0/0/4(U)      GE0/0/5(D)      GE0/0/6(D)      GE0/0/7(D)      
                GE0/0/8(D)      GE0/0/9(D)      GE0/0/10(D)     GE0/0/11(D)     
                GE0/0/12(D)     GE0/0/13(D)     GE0/0/14(D)     GE0/0/15(D)     
                GE0/0/16(D)     GE0/0/17(D)     GE0/0/18(D)     GE0/0/19(D)     
                GE0/0/20(D)     GE0/0/21(D)     GE0/0/22(D)     GE0/0/23(D)     
                GE0/0/24(D)                                                     

10   common  UT:GE0/0/1(U)      GE0/0/2(U)                                      

20   common  UT:GE0/0/3(U)                                                      


VID  Status  Property      MAC-LRN Statistics Description      
--------------------------------------------------------------------------------

1    enable  default       enable  disable    VLAN 0001                         
10   enable  default       enable  disable    VLAN 0010                         
20   enable  default       enable  disable    VLAN 0020  
```

<font color=red>注： VLAN 1 属于默认 VLAN，默认情况下所有接口都在 VLAN 1 下。VLAN 隔离广播的同时，也会隔离 arp，从而导致单播无法通信。如果想让不同 VLAN 单播可以通信，还需要三层设备（路由器、三层交换机）做路由。</font>

注：默认情况下，交换机的一个接口只能从属一个 VLAN，只允许该 VLAN 的数据通过。

## 4.2 Trunk（干道，主干链路）

> 通常用于交换机和交换机之间，通过一个接口传输多个 VLAN 的数据包。

### 4.2.1 配置

![](/img/Huawei_route/Huawei_route_base_24.jpg)

将交换机与交换机相连接的端口类型改为 Trunk 类型

```
[SW1]int gi0/0/4
[SW1-GigabitEthernet0/0/4]port link-type trunk 		#将接口类型配置为 trunk
[SW1-GigabitEthernet0/0/4]port trunk allow-pass vlan all 	#允许所有的 VLAN 通过
[SW1-GigabitEthernet0/0/4]dis this
#
interface GigabitEthernet0/0/4
 port link-type trunk
 port trunk allow-pass vlan 2 to 4094
#
return


[SW1-GigabitEthernet0/0/4]port link-type ?
  access        Access port			#接 PC 端口
  dot1q-tunnel  QinQ port
  hybrid        Hybrid port			#混合口 即可以接 PC 也可以接交换机（华为交换机的默认接口）
  trunk         Trunk port			#接交换机
  
#查看接口类型
[SW1]dis port vlan
Port                    Link Type    PVID  Trunk VLAN List
-------------------------------------------------------------------------------
GigabitEthernet0/0/1    access       10    -                                   
GigabitEthernet0/0/2    access       10    -                                   
GigabitEthernet0/0/3    access       20    -                                   
GigabitEthernet0/0/4    trunk        1     1
GigabitEthernet0/0/5    hybrid       1     -                                   
GigabitEthernet0/0/6    hybrid       1     -                                             GigabitEthernet0/0/7    hybrid       1     -       
```

PC 到交换机 access 口的包：

![](/img/Huawei_route/Huawei_route_base_25.jpg)

<font color=red>注：PC 不识 VLAN 标记，不识 tag，只有通过交换机的 trunk 接口发出的报文才具备 VLAN 的标记（802.1q tag），见下图</font>

交换机 trunk 到 trunk 交换机包：

![](/img/Huawei_route/Huawei_route_base_26.jpg)

### 4.2.2 PVID(本征 VLAN，native vlan)

该 VLAN 的报文经过 trunk 接口时不打标记。默认情况下本征 VLAN 是 VLAN 1。

```
[SW1]dis port vlan 
Port                    Link Type    PVID  Trunk VLAN List
-------------------------------------------------------------------------------
GigabitEthernet0/0/1    access       10    -                                   
GigabitEthernet0/0/2    access       10    -                                   
GigabitEthernet0/0/3    access       20    -                                   
GigabitEthernet0/0/4    trunk        1     1-4094
GigabitEthernet0/0/5    hybrid       1     -                                   
GigabitEthernet0/0/6    hybrid       1     -                                   
GigabitEthernet0/0/7    hybrid       1     -     
```

例：

![](/img/Huawei_route/Huawei_route_base_27.jpg)

PC8 和 PC9 默认属于 VLAN 1，它们通信经过交换机 trunk 时，是不打 tag 的。

![](/img/Huawei_route/Huawei_route_base_28.jpg)

#### 修改 PVID

```
#两台相连交换机 trunk 都需要修改
[SW1]int gi 0/0/4	
[SW1-GigabitEthernet0/0/4]port trunk pvid vlan 20

[SW2]int gi 0/0/1	
[SW2-GigabitEthernet0/0/1]port trunk pvid vlan 20
```

当 PVID 修改为 VLAN 20 后，VLAN 20 下的 PC 通信经过交换机 trunk 时不打 tag。



## 4.3 VLAN 间路由

<font color=red>注：不同的 VLAN 之间互相通信必须要有三层设备（路由器、多层交换机）做中转。</font>



### 4.3.1 多层交换机--SVI（switch virtual interface，常用）

![](/img/Huawei_route/Huawei_route_base_29.jpg)

#### SVI 配置

```
[Huawei]int Vlanif 10
[Huawei-Vlanif10]ip address 192.168.10.1 24		#给 VLAN 10 配置 ip 地址 10.1 作为 VLAN 10 的用户网关
[Huawei-Vlanif10]dis this
#
interface Vlanif10
 ip address 192.168.10.1 255.255.255.0
#
return

[Huawei]int Vlanif 20
[Huawei-Vlanif20]ip address 192.168.20.1 24		#给 VLAN 20 配置 ip 地址 20.1 作为 VLAN 20 的用户网关
#调试
[Huawei]dis ip interface brief 
*down: administratively down
^down: standby
(l): loopback
(s): spoofing
The number of interface that is UP in Physical is 4
The number of interface that is DOWN in Physical is 1
The number of interface that is UP in Protocol is 3
The number of interface that is DOWN in Protocol is 2

Interface                         IP Address/Mask      Physical   Protocol  
MEth0/0/1                         unassigned           down       down      
NULL0                             unassigned           up         up(s)     
Vlanif1                           unassigned           up         down      
Vlanif10                          192.168.10.1/24      up         up        
Vlanif20                          192.168.20.1/24      up         up
#显示路由表
[Huawei]dis ip routing-table 
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 6        Routes : 6        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
   192.168.10.0/24  Direct  0    0           D   192.168.10.1    Vlanif10
   192.168.10.1/32  Direct  0    0           D   127.0.0.1       Vlanif10
   192.168.20.0/24  Direct  0    0           D   192.168.20.1    Vlanif20
   192.168.20.1/32  Direct  0    0           D   127.0.0.1       Vlanif20
```

注：

VLAN 间路由：通过三层设备路由，使得不同 VLAN 间可以互相通信，但是仅仅允许单播通信。不同 VLAN 之间广播帧依然被隔离既没有失去 VLAN 原本的意义。



### 4.3.2 路由器--单臂路由

![](/img/Huawei_route/Huawei_route_base_30.jpg)

#### 路由配置

```
[Router]int e0/0/0.?					#可生成的逻辑接口
  <1-4096>   
[Router]int e0/0/0.10
[Router-Ethernet0/0/0.10]ip address 192.168.10.254 24	#为逻辑接口 10 配置10网段的网关
[Router-Ethernet0/0/0.10]dot1q termination vid 10		#绑定 VLAN 10
[Router-Ethernet0/0/0.10]arp broadcast enable 			#使子接口有ARP广播功能
[Router-Ethernet0/0/0.10]dis this
#
interface Ethernet0/0/0.10
 dot1q termination vid 10
 ip address 192.168.10.254 255.255.255.0
 arp broadcast enable
#
return

[Router]int e0/0/0.20
[Router-Ethernet0/0/0.20]ip address 192.168.20.254 24
[Router-Ethernet0/0/0.20]dot1q termination vid 20	
[Router-Ethernet0/0/0.20]arp broadcast enable 
[Router-Ethernet0/0/0.20]dis this
#
interface Ethernet0/0/0.20
 dot1q termination vid 20
 ip address 192.168.20.254 255.255.255.0
 arp broadcast enable
#
return

[Router]dis ip routing-table
Route Flags: R - relay, D - download to fib
------------------------------------------------------------------------------
Routing Tables: Public
         Destinations : 8        Routes : 8        

Destination/Mask    Proto   Pre  Cost      Flags NextHop         Interface

      127.0.0.0/8   Direct  0    0           D   127.0.0.1       InLoopBack0
      127.0.0.1/32  Direct  0    0           D   127.0.0.1       InLoopBack0
   192.168.10.0/24  Direct  0    0           D   192.168.10.254  Ethernet0/0/0.10
   192.168.10.2/32  Direct  0    0           D   192.168.10.2    Ethernet0/0/0.10
 192.168.10.254/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0.10
   192.168.20.0/24  Direct  0    0           D   192.168.20.254  Ethernet0/0/0.20
   192.168.20.2/32  Direct  0    0           D   192.168.20.2    Ethernet0/0/0.20
 192.168.20.254/32  Direct  0    0           D   127.0.0.1       Ethernet0/0/0.20
```



## 4.4 ACL（Access Control List）

> 访问控制列表

- 基本ACL（2000-2999）：只能匹配源 IP 地址
- 高级ACL（3000-3999）：可以匹配源IP、目标IP、源端口、目标端口等三层和四层的字段

### 4.4.1 配置

![](/img/Huawei_route/Huawei_route_base_31.jpg)

**例1：**

禁止 Client 1 192.168.10.1 访问172.16.10.0 网段：

配置 R2 路由：

```
#配置 ACL 规则
[R2]acl ?
  INTEGER<2000-2999>  Basic access-list(add to current using rules)			#基本 ACL
  INTEGER<3000-3999>  Advanced access-list(add to current using rules)		#高级 ACL
  INTEGER<4000-4999>  Specify a L2 acl group
  ipv6                ACL IPv6 
  name                Specify a named ACL
  number              Specify a numbered ACL
[R2-acl-basic-2000]rule ?
  INTEGER<0-4294967294>  ID of ACL rule
  deny                   Specify matched packet deny		#拒绝
  permit                 Specify matched packet permit		#允许
[R2-acl-basic-2000]rule deny source 192.168.10.1 ?
  IP_ADDR<X.X.X.X>  Wildcard of source
  0                 Wildcard bits : 0.0.0.0 ( a host )
[R2-acl-basic-2000]rule deny source 192.168.10.1 0		#拒绝源地址为192.168.10.1的任何数据包，0代表主机，不代表某个网段
[R2-acl-basic-2000]dis this
[V200R003C00]
#
acl number 2000  
 rule 5 deny source 192.168.10.1 0 		# 5 为自动生成的执行编号，第二条则为10（每条+5）
#
return
#在接口的入方向调用 ACL 规则
[R2]int gi0/0/0
[R2-GigabitEthernet0/0/0]traffic-filter inbound acl 2000 #在 R2 接口的入（inbound）方向调用 acl
[R2-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 12.1.1.2 255.255.255.0 
 traffic-filter inbound acl 2000
#
return
```

**例2：**

![](/img/Huawei_route/Huawei_route_base_31.jpg)

在 R2 上配置*高级 ACL* 拒绝 Client 1 和 PC2 ping Server1（拒绝 ICMP 协议数据包），但是允许其 HTTP 访问 Server 1。

```
#配置 ACL 规则
[R2]acl 3000
[R2-acl-adv-3000]rule deny icmp source 192.168.10.0 0.0.0.255 destination 172.16
.10.2 0				#拒绝 192.168.10.0 网段 ping（ICMP） Server 1(172.16
.10.2 0代表本机)
[R2-acl-adv-3000]dis this
[V200R003C00]
#
acl number 3000  
 rule 5 deny icmp source 192.168.10.0 0.0.0.255 destination 172.16.10.2 0 
#
return

#接口入调用 ACL 规则
[R2]int gi0/0/0
[R2-GigabitEthernet0/0/0]traffic-filter inbound acl 3000
```



**例3：**

![](/img/Huawei_route/Huawei_route_base_31.jpg)

拒绝源地址 192.168.10.2 telnet（port 23） 访问12.1.1.2

```
#配置 ACL 规则
[R2]acl 3001
[R2-acl-adv-3001]rule deny tcp source 192.168.10.2 0 destination 12.1.1.2 0 dest
ination-port eq 23
[R2-acl-adv-3001]dis this
[V200R003C00]
#
acl number 3001  
 rule 5 deny tcp source 192.168.10.2 0 destination 12.1.1.2 0 destination-port e
q telnet 
#
return

#接口入调用 ACL 规则
[R2]int gi0/0/0
[R2-GigabitEthernet0/0/0]traffic-filter inbound acl 3001
```



<font color=red>**注：**</font>

1. **一个接口的同一个方向，只能调用一个 ACL**
2. **一个 ACL 里面可以有多个 rule 规则，从上往下依次执行**
3. **数据包一旦被某 rule 匹配，就不再继续向下匹配**
4. **默认隐含放过所有（华为的 ACL 用来拒绝数据包时）**



# 五、NAT（Network Address Translation）

> 网络地址转换主要用于实现位于内部网络的主机访问外部网络的功能。当局域网内的主机需要访问外部网络时，通过 NAT 技术可以将其私网地址转换为公网地址，并且多个私网地址可以共用一个公网地址，这样既能保证网络互通，有节省公网地址。

- NAT 一般部署在连接内网和外网的网关设备上

**私有地址：任何人都能使用**

A 10.0.0.0/8

B 172.16.0.0-172.31.255.255

C 192.168.0.0/16



## 5.1 Static NAT 配置 
> 一对一（一一映射），一个私网地址对应一个公网地址，外网的用户可以访问内网的主机

![](/img/Huawei_route/Huawei_route_base_32.jpg)

基础配置：

1. 配置好 IP 地址
2. 出口配置缺省路由`[OUT]ip route-static 0.0.0.0 0 12.1.1.6`
3. 在企业路由WAN端口（GE 0/0/1）配置 Static NAT

```
[OUT]int gi 0/0/1
[OUT-GigabitEthernet0/0/1]nat static ?
  enable    Enable function
  global    Specify global information of NAT		#全局公网地址
  protocol  Specify protocol
[OUT-GigabitEthernet0/0/1]nat static global 12.1.1.2 ?
  inside  Specify inside information of NAT  		#内部网络
[OUT-GigabitEthernet0/0/1]nat static global 12.1.1.2 inside 192.168.1.2 #将私网地址 1.2 和 12.1.1.2 做一对一映射
[OUT-GigabitEthernet0/0/1]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/1
 description TO_wan
 ip address 12.1.1.1 255.255.255.248 
 nat static global 12.1.1.2 inside 192.168.1.2 netmask 255.255.255.255
#
return
```

用内网 PC 192.168.1.2 Ping 百度（9.9.9.9），从企业路由WAN端口（GE 0/0/1）出抓包：

![](/img/Huawei_route/Huawei_route_base_33.jpg)

发现此时内网地址 192.168.1.2 已经被转换为 12.1.1.2 

调试：查看 NAT 转换过程：

```
[OUT]dis nat session protocol icmp
  NAT Session Table Information:

     Protocol          : ICMP(1)
     SrcAddr   Vpn     : 192.168.1.2                                    
     DestAddr  Vpn     : 9.9.9.9                                        
     Type Code IcmpId  : 0   8   49311
     NAT-Info
       New SrcAddr     : 12.1.1.2       
       New DestAddr    : ----
       New IcmpId      : ----

```

## 5.2 NAPT (Network Address Port Translation)
由于NAT实现是私有IP和NAT的公共IP之间的转换，那么，私有网中同时与公共网进行通信的主机数量就受到NAT的公共IP地址数量的限制。为了克服 这种限制，NAT被进一步扩展到在进行IP地址转换的同时进行Port的转换，这就是网络地址端口转换NAPT（Network Address Port Translation）技术。
    NAPT与NAT的区别在于，NAPT不仅转换IP包中的IP地址，还对IP包中TCP和UDP的Port进行转换。这使得多台私有网主机利用1个NAT公共IP就可以同时和公共网进行通信。（NAPT多了对TCP和UDP的端口号的转换）

    私有网主机192.168.1.2要访问公共网中的 Http服务器166.111.80.200。首先，要建立TCP连接，假设分配的TCP Port是1010，发送了1个IP包（Des=166.111.80.200:80,Src=192.168.1.2:1010）,当IP包经过NAT 网关时，NAT会将IP包的源IP转换为NAT的公共IP，同时将源Port转换为NAT动态分配的1个Port。然后，转发到公共网，此时IP包 （Des=166.111.80.200：80，Src=202.204.65.2:2010）已经不含任何私有网IP和Port的信息。由于IP包的源 IP和Port已经被转换成NAT的公共IP和Port，响应的IP包 （Des=202.204.65.2:,Src=2010166.111.80.200:80）将被发送到NAT。这时NAT会将IP包的目的IP转换成 私有网主机的IP，同时将目的Port转换为私有网主机的Port，然后将IP包 （Des=192.168.1.2:1010，Src=166.111.80.200:80）转发到私网。对于通信双方而言，这种IP地址和Port的转 换是完全透明的。

## 5.3 NAT -- Easy IP

Easy IP 方式的实现原理与上节介绍的地址池NAPT 转换原理类似，可以算是NAPT的一种特例，不同的是Easy IP 方式可以实现自动根据路由器上WAN 接口的公网IP 地址实现与私网IP 地址之间的映射（无需创建公网地址池）。
Easy IP 主要应用于将路由器WAN 接口IP 地址作为要被映射的公网IP 地址的情形，特别适合小型局域网接入Internet 的情况。这里的小型局域网主要指中小型网吧、小型办公室等环境，一般具有以下特点：内部主机较少、出接口通过拨号方式获得临时（或固定）公网IP 地址以供内部主机访问Internet。图6-3 所示为Easy IP 方式的实现原理，具体过程如下。
![](/img/Huawei_route/Huawei_route_base_57.jpg)
① 假设私网中的Host A 主机要访问公网的Server，首先要向Router 发送一个请求报文（即Outbound 方向），此时报文中的源地址是10.1.1.100，端口号1540。
② Router 在收到请求报文后自动利用公网侧WAN 接口临时或者固定的“公网IP地址:端口号”（162.10.2.8:5480），建立与内网侧报文“源IP 地址:源端口号”间的Easy IP转换表项（也包括正、反两个方向），并依据正向Easy IP 表项的查找结果将报文转换后向公网侧发送。此时，转换后的报文源地址和源端口号由原来的（10.1.1.100:1540）转换成了（162.10.2.8:5480）。
③ Server 在收到请求报文后需要向Router 发送响应报文（即Inbound 方向），此时只须将收到的请求报文中的源IP 地址、源端口号和目的IP 地址、目的端口号对调即可，即此时的响应报文中的目的IP 地址、目的端口号为162.10.2.8:5480。
④ Router 在收到公网侧Server 的回应报文后，根据其“目的IP 地址:目的端口号”查找反向Easy IP 表项，并依据查找结果将报文转换后向内网侧发送。即转换后的报文中的目的IP 地址为10.1.1.100，目的端口号为1540，与Host A 发送请求报文中的源IP地址和源端口完全一样。
如果私网中的Host B 也要访问公网，则它所利用的公网IP 地址与Host A 一样，都是路由器WAN 口的公网IP 地址，但转换时所用的端口号一定要与Host A 转换时所用的端口不一样。


> 允许多个私网地址转换成一个公网 IP ，常用

![](/img/Huawei_route/Huawei_route_base_32.jpg)

基础配置：

1. 首先配置 ACL 匹配内网私网地址段

   ```
   [OUT]acl 2000
   [OUT-acl-basic-2000]rule permit source 192.168.1.0 0.0.0.255
   [OUT-acl-basic-2000]dis this
   [V200R003C00]
   #
   acl number 2000  
    rule 5 permit source 192.168.1.0 0.0.0.255 
   #
   return
   [OUT-acl-basic-2000]
   ```

<font color=red>注：ACL 用来做匹配范围时，没有默认隐含允许所有的规则</font>

2. ```
   [OUT-GigabitEthernet0/0/1]nat ?
     outbound  Specify net address translation		#Easy IP，配置 NAT 地址池的转换策略，可以选择匹配ACL模式或不匹配ACL两种模式
     server    Specify NAT server			#可以将某服务器的某端口映射出去（安全）
     static    Specify static NAT			#静态 NAT，一对一
    [OUT-GigabitEthernet0/0/1]nat outbound ?
     INTEGER<2000-3999>  Apply basic or advanced ACL		#配置的 ACL 编号
   [OUT-GigabitEthernet0/0/1]nat outbound 2000 
   ```

   原理：内网私网地址出包时转换成公网接口 WAN（GE 0/0/1）当前的 IP 地址

### 5.3.1 NAT -- Server

> 可以将某服务器的某端口映射出去（安全）

配置：

![](/img/Huawei_route/Huawei_route_base_34.jpg)

```
[OUT-GigabitEthernet0/0/1]nat server protocol tcp global 12.1.1.3 www inside 192
.168.1.254 www 		#将服务器的80端口映射成公网地址的80端口
[OUT-GigabitEthernet0/0/1]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/1
 description TO_wan
 ip address 12.1.1.1 255.255.255.248 
 nat server protocol tcp global 12.1.1.3 www inside 192.168.1.254 www
#
return
```



# 六、广域网封装协议

> 广域网中经常会使用串行链路来提供远距离的数据传输，高级数据链路控制 HDLC （  High-Level Data Link Control  ）和点对点协议 PPP （  Point to Point Protocol ）是两种典型的串口封装协议。 



## 6.1 PPP（  Point to Point Protocol ）

> PPP （点对点协议）协议是一种点到点链路层协议，主要用于在全双工的同异步链路上进行点到点的数据传输

### 6.1.1 PPP 配置

![](/img/Huawei_route/Huawei_route_base_35.jpg)

```
[R1]int Serial 4/0/0
[R1-Serial4/0/0]ip add 12.1.1.1 24
[R1-Serial4/0/0]dis this
[V200R003C00]
#
interface Serial4/0/0
 link-protocol ppp
 ip address 12.1.1.1 255.255.255.0 
#
return
[R1]ping 12.1.1.2
  PING 12.1.1.2: 56  data bytes, press CTRL_C to break
    Reply from 12.1.1.2: bytes=56 Sequence=1 ttl=255 time=20 ms
    Reply from 12.1.1.2: bytes=56 Sequence=2 ttl=255 time=30 ms

```

抓包，PPP的封装方式：

![](/img/Huawei_route/Huawei_route_base_36.jpg)

 <font color=red>注：PPP 可以对链路做认证</font>：PAP 认证、CHAP 认证



#### PAP认证配置（密码明文传输，两次握手）

![](/img/Huawei_route/Huawei_route_base_37.jpg)

```
#认证端（服务端）
[R2]aaa
[R2-aaa]local-user HCNP password cipher hcnp123		#创建本地用户名和密码
Info: Add a new user.
[R2-aaa]local-user hcnp service-type ppp			#创建的账户用作 ppp 认证
[R2]int Serial 4/0/0 
[R2-Serial4/0/0]ppp authentication-mode pap			#进入接口开启认证

#被认证端（客户端）
[R1]int Serial 4/0/0
[R1-Serial4/0/0]ppp pap local-user HCNP password ?
  cipher  Display the current password with cipher text 
  simple  Display the current password with plain text 
[R1-Serial4/0/0]ppp pap local-user HCNP password simple hcnp123 #发送 pap 用户名密码认证
```



#### CHAP 认证配置（三次握手，加密传输）

![](/img/Huawei_route/Huawei_route_base_37.jpg)

```
#认证端（服务端）
[R2-Serial4/0/0]ppp authentication-mode chap 
[R2-Serial4/0/0]dis this
[V200R003C00]
#
interface Serial4/0/0
 link-protocol ppp
 ppp authentication-mode chap 
 ip address 12.1.1.2 255.255.255.0 
#
return

#被认证端（客户端）
[R1-Serial4/0/0]ppp chap user HCNP
[R1-Serial4/0/0]ppp chap password simple hcnp123
```



## 6.2 HDLC（High-Level Data Link Control ）

> HDLC（高级数据链路控制） 是高级数据链路控制协议，是一种数据链路层的协议。HDLC 是一个 ISO 标准的面向位的数据链路协议，其在同步串行数据链路上封装数据，最常用于点对点链接

### 6.2.1 HDLC 配置

![](/img/Huawei_route/Huawei_route_base_35.jpg)

```
[R1]int s4/0/0
[R1-Serial4/0/0]dis this
[V200R003C00]
#
interface Serial4/0/0
 link-protocol ppp					#华为、H3C 串口默认封装方式为 PPP，思科为 HDLC
 ip address 12.1.1.1 255.255.255.0 
#
return
[R1-Serial4/0/0]link-protocol ?
  fr    Select FR as line protocol
  hdlc  Enable HDLC protocol				
  lapb  LAPB(X.25 level 2 protocol)
  ppp   Point-to-Point protocol 
  sdlc  SDLC(Synchronous Data Line Control) protocol 
  x25   X.25 protocol
[R1-Serial4/0/0]link-protocol hdlc 		#手动修改成 HDLC
Warning: The encapsulation protocol of the link will be changed. Continue? [Y/N]
:y
[R1-Serial4/0/0]dis this
[V200R003C00]
#
interface Serial4/0/0
 link-protocol hdlc
 ip address 12.1.1.1 255.255.255.0 
#
return
```

<font color=red>注：两个相连端口封装方式需要一致，不然无法通信，所以路由 R2 需要做相同配置。</font>

通信抓包：

![](/img/Huawei_route/Huawei_route_base_35.jpg)



## 6.3 FR（Frame-relay）

> 帧中继

### 6.3.1 FR 配置

![](/img/Huawei_route/Huawei_route_base_39.jpg)

```
[R1-Serial4/0/0]dis this
[V200R003C00]
#
interface Serial4/0/0
 link-protocol fr
 ip address 12.1.1.1 255.255.255.0 
#
return

#查看映射
[R1]dis fr map-info 
Map Statistics for interface Serial4/0/0 (DTE)
  DLCI = 102, IP INARP 12.1.1.2, Serial4/0/0
    create time = 2020/08/07 09:25:46, status = ACTIVE
    encapsulation = ietf, vlink = 2, broadcast
```

抓包：

![](/img/Huawei_route_base_40.jpg)



# 七、链路聚合

>链路捆绑、端口聚合、eth-channel
>
>随着网络规模不断扩大，用户对骨干链路的带宽和可靠性提出了越来越高的要求。在传统技术中，常用更换高速率的接口板或更换支持高速率接口板的设备的方式来增加带宽，但这种方案需要付出高额的费用，而且不够灵活。
>采用链路聚合技术可以在不进行硬件升级的条件下，通过将多个物理接口捆绑为一个逻辑接口，来达到增加链路带宽的目的。在实现增大带宽目的的同时，链路聚合采用备份链路的机制，可以有效的提高设备之间链路的可靠性。



### 配置

![](/img/Huawei_route_base_41.jpg)

```
#现状态
[SW1]dis stp brief
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        ROOT  FORWARDING      NONE		#FORWARDING：转发
   0    GigabitEthernet0/0/2        ALTE  DISCARDING      NONE		#DISCARDING：丢弃
   0    GigabitEthernet0/0/3        ALTE  DISCARDING      NONE
   0    GigabitEthernet0/0/4        DESI  FORWARDING      NONE
   
#配置链路聚合
[SW1]int Eth-Trunk 1				#创建逻辑捆绑接口组 1
[SW1-GigabitEthernet0/0/1]eth-trunk 1	#进入接口，将接口加入 Eth-Trunk 1
[SW1-GigabitEthernet0/0/2]eth-trunk 1
[SW1-GigabitEthernet0/0/3]eth-trunk 1
#查看配置
[SW1]dis eth-trunk 
Eth-Trunk1's state information is:
WorkingMode: NORMAL         Hash arithmetic: According to SIP-XOR-DIP         
Least Active-linknumber: 1  Max Bandwidth-affected-linknumber: 8  #最多能捆绑8个链路      
Operate status: up         Number Of Up Port In Trunk: 3                     
--------------------------------------------------------------------------------
PortName                      Status      Weight 
GigabitEthernet0/0/1          Up          1      
GigabitEthernet0/0/2          Up          1      
GigabitEthernet0/0/3          Up          1   

[SW1]dis stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/4        DESI  FORWARDING      NONE
   0    Eth-Trunk1                  ROOT  FORWARDING      NONE
#交换机 R2 同样配置
```



# 八、VRRP

> Virtual Router Redundancy [rɪˈdʌndənsi] Protocol，**虚拟网关冗余协议**
>
> 三层网关冗余技术，对用户的网关做冗余

## 8.1 配置

![](/img/Huawei_route/Huawei_route_base_42.jpg)

#### 基础配置

- 核心的 IP 地址
- 接入层交换机无需配置
- 用户网关配置成 192.168.10.1

#### VRRP 配置

```
#核心1
[core1]int gi 0/0/0			#下联用户接口配置
[core1-GigabitEthernet0/0/0]vrrp ?
  arp       Gratuitous arp
  un-check  Uncheck VRRP packet TTL value
  vrid      Specify virtual router identifier
[core1-GigabitEthernet0/0/0]vrrp vrid ?			#指定虚拟路由器标识符（Specify virtual router identifier）
  INTEGER<1-255>  Virtual router identifier
[core1-GigabitEthernet0/0/0]vrrp vrid 1 ?
  authentication-mode  Specify password and authentication mode
  preempt-mode         Specify preempt mode
  priority             Specify priority
  timer                Specify timer
  track                Specify the track configuration
  version-3            Specify the device to support V3 for VRRP
  virtual-ip           Specify virtual IP address
#创建虚拟组1，并指定虚拟 IP 地址，该虚拟地址作为用户网关
[core1-GigabitEthernet0/0/0]vrrp vrid 1 virtual-ip 192.168.10.1 
[core1-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 192.168.10.253 255.255.255.0 
 vrrp vrid 1 virtual-ip 192.168.10.1
#
return
#核心2同样配置
[core2-GigabitEthernet0/0/0]vrrp vrid 1 priority 105	#将优先级改为105作为主路由器，优先级默认是100。数字越大越优先
[core2-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 192.168.10.254 255.255.255.0 
 vrrp vrid 1 virtual-ip 192.168.10.1
 vrrp vrid 1 priority 105
#
return
#查看 VRRP 状态
[core1]dis vrrp brief 
Total:1     Master:0     Backup:1     Non-active:0      
VRID  State        Interface                Type     Virtual IP     
----------------------------------------------------------------
1     Backup       GE0/0/0                  Normal   192.168.10.1 

[core2]dis vrrp brief 
Total:1     Master:1     Backup:0     Non-active:0      
VRID  State        Interface                Type     Virtual IP     
----------------------------------------------------------------
1     Master       GE0/0/0                  Normal   192.168.10.1 、

#查看 VRRP 详细信息
[core2]dis vrrp
  GigabitEthernet0/0/0 | Virtual Router 1
    State : Master
    Virtual IP : 192.168.10.1
    Master IP : 192.168.10.254
    PriorityRun : 105
    PriorityConfig : 105
    MasterPriority : 105
    Preempt : YES   Delay Time : 0 s
    TimerRun : 1 s
    TimerConfig : 1 s
    Auth type : NONE				#认证信息
    Virtual MAC : 0000-5e00-0101
    Check TTL : YES
    Config type : normal-vrrp
    Backup-forward : disabled
    Create time : 2020-08-10 12:19:35 UTC-08:00
    Last change time : 2020-08-10 12:24:04 UTC-08:00
    
#配置如上联接口断开，降低 master 优先级
##配置跟踪上联接口 GE 0/0/1 状态，当发现 GE 0/0/1 口 down 时，将自动降低此路由的优先级-10，以让出 master 的位置
[core2-GigabitEthernet0/0/0]vrrp vrid 1 track interface  GigabitEthernet 0/0/1
[core2-GigabitEthernet0/0/0]dis this
[V200R003C00]
#
interface GigabitEthernet0/0/0
 ip address 192.168.10.254 255.255.255.0 
 vrrp vrid 1 virtual-ip 192.168.10.1
 vrrp vrid 1 priority 105
 vrrp vrid 1 track（跟踪） interface GigabitEthernet0/0/1
#
return
[core2]dis vrrp
  GigabitEthernet0/0/0 | Virtual Router 1
    State : Backup
    Virtual IP : 192.168.10.1
    Master IP : 192.168.10.253
    PriorityRun : 95		<---降低后的优先级
    PriorityConfig : 105
    MasterPriority : 100
    Preempt : YES   Delay Time : 0 s
    TimerRun : 1 s
    TimerConfig : 1 s
    Auth type : NONE
    Virtual MAC : 0000-5e00-0101
    Check TTL : YES
    Config type : normal-vrrp
    Backup-forward : disabled
    Track IF : GigabitEthernet0/0/1   Priority reduced : 10	<---降低优先级10
    IF state : DOWN		<---上联端口断开
    Create time : 2020-08-10 12:19:35 UTC-08:00
    Last change time : 2020-08-10 13:17:30 UTC-08:00

#开启简单的密码认证，两个核心都需要配置（可选配置）
[core2-GigabitEthernet0/0/0]vrrp vrid 1 authentication-mode simple plain 123  #配置简单明文密码123
```

#### 工作原理

抓取核心1 GE 0/0/0 口报文

![](/img/Huawei_route/Huawei_route_base_43.jpg)

核心路由器会每隔一段时间（约1S）发送一个特定的 VRRP 报文。如果一段时间没有收到对方发来的 VRRP 报文，就认定对方 master 设备出现故障。此时 backup 会总动切换成 master


# 九、STP （Spanning Tree Protocol）

>为了提高网络可靠性，交换网络中通常会使用冗余链路。然而，冗余链路会给交换网络带来环路风险，并导致广播风暴以及 MAC 地址表不稳定等问题，进而会影响到用户的通信质量。生成树协议 STP 可以在提高可靠性的同时又能避免环路带来的各种问题。

<font color=red>**作用：防止交换环路**</font>
![](/img/Huawei_route/Huawei_route_base_44.jpg)

华为交换机默认开机启动 STP

## 9.1 实验

将交换机 STP 关闭，抓包：
![](/img/Huawei_route/Huawei_route_base_44.jpg)
```
[SW1]stp disable 
```
PC ping ip 1.1.1.255 ，为广播地址，其目标 MAC 地址为 ff:ff:ff:ff:ff:ff（交换机靠 MAC 转发）

![](/img/Huawei_route/Huawei_route_base_45.jpg)

### 9.1.1 引发各种问题

#### 广播风暴

![](/img/Huawei_route/Huawei_route_base_46.jpg)

#### MAC 地址表震荡

![](/imgHuawei_route//Huawei_route_base_47.jpg)


## 9.2 工作原理

STP通过阻塞端口来消除环路，并能够实现链路备份的目的。
查看阻塞端口：
```
[SW1]dis stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        DESI  DISCARDING      NONE
   0    GigabitEthernet0/0/2        DESI  DISCARDING      NONE
   0    GigabitEthernet0/0/3        ROOT  FORWARDING      NONE

```

## 9.3 STP 算法

先选出不被阻塞的接口，剩下的接口都会被阻塞。

### 9.3.1 过程

1. 整个网络（广播域）先选出根桥（根交换机），先比较优先级（默认32768），再比较 MAC 地址，越小越优先。根桥上面的端口都是**指定端口**

![](/img/Huawei_route/Huawei_route_base_48.jpg)

显示：

```
<SW1>dis stp
-------[CIST Global Info][Mode MSTP]-------
CIST Bridge         :32768.4c1f-cc6b-2dc9     #桥ID（优先级+MAC）
Config Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
Active Times        :Hello 2s MaxAge 20s FwDly 15s MaxHop 20
CIST Root/ERPC      :32768.4c1f-cc5f-2395 / 20000   #显示根桥信息
CIST RegRoot/IRPC   :32768.4c1f-cc6b-2dc9 / 0
CIST RootPortId     :128.3
BPDU-Protection     :Disabled
TC or TCN received  :5
TC count per hello  :0
STP Converge Mode   :Normal 
Time since last TC  :0 days 0h:17m:53s
Number of TC        :6
Last TC occurred    :GigabitEthernet0/0/1
----[Port1(GigabitEthernet0/0/1)][FORWARDING]----
 Port Protocol       :Enabled
 Port Role           :Designated Port
 Port Priority       :128
 Port Cost(Dot1T )   :Config=auto / Active=20000
 Designated Bridge/Port   :32768.4c1f-cc6b-2dc9 / 128.1
 Port Edged          :Config=default / Active=disabled
 Point-to-point      :Config=auto / Active=true
 Transit Limit       :147 packets/hello-time
 Protection Type     :None
  ---- More ----
```


2. 非根桥上面选举根端口（根端口有且仅有一个）到达根桥最近的端口当选为根端口
非根交换机在选举根端口时分别依据该端口的根路劲开销、对端BID（桥ID）、对端 PID 和本端 PID
![](/img/Huawei_route_base_49.jpg)

3. 每段链路必须选举一个指定端口（且只有一个），桥 ID （优先级+MAC）较小的交换机上面的端口当选为指定端口
![](/img/Huawei_route_base_50.jpg)

4. 剩下的端口全部被阻塞
![](/img/Huawei_route_base_51.jpg)
查看状态：
```
<SW2>dis stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        ALTE  DISCARDING      NONE    #阻塞
   0    GigabitEthernet0/0/2        ROOT  FORWARDING      NONE
```


## 9.4 更改交换机的优先级
```
[SW2]stp priority ?
  INTEGER<0-61440>  Bridge priority, in steps of 4096   #需要4096的倍数

[SW2]stp priority 0   #将优先级改为0
#或者直接设置为根桥
[SW2]stp root ?
  primary    Primary root switch    #设置为根桥
  secondary  Secondary root switch  #设备备份根桥
```
![](/img/Huawei_route/Huawei_route_base_51.jpg)

---------------->
![](/img/Huawei_route/Huawei_route_base_52.jpg)


## 9.5 BPDU 报文
BPDU 包含桥 ID、路径开销、端口 ID、计数器等参数

![](/img/Huawei_route/Huawei_route_base_53.jpg)



## 9.6 端口重启过程

down ---> listening ---> learning ---> forwarding
此过程大概 30s
listening：监听端口
learning：检测该端口是否需要阻塞，如要阻塞，则阻塞端口，防止环路

设置边缘端口：建议将接 PC 的接口配置为边缘端口（减少接口的收敛时间）

```
[SW1-GigabitEthernet0/0/1]stp edged-port enable
```

## 9.7 STP 根保护

一旦使能根保护功能的指定端口收到优先级更低的 BPUD 时，端口状态将进入 Discarding 状态，不再转发报文。经过一段时间（通常为两倍的 Forward Delay）,如果端口一直没有再收到优先级较高（数值低）的 BPUD，端口会自动恢复到正常的 Forwarding 状态。
<font color=red>注：该指令只能在指定端口配置才会生效</font>

![](/img/Huawei_route/Huawei_route_base_54.jpg)

在根桥上的两个端口设置根保护，在 SW1 上接个优先级为 0 的交换机
```
[SW2-GigabitEthernet0/0/1]stp root-protection 
[SW2-GigabitEthernet0/0/2]stp root-protection
#接上优先级为0的交换机，发现根桥设置根保护的端口为阻塞状态
[SW2]dis stp brief 
 MSTID  Port                        Role  STP State     Protection
   0    GigabitEthernet0/0/1        DESI  DISCARDING      ROOT
   0    GigabitEthernet0/0/2        DESI  DISCARDING      ROOT
```

## 9.8 STP BPDU 防护

作用：保护根桥（全局），开启 BPDU 保护后，如果从<font color=red>边缘端口</font>收到 STP 报文，交换机会自动将该接口 shutdown，从而确保根桥不会被抢占，同时确保不会出现环路。

![](/img/Huawei_route/Huawei_route_base_54.jpg)
```
#首先在 SW1 上的 GE 0/0/1 口配置边缘接口
[SW1-GigabitEthernet0/0/1]stp edged-port enable 
在交换机 SW1 上配置 STP BPUD 防护
[SW1]stp bpdu-protection
#30S 后自动 up ，自动恢复机制
[SW1]error-down auto-recovery cause bpdu-protection interval 30
```

## 9.9 RSTP

Rapaid STP  快速的生成树协议，STP 升级版
设置:
```
[SW2]stp mode rstp    #将 STP 模式切换为 RSTP
```


# 十、IPV6

>Internal protocol version 6

ipv4 32bit 地址个数为 2^32
ipv6 128bit 地址个数为 2^128

ipv6 实例：
**2001:0DB8:0000:0000:0000:0000:0346:8D58**
ipv6 由8个字段组成，每个字段占16个 bit
上面地址简写：2001:DB8:0:0:0:0:346:8D58 ---> 2001:DB8::346:8D58

## 10.1 特殊 ipv6 地址

1. ::1 本地环回地址
2. :: 相当于 ipv4 0.0.0.0
3. FF 开头 组播 v6 地址 例如：FF::5 类似于224.0.0.5


## 10.2 ipv6 静态路由配置

![](/img/Huawei_route/Huawei_route_base_56.jpg)

```
[R1]ipv6      #全局使能 ipv6 功能
[R1-GigabitEthernet0/0/1]ipv6 enable
[R1-GigabitEthernet0/0/1]ipv6 address 12::1 ?
  INTEGER<1-128>  IPv6 prefix length <1-128>
  link-local      Use link-local address
[R1-GigabitEthernet0/0/1]ipv6 address 12::1 64  #设置 ipv6 地址和前缀长度64
#查看路由器接口信息
[R1]dis ipv6 int brief 
*down: administratively down
(l): loopback
(s): spoofing
Interface                    Physical              Protocol   
GigabitEthernet0/0/0         up                    up         
[IPv6 Address] 2001::1
GigabitEthernet0/0/1         up                    up         
[IPv6 Address] 12::1
#查看路由表
[R1]dis ipv6 routing-table
#配置静态路由
[R1]ipv6 route-static 2002:: 64 12::2
#在 R2 配置缺省路由
[R2]ipv6 route-static :: 0 12::1
#查看 R1 路由表
[R1]dis ipv6 routing-table 
Routing Table : Public
	Destinations : 7	Routes : 7

 Destination  : ::1                             PrefixLength : 128
 NextHop      : ::1                             Preference   : 0
 Cost         : 0                               Protocol     : Direct
 RelayNextHop : ::                              TunnelID     : 0x0
 Interface    : InLoopBack0                     Flags        : D

 Destination  : 2002::                          PrefixLength : 64
 NextHop      : 12::2                           Preference   : 60
 Cost         : 0                               Protocol     : Static
 RelayNextHop : ::                              TunnelID     : 0x0
 Interface    : GigabitEthernet0/0/1            Flags        : RD
```
ipv6 报文：
![](/img/Huawei_route/Huawei_route_base_54.jpg)


## 10.3 IPV6 地址分类

#### 单播
#### 组播
#### 任意播（取消广播概念）

ipv6 无状态自动配置：PC 会通过发送特定类型的 icmp 报文请求路由器接口前缀，结合自己的 MAC 地址自动生成全球独一无二的 ipv6 地址

ipv6 中以 FE80:: 开头的地址都属于本地链路地址（Link-local），只有在本地链路有效。启用了 ipv6 功能的接口都会自动生成相应的 Link-local 地址。












