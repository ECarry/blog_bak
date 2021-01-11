---
layout: post
title: DNS 服务器
tags: [linux]
index_img: /img/linux_dns/index.jpg
banner_img: /img/linux_dns/index.jpg
categories: [linux]
date: 2021-01-05 09:34:31
---

# 一、 DNS？

## 1.1 DNS

域名系统（英文：**D**omain **N**ame **S**ystem，缩写：DNS）是互联网的一项服务。它作为将 **域名** 和 **IP 地址** 相互映射的一个分布式数据库，能够使人更方便地访问互联网。DNS使用TCP和UDP端口53。当前，对于每一级域名长度的限制是63个字符，域名总长度则不能超过253个字符。

## 1.2 DNS 的分层结构

![](/img/linux_dns/dns_1.png)

# 二、 BIND 9

## 2.1 使用 bind9 搭建 DNS 服务

{% note info %}
测试环境 CentOS/RedHat7
DNS Server:192.168.8.201
web1:
web2:
{% endnote %}

### 2.1.1 关闭防火墙及 selinux

```shell
systemctl stop firewalld
systemctl disable firewalld
setenforce 0
```

### 2.1.2 安装必要工具

```shell
yum install wget net-tools telnet tree nmap sysstat lrzsz dos2unix bind-utils -y
```

### 2.1.3 DNS 服务初始化

```shell
yum install -y bind
```

### 2.1.4 修改配置文件

配置文件位于 `/etc/named.conf`

```conf
options {
        listen-on port 53 { 192.168.8.201; };   #修改为本机的 ip 地址
       #listen-on-v6 port 53 { ::1; }; 删除 IPV6
        directory       "/var/named";
        dump-file       "/var/named/data/cache_dump.db";
        statistics-file "/var/named/data/named_stats.txt";
        memstatistics-file "/var/named/data/named_mem_stats.txt";
        recursing-file  "/var/named/data/named.recursing";
        secroots-file   "/var/named/data/named.secroots";
        allow-query     { any; };   #修改为 any，让所有主机能够被查询

        #添加上级 DNS 地址,也就是本地网关地址 192.168.8.1
        forwarders      { 192.168.8.1; };

        # DNS 采用递归算法查询（还有一种迭代查询）
        recursion yes;

        dnssec-enable yes;
        dnssec-validation yes;

        /* Path to ISC DLV key */
        bindkeys-file "/etc/named.root.key";

        managed-keys-directory "/var/named/dynamic";

        pid-file "/run/named/named.pid";
        session-keyfile "/run/named/session.key";
};

logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};

zone "." IN {
        type hint;
        file "named.ca";
};

include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";

```

### 2.1.5 检测配置文件是否有语法错误

`named-checkconf`

## 2.2 配置区域文件

 `/etc/named.rfc1912.zones `

 ```conf
zone "web.com" IN {
        type master;
        file "web.com.zone";    #web.com是域名下的 DNS 解析数据文件，放在 /var/named/web.com.zone
        allow-update { 192.168.2.119; };
};
 ```

 ## 2.3 编辑区域数据文件

`/var/named/web.com.zone `

```conf
$ORIGIN web.com.
$TTL 600    ; 10 minutes
@       IN SOA  dns.web.com. dnsadmin.web.com. (
                2021011101 ; serial ,需要保持一致
                10800      ; refresh (3 hours)
                900        ; retry (15 minutes)
                604800     ; expire (1 week)
                86400      ; minimum (1 day)
                )
            NS   dns.web.com.
$TTL 60 ; 1 minute
dns                A    192.168.2.119
ops-web            A    192.168.2.119
web1               A    192.168.2.176
web2               A    192.168.2.181
```

在此检查配置文件 `named-checkconf`

## 2.4 启动服务

```shell
# 启动服务
[root@ops-web ~]# systemctl start named
# 检查服务是否正常，查看53端口
[root@ops-web ~]# netstat -antulp |grep 53
tcp        0      0 192.168.2.119:53        0.0.0.0:*               LISTEN      13871/named         
tcp        0      0 127.0.0.1:953           0.0.0.0:*               LISTEN      13871/named         
tcp        0      0 192.168.2.119:45221     199.7.83.42:53          TIME_WAIT   -                   
tcp        0      0 192.168.2.119:22        192.168.2.224:56953     ESTABLISHED 8706/sshd: root@pts 
tcp6       0      0 ::1:953                 :::*                    LISTEN      13871/named         
udp        0      0 192.168.2.119:53        0.0.0.0:*                           13871/named 
# 使用 dig 检测
[root@ops-web ~]# dig -t A web1.web.com @192.168.2.119 +short
192.168.2.176
[root@ops-web ~]# dig -t A web2.web.com @192.168.2.119 +short
192.168.2.181

```

## 2.5 配置客户端

将两个 web 服务端网络配置文件 DNS1 改为 DNS 服务器 IP（192.168.2.119）

`sed -i 's/DNS1="192.168.2.1"/DNS1="192.168.2.119"/' /etc/sysconfig/network-scripts/ifcfg-eth0 `

重启网络服务 `systemctl restart network`

检测 DNS 解析是否正常

```shell
[root@web1 ~]# ping ops-web
PING ops-web.web.com (192.168.2.119) 56(84) bytes of data.
64 bytes from 192.168.2.119 (192.168.2.119): icmp_seq=1 ttl=64 time=0.585 ms
```