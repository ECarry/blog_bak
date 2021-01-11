---
layout: post
title: Ansible 自动化运维工具
tags: [Linux, Ansible]
index_img: /img/ansible/ansible_img.jpg
banner_img: /img/ansible/ansible_banner.jpg
categories: [Ansible]
date: 2021-01-08 09:42:47
---

# About Ansible
---------------
Ansible is an IT automation tool. It can configure systems, deploy software, and orchestrate more advanced IT tasks such as continuous deployments or zero downtime rolling updates.

<!-- more -->

## 特性

- 模块化：调用特定模块，完成特定任务
- Paramiko(python 对 ssh 的实现)，PyYAML，Jinja2（模块语言）三个关键模块
- 支持自定义模块，可使用任何编程语言写模块
- 基于 python 语言实现
- 部署简单，基于 python 和 SSH，agentless，无需代理不依赖 PKI（无需 ssl）
- 安全，基于 OpenSSH
- 幂等性：一个任务执行1遍和执行n遍效果一样，不因重复执行带来意外情况
- 支持 playbook 编排任务，YAML 格式，编排任务，支持丰富的数据结构
- 较强大的多层解决方案 role

## 架构

### 组成

![](/img/ansible/ansible_1.png)

组合 INVENTORY、API、MODULES、PLUGINS 的绿框，可以理解为是 ansible 命令工具，其为核心执行工具

- INVENTORY：Ansible 管理主机的清单 `/etc/anaible/hosts`
- MODULES：Ansible 执行命令的功能模块，多数为内置核心模块，也可自定义
- PLUGINS：模块功能的补充，如连接类型插件、循环插件、变量插件、过滤插件等，该功能不常用
- API：供第三方程序调用的应用程序编程接口

### Ansible 命令执行来源

- USER 普通用户，即SYSTEM ADMINISTRATOR
- PLAYBOOKS：任务剧本（任务集），编排定义 Ansible 任务集的配置文件，由Ansible顺序依次执行，通常是 JSON格式的 YML 文件
- CMDB（配置管理数据库） API 调用
- PUBLIC/PRIVATE CLOUD API 调用
- USER-> Ansible Playbook -> Ansibile

### 注意事项

- 执行 ansible 的主机一般称为主控端，中控，master 或堡垒机
- 主控端 Python 版本需要2.6或以上
- 被控端 Python 版本小于2.4，需要安装 python-simplejson
- 被控端如开启 SELinux 需要安装 libselinux-python
- windows 不能做为主控端

# 一、安装

测试环境：

- 管理主机：ops@192.168.2.119
- 被管理主机：client@192.168.2.176
- 被管理主机：client@192.168.2.181

## 1.1 使用 epel 源 安装

### 1.1.1 先安装 epel 源

```shell
# 适用于 RHEL/CentOS 7
yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
```

### 1.1.2 安装 ansible

```shell
yum install -y ansible
# 安装成功后验证
[root@ops ~]# ansible --version
ansible 2.9.16
  config file = /etc/ansible/ansible.cfg        # 默认配置文件目录
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr  2 2020, 13:16:51) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]
```

## 1.2 使用 pip 安装


# 二、Ansible 工具及演示

## 2.1 工具

- `/usr/bin/ansible` 主程序，临时命令执行工具
- `/usr/bin/ansible-doc` 查看配置文档，模块功能查看工具
- `/usr/bin/ansible-galaxy` 下载/上传优秀代码或Roles模块的官网平台
- `/usr/bin/ansible-playbook` 定制自动化任务，编排剧本工具
- `/usr/bin/ansible-pull` 远程执行命令的工具
- `/usr/bin/ansible-vault` 文件加密工具
- `/usr/bin/ansible-console` 基于Console界面与用户交互的执行工具

### 利用ansible实现管理的主要方式：

- **Ad-Hoc** 即利用 ansible 命令，主要用于临时命令使用场景（单条的执行命令）
- **Ansible-playbook** 主要用于长期规划好的，大型项目的场景，需要有前期的规划过程

### ansible-doc 

用来显示模块帮助

格式

```shell
ansible-doc [options] [module...]

-l, --list          #列出可用模块
-s, --snippet       #显示指定模块的playbook片段

#显示有 3387 个模块
[root@ops ~]# ansible-doc -l|wc -l
3387
# 显示某个模块详细信息
[root@ops ~]# ansible-doc ping
# 显示某个模块简介
[root@ops ~]# ansible-doc -s ping
- name: Try to connect to host, verify a usable python and return `pong' on success
  ping:
      data:                  # Data to return for the `ping' return value. If this parameter is set to `crash', the module will
                               cause an exception.

```
### ansible 

此工具通过ssh协议，实现对远程主机的配置管理、应用部署、任务执行等功能

格式

```shell
ansible <host-pattern> [-m module_name] [-a args]

--version           #显示版本
-m module           #指定模块，默认为command
-v                  #详细过程 –vv  -vvv更详细
--list-hosts        #显示主机列表，可简写 --list
-k, --ask-pass      #提示输入ssh连接密码，默认Key验证    
-C, --check         #检查，并不执行
-T, --timeout=TIMEOUT #执行命令的超时时间，默认10s
-u, --user=REMOTE_USER #执行远程执行的用户
-b, --become        #代替旧版的sudo 切换
--become-user=USERNAME  #指定sudo的runas用户，默认为root
-K, --ask-become-pass  #提示输入sudo时的口令
```

**ansible 的 Host-pattern**

用于匹配被控制的主机列表

ALL: 表示所有 Inventory 中的所有主机

**示例：**

ping 所有主机
```shell 
ansible all –m ping
```

ping webserver 和 dbserver 组里的主机
```shell 
ansible "webserver:&dbserver" –m ping
```

ping 在 webserver 组但不在 dbserver 组里的主机（使用单引号）
```shell  
ansible 'webserver:!dbserver' –m ping
```

#### ansible 命令执行过程

1. 加载自己的配置文件 默认/etc/ansible/ansible.cfg

2. 加载自己对应的模块文件，如：command

3. 通过ansible将模块或命令生成对应的临时py文件，并将该文件传输至远程服务器的对应执行用户$HOME/.ansible/tmp/ansible-tmp-数字/XXX.PY文件

4. 给文件 +x 执行

5. 执行并返回结果

6. 删除临时py文件，退出

#### ansible 的执行状态

```shell
[colors]
#highlight = white
#verbose = blue
#warn = bright purple
#error = red
#debug = dark gray
#deprecate = purple
#skip = cyan
#unreachable = red
#ok = green             #绿色：执行成功并且不需要做改变的操作
#changed = yellow       #黄色：执行成功并且对目标主机做变更
#diff_add = green       
#diff_remove = red      
#diff_lines = cyan
#红色：执行失败
```

## 2.2 管理节点与被管理节点建立 ssh 信任关系

### 2.2.1 管理节点（ansible）中创建密钥对

```shell
ssh-ketgen -t rsa
[root@ops ~]# ls .ssh/
id_rsa  id_rsa.pub
```

### 2.2.2 将本地公钥传输到被管理节点

每个管理节点都需要传输，使用 `ssh-copy-id root@ip` 传输公钥：

```shell
[root@ops ~]# ssh-copy-id root@192.168.2.176
/usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/root/.ssh/id_rsa.pub"
The authenticity of host '192.168.2.176 (192.168.2.176)' can't be established.
ECDSA key fingerprint is SHA256:aiIpXJC+jZccDcmkShcEPuUUpOXpLioVmKWDy9wOpI8.
ECDSA key fingerprint is MD5:cc:d7:5a:0c:54:a4:d2:c9:9c:c3:c6:cb:4a:3e:31:bd.
Are you sure you want to continue connecting (yes/no)? yes
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
root@192.168.2.176's password:

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'root@192.168.2.176'"
and check to make sure that only the key(s) you wanted were added.
```

## 2.3 测试与被管理节点的网络连通性

### 2.3.1 测试初始连通性

```shell
[root@ops ~]# ansible all -i 192.168.2.176,192.168.2.181 -m ping
192.168.2.176 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
192.168.2.181 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```

### 2.3.2 测试配置客户端是否成功

创建 `/tmp/test.conf` ，将配置文件传输到所有客户端 `/tmp` 下，检测是否成功：

```shell
[root@ops ~]# touch /tmp/test.conf
[root@ops ~]# ansible all -i 192.168.2.176,192.168.2.181 -m copy -a "src=/tmp/test.conf dest=/tmp/test.conf"
192.168.2.176 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "dest": "/tmp/test.conf",
    "gid": 0,
    "group": "root",
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
    "mode": "0644",
    "owner": "root",
    "secontext": "unconfined_u:object_r:admin_home_t:s0",
    "size": 0,
    "src": "/root/.ansible/tmp/ansible-tmp-1610176524.97-10342-150200890766442/source",
    "state": "file",
    "uid": 0
}
192.168.2.181 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": true,
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "dest": "/tmp/test.conf",
    "gid": 0,
    "group": "root",
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
    "mode": "0644",
    "owner": "root",
    "secontext": "unconfined_u:object_r:admin_home_t:s0",
    "size": 0,
    "src": "/root/.ansible/tmp/ansible-tmp-1610176524.98-10344-72921048389134/source",
    "state": "file",
    "uid": 0
}
```

## 2.4 Ansible 资产

Ansible 的资产为**静态资产**和**动态资产**。

### 2.4.1 静态资产

默认静态资产文件位于 `/etc/ansible/hosts`

```shell
[root@ops ~]# ls /etc/ansible/
ansible.cfg  hosts  roles
[root@ops ansible]# cat hosts

# Ex 1: 无组的主机, 需要在所有组的前面。

## green.example.com
## blue.example.com
## 192.168.100.1
## 192.168.100.10

# Ex 2: 属于 'webservers' group

## [webservers]
## alpha.example.org
## beta.example.org
## 192.168.1.100
## 192.168.1.110

# If you have multiple hosts following a pattern you can specify
# them like this:

## www[001:006].example.com

# Ex 3: A collection of database servers in the 'dbservers' group

## [dbservers]
##
## db01.intranet.mydomain.net
## db02.intranet.mydomain.net
## 10.25.1.56
## 10.25.1.57

# Here's another example of host ranges, this time there are no
# leading 0s:

## db-[99:101]-node.example.com

```

#### 自定义资产文件

创建自定义资产文件 test.ini

```shell
[root@ops ~]# cat test.ini
[test]
192.168.2.176
192.168.2.181
```

列出自定义资产文件里的所有主机 `ansible all -i test.ini --list-hosts`，或者列出某个组的主机 `ansible test -i test.ini --list-hosts`

```shell
[root@ops ~]# ansible all -i test.ini --list-hosts
  hosts (2):
    192.168.2.176
    192.168.2.181
```

<font color=red>提示：可在 IP 后面加入端口号，如 192.168.2.176:2333 (2333端口为改后的ssh端口)</font>





