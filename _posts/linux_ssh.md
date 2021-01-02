---
layout: post
title: Linux ssh
tags: [Linux]
index_img: https://th.wallhaven.cc/small/4x/4x92p3.jpg
banner_img: https://w.wallhaven.cc/full/4x/wallhaven-4x92p3.jpg
categories: [Linux]
date: 2020-09-21 17:15:14
---

# 一、 SSH

## 1.1 ssh

ssh：Secure Shell，应用层协议，默认端口 22/tcp

### 特点

- 通信过程及认证过程是加密的，主机认证
- 用户认证过程加密
- 数据传输过程加密

### 认证过程

- 基于口令认证
- 基于密钥认证

## 1.2 openSSH

### 1.2.1 用于 linux，C/S 架构：

- 服务端：sshd，配置文件 /etc/ssh/sshd_config
- 客户端：ssh，配置文件 /etc/ssh/ssh_config
- ssh-keygen：秘钥生成器
- ssh-copy-id：将公钥传输至远程服务器
- scp：跨主机安全复制工具

#### ssh 使用

- ssh USERNAME@HOST：用 USERNAME 登录 HOST
- ssh USERNAME@HOST 'COMMAND'：不登录执行远程主机命令，并打印在屏幕

#### scp 使用

- scp SRC DEST
  - -r
  - -a
- scp /path/to/copy USERNAME@HOST:/path/to/save：将本地文件复制到远程主机上
- scp USERNAME@HOST:/path/to/copy /path/to/save：将远程主机上的文件复制到本机上

#### ssh-keygen 使用

- ssh-keygen -t rsa
  - ~/.ssh/id_rsa：密钥
  - \~/.ssh/id_rsa.pub：公钥，可复制到远程主机 `~/.ssh/authorized_keys` 下
  - -f /path/to/KEY_FILE：保存密钥到指定目录
  - -p ' '：指定加密密钥的密码，或为空

#### ssh-copy-id

- ssh-copy-id
  - -i ~/.ssh/id_rsa.pub USERNAME@HOST：将公钥拷贝到远程主机

# 二、 配置文件

