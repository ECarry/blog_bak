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

## 2.4 修改监控项存储的值

可以修改监控项所存储的值，方便后续针对性的处理

![](/img/zabbix_host/zabbix_host_15.jpg)

# 三、 触发器 trigger

当采集的值定义完了以后，就可以来定义触发器了。触发器的定义是：**界定某特定的 item 采集到的数据的非合理区间或非合理状态。通常为逻辑表达式。**逻辑表达式（阈值）：通常用于定义数据的不合理区间。

评定采样数值是否为合理区间的比较稳妥的方法是——根据最后N次的平均值来判定结果；这个最后N次通常有两种定义方式：

- 最近N分钟所得结果的平均值
- 最近N次所得结果的平均值

## 3.1 触发器表达式

`{<server>:<key>.<function>(<parameter>)}<operator><constant>`

- server：主机名称
- key：主机上关系的相应监控项的key
- function：评估采集到的数据是否在合理范围内时所使用的函数，其评估过程可以根据采取的数据、当前时间及其它因素进行
- 目前触发器所支持的函数有avg、count、change、date、dayofweek、delta、diff、iregexp、last、max、min、nodata、now、sum等
- parameter：函数参数；大多数数值函数可以接受秒数为其参数，而如果在数值参数之前使用“#”做为前缀，则表示为最近几次的取值，如sum(300)表示300秒内所有取值之和，而sum(#10)则表示最近10次取值之和
- 此外，avg、count、last、min和max还支持使用第二个参数，用于完 成时间限定；例如，max(1h,7d)将返回一周之前的最大值

## 3.2 添加一个触发器

添加一个触发器，检测网络入流量是否过高，过高告警

![](/img/zabbix_host/zabbix_host_16.jpg)

问题表达式：

`{web1.web.com:net.if.in[eth0,bytes].last(#1,5)}>=10485760`

主机为 web1，监控网卡 eth0 的数据量，单位为 bytes，获取网络流入的最后一个值，如流入主机的网络大于 10MB，则触发告警

恢复表达式：

`{web1.web.com:net.if.in[eth0,bytes].last(#3,5)}<=9437184`

获取网络流入的最后三个值的平均数，如流入主机的网络小于 9MB，则告警消除

表达式可以直接点击右侧的添加，然后定义自己所需的内容，即可自动生成：

![](/img/zabbix_host/zabbix_host_17.jpg)

模拟报警，拷贝数据，数据量为 11 MB/s：

![](/img/zabbix_host/zabbix_host_18.jpg)

首页收到报警：

![](/img/zabbix_host/zabbix_host_19.jpg)

已知问题，可以将问题忽略，比如拷贝数据造成的网络拥堵：

![](/img/zabbix_host/zabbix_host_20.jpg)

## 3.3 触发器的依赖关系

- 触发器彼此之间可能会存在依赖关系的，一旦某一个触发器被触发了，那么依赖这个触发器的其余触发器都不需要再报警。

- 多台主机是通过交换机的网络连接线来实现被监控的。如果交换机出了故障，我们的主机自然也无法继续被监控，如果此时，所有主机统统报警……想想也是一件很可怕的事情。要解决这样的问题，就是定义触发器之间的依赖关系，当交换机挂掉，只有自己报警就可以了，其余的主机就不需要在报警了。**这样，也更易于我们判断真正故障所在。**

- **注意**：目前 zabbix 不能够直接定义主机间的依赖关系，其依赖关系仅能通过触发器来定义。

- 定义一个依赖关系：打开任意一个触发器，上面就有依赖关系，我们进行定义即可：
![](/img/zabbix_host/zabbix_host_21.jpg)

触发器可以有多级依赖关系：

![](/img/zabbix_host/zabbix_host_22.png)

# 四、 定义动作 action

## 4.1 动作简介

- 需要去基于一个对应的事件为条件来指明该做什么事，一般就是执行远程命令或者发警报。
- 有一个**告警升级**的机制，所以，当发现问题的时候，一般是先执行一个**远程操作命令**，如果能够解决问题，就会发一个恢复操作的讯息给接收人，如果问题依然存在，则会执行**发警报**的操作，一般默认的警报接收人是当前系统中有的 zabbix 用户，所以当有人需要收到警报操作的话，我们则需要把它加入我们的定义之中。
- 每一个用户也应该有一个接收告警信息的方式，即媒介，就像我们接收短信是需要有手机号的一样。
- 每一个监控主机，能够传播告警信息的媒介有很多种，就算我们的每一种大的媒介，能够定义出来的实施媒介也有很多种。而对于一个媒介来说，每一个用户都有一个统一的或者不同的接收告警信息的端点，我们称之为目标地或者目的地。

## 4.2 定义一个动作

定义一个动作，监控器监控主机上的 redis 服务（监听 redis 6379 端口）时候存活，如果监控 redis 服务 down，尝试设定好的动作（如重启 redis 服务），当 redis 服务起来后，将信息用媒介发送给管理员，服务要是起不来，则发送告警给管理员。

### 4.2.1 创建 redis 服务

在 web1 服务器上安装 redis，并启动

```shell
yum install redis -y
# 修改 /etc/redis.conf,不做任何认证操作
bind=0.0.0.0
# 启动服务
systemctl start redis
# 查看端口是否已启用
netstat -antlp|grep 6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      15272/redis-server 
```

### 4.2.2 创建监控项

![](/img/zabbix_host/zabbix_host_23.png)

监听 web1 主机上的 6379端口，存活返回1，否则返回0

查看最新数据，显示 redis server up

![](/img/zabbix_host/zabbix_host_28.png)

### 4.2.3 创建触发器

![](/img/zabbix_host/zabbix_host_24.png)

当监控项返回最后一个数值为0时，触发严重告警

在 web1 上手动关闭 redis 服务，触发告警

![](/img/zabbix_host/zabbix_host_29.png)

### 4.2.4 创建动作

![](/img/zabbix_host/zabbix_host_25.png)

触发动作后，执行第一条命令：

![](/img/zabbix_host/zabbix_host_30.png)

重启 redis `sudo systemctl restart redis`

<font color=red>注意：要执行远程 sudo 命令，需要修改 web1 主机上的配置</font>

- 修改 sudo 配置文件使 zabbix 用户能够临时拥有管理员权限
- 修改 zabbix 配置文件使其允许接收远程命令

```shell
visudo          # 相当于“vim /etc/sudoers”
## Allow root to run any commands anywhere
root    ALL=(ALL)   ALL
zabbix    ALL=(ALL)   NOPASSWD: ALL     # 添加的一行，表示不需要输入密码

vim /etc/zabbix/zabbix_agentd.conf
EnableRemoteCommands=1          # 允许接收远程命令
LogRemoteCommands=1             # 把接收的远程命令记入日志

systemctl restart zabbix-agent.service
```

