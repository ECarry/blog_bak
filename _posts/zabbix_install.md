---
layout: post
title: 在 CentOS 7 下 搭建 Zabbix 5.0 LTS 日志
tags: [Zabbix]
index_img: https://th.wallhaven.cc/small/4d/4dveeg.jpg
banner_img: https://w.wallhaven.cc/full/4d/wallhaven-4dveeg.jpg
categories: [Zabbix]
date: 2020-08-27 10:45:40
---

# 一、 YUM 安装-服务端

## 1.1 关闭防火墙和 selinux ，重启

```bash
[root@localhost ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
[root@localhost ~]# systemctl disable firewalld
[root@localhost ~]# init 6
```

## 1.2 安装 zabbix rpm 源，替换成 阿里云的 zabbix 源

[阿里云源](https://mirrors.aliyun.com)

```bash
[root@localhost ~]# rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
[root@localhost ~]# sed -i 's@http://repo.zabbix.com@https://mirrors.aliyun.com/zabbix@' /etc/yum.repos.d/zabbix.repo
[root@localhost ~]# yum clean all
```

## 1.3 安装 zabbix server 和 agent

```bash
[root@localhost ~]# yum install zabbix-server-mysql zabbix-agent -y
```

## 1.4 安装 Software Collections，便于后续安装高版本的 php，默认 yum 安装的 php 版本为 5.4 过低

```bash
[root@localhost ~]# yum install centos-release-scl -y
```

启用 zabbix 前端源，修改 vi /etc/yum.repos.d/zabbix.repo，将 **[zabbix-frontend]** 下的 **enabled=1**

## 1.5 安装 zabbix 前端和相关环境

```bash
[root@localhost ~]# yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
```

## 1.6 yum 安装 centos7 默认的 mariadb 数据库并配置数据库
```bash
[root@localhost ~]# yum install mariadb-server -y

#启动数据库，并配置开机自动启动
[root@localhost ~]# systemctl enable --now mariadb

#使用以下命令初始化 mariadb 并配置 root 密码
[root@localhost ~]# mysql_secure_installation 

NOTE: RUNNING ALL PARTS OF THIS SCRIPT IS RECOMMENDED FOR ALL MariaDB
      SERVERS IN PRODUCTION USE!  PLEASE READ EACH STEP CAREFULLY!

In order to log into MariaDB to secure it, we'll need the current
password for the root user.  If you've just installed MariaDB, and
you haven't set the root password yet, the password will be blank,
so you should just press enter here.

Enter current password for root (enter for none): root123.

#使用 root 用户进入 mysql，并建立 zabbix 数据库，注意数据库编码
[root@localhost ~]# mysql -u root -p 
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 11
Server version: 5.5.65-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> 
create database zabbix character set utf8 collate utf8_bin;
create user zabbix@localhost identified by 'password';
grant all privileges on zabbix.* to zabbix@localhost;
quit;

#使用以下命令导入 zabbix 数据库，zabbix 数据库用户为 zabbix，密码为 password
[root@localhost ~]# zcat /usr/share/doc/zabbix-server-mysql*/create.sql.gz | mysql -uzabbix -p zabbix

#修改 zabbix server 配置文件vi /etc/zabbix/zabbix_server.conf 里的数据库密码
DBPassword=password

#修改 zabbix 的 php 配置文件vi /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf 里的时区，改成 Asia/Shanghai
php_value[date.timezone] = Asia/Shanghai

#启动相关服务，并配置开机自动启动
[root@localhost ~]# systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
[root@localhost ~]# systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm
```

## 1.7 使用浏览器访问 http://ip/zabbix 即可访问 zabbix 的 web 页面

### 默认登录用户名 `Admin` 密码 `zabbix`

# 二、 安装客户端

## 2.1 关闭防火墙和 selinux ，重启

```bash
[root@localhost ~]# sed -i 's/SELINUX=enforcing/SELINUX=disabled/' /etc/selinux/config
[root@localhost ~]# systemctl disable firewalld
[root@localhost ~]# init 6
```

## 2.2 安装 zabbix rpm 源，替换成 阿里云的 zabbix 源

[阿里云源](https://mirrors.aliyun.com)

```bash
[root@localhost ~]# rpm -Uvh https://mirrors.aliyun.com/zabbix/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
[root@localhost ~]# sed -i 's@http://repo.zabbix.com@https://mirrors.aliyun.com/zabbix@' /etc/yum.repos.d/zabbix.repo
[root@localhost ~]# yum clean all
```

## 2.3 安装 zabbix agent

```bash
[root@localhost ~]# yum install zabbix-agent -y
```

## 2.4 修改 zabbix-agent 配置文件

配置文件位置：`/etc/zabbix/zabbix_agentd.conf`

```
[root@localhost ~]# vi /etc/zabbix/zabbix_agentd.conf
PidFile=/run/zabbix/zabbix_agentd.pid
LogFile=/var/log/zabbix/zabbix_agentd.log
LogFileSize=0
Server=192.168.1.1            #服务端地址
ListenPort=10050              #监听端口
ListenIP=192.168.1.1          #客户端本机IP
ServerActive=192.168.1.1      #服务端地址
Hostname=Ubuntu               #主机名，需要跟后面网页配置的主机名相同
Include=/etc/zabbix/zabbix_agentd.d/*.conf
```

## 2.5 启动或重启 zabbix-agent 服务

`systemctl restart zabbix-agent`

### 查看 zabbix-agent 状态

`systemctl status zabbix-agent`

### 设置开机自动启动

`systemctl enable zabbix-agent`