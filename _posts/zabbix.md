---
layout: post
title: Zabbix 
tags: [Zabbix]
index_img: /img/zabbix/zabbix_logo.jpg
banner_img: https://w.wallhaven.cc/full/5w/wallhaven-5w26x1.jpg
categories: [Zabbix]
date: 2020-7-8 17:14:35
---

# About 

Zabbix 是一个基于 WEB 界面的提供**分布式系统监视**以及**网络监视**功能的企业级的开源解决方案。zabbix 能监视各种网络参数，保证服务器系统的安全运营；并提供灵活的通知机制以让系统管理员快速定位、解决存在的各种问题。

<!-- more -->

{% note success %}
服务端及客户端安装配置文档：[在 CentOS 7 下 搭建 Zabbix 5.0 LTS 日志](https://ecarry.cc/2020/08/27/zabbix_install/)
{% endnote %}

# 一、 Zabbix 监控架构

## 1.1 Zabbix 监控架构

 ![Zabbix 监控架构](/img/zabbix/zabbix_1.png)

- 由两台 Zabbix server 构成监控端，一主一备
- Zabbix db 所有配置信息以及 Zabbix 采集到的数据都被存储在数据库中
- 两台代理服务器，可以代替 Zabbix server 采集性能和可用性数据。Zabbix proxy 在 Zabbix 的部署是可选部分；但是 proxy 的部署可以很好的分担单个 Zabbix server 的负载
- Zabbix agents 部署在被监控目标上，用于主动监控本地资源和应用程序，并将收集的数据发送给 Zabbix server

## 1.2 Zabbix 特性

### 1.2.1 优点

- 开源，无软件成本投入

- Server 对设**备性能要求低**

- 支持设备多，自带多种监控模板

- 支持分布式集中管理，有自动发现功能，可以**实现自动化监控**

- 开放式接口，扩展性强，插件编写容易

- 当监控的 item 比较多、服务器队列比较大时可以采用主动状态，被监控客户端主动从 server 端去下载需要监控的 item，然后取数据上传到 server 端。 这种方式对服务器的负载比较小

- Api 的支持，方便与其他系统结合

### 1.2.2 缺点

- ​需在被监控主机上安装 agent,所有数据都存在数据库里, 产生的数据据很大,瓶颈主要在数据库

- ​项目批量修改不方便   

- ​入门容易，能实现基础的监控，但是深层次需求需要非常熟悉 Zabbix 并进行大量的二次定制开发难度较大

- ​系统级别报警设置相对比较多，如果不筛选的话报警邮件会很多；并且自定义的项目报警需要自己设置，过程比较繁琐

- ​少数据汇总功能，如无法查看一组服务器平均值，需进行二次开发

## 1.3 Zabbix 监控对象

 ![](/img/zabbix/zabbix_2.png)

## 1.4 Zabbix 监控方式

### 1.4.1 被动模式

服务端向客户端请求获取配置的各个监控相关的数据，客户端收到请求后，获取数据后返回给服务端

### 1.4.2 主动模式

客户端向服务端请求与自己相关的监控配置，主动的将服务端需要的监控数据收集发给服务端

# 二、 Zabbix 服务架构

## 2.1 Zabbix 组件

 ![](/img/zabbix/zabbix_3.png)

