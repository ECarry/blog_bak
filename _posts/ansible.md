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

# 一、 安装

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

`pip install ansible`

# 二、 Ansible 工具及演示

## 2.1 管理节点与被管理节点建立 ssh 信任关系

### 2.1.1 管理节点（ansible）中创建密钥对

```shell
ssh-keygen -t rsa
[root@ops ~]# ls .ssh/
id_rsa  id_rsa.pub
```

### 2.1.2 将本地公钥传输到被管理节点

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

## 2.2 测试与被管理节点的网络连通性

### 2.2.1 测试初始连通性

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

### 2.2.2 测试配置客户端是否成功

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

## 2.3 Ansible 资产

Ansible 的资产为**静态资产**和**动态资产**。

### 2.3.1 静态资产

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

## 2.4 工具

- `/usr/bin/ansible` 主程序，临时命令执行工具
- `/usr/bin/ansible-doc` 查看配置文档，模块功能查看工具
- `/usr/bin/ansible-galaxy` 下载/上传优秀代码或Roles模块的官网平台
- `/usr/bin/ansible-playbook` 定制自动化任务，编排剧本工具
- `/usr/bin/ansible-pull` 远程执行命令的工具
- `/usr/bin/ansible-vault` 文件加密工具
- `/usr/bin/ansible-console` 基于Console界面与用户交互的执行工具

**利用ansible实现管理的主要方式：**

- **Ad-Hoc** 即利用 ansible 命令，主要用于临时命令使用场景（单条的执行命令）
- **Ansible-playbook** 主要用于长期规划好的，大型项目的场景，需要有前期的规划过程


### 2.4.1 ansible-doc 

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
### 2.4.2 ansible 

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
#ok = green             
#changed = yellow       
#diff_add = green       
#diff_remove = red      
#diff_lines = cyan

#红色：执行失败
#黄色：执行成功并且对目标主机做变更
#绿色：执行成功并且不需要做改变的操作
```

### 2.4.3 ansible-galaxy

此工具会连接 https://galaxy.ansible.com 下载相应的 roles

**示例：**

```shell
#列出所有已安装的galaxy
ansible-galaxy list
#安装galaxy
ansible-galaxy install geerlingguy.mysql
ansible-galaxy install geerlingguy.redis
#删除galaxy
ansible-galaxy remove geerlingguy.redis
```

### 2.4.4 ansible-vault

此工具可以用于加密解密yml文件

**格式：**
`ansible-vault [create|decrypt|edit|encrypt|rekey|view]`

```shell
ansible-vault encrypt hello.yml     #加密
ansible-vault decrypt hello.yml     #解密
ansible-vault view hello.yml        #查看
ansible-vault edit  hello.yml       #编辑加密文件
ansible-vault rekey  hello.yml      #修改口令
ansible-vault create new.yml        #创建新文件
```

## 2.5 常用模块

查看已安装的模块 `ansible-doc -l`

### 2.5.1 Command 模块

**功能：**在远程主机执行命令，此为默认模块，可忽略 -m 选项

<font color=red>注意：</font>此命令不支持 $VARNAME < > | ; & 等，用shell模块实现（变量，通配符，重定向，管道等都不支持）

**示例：**

```
# 在每台主机上创建 hello.txt
ansible all -m command -a 'touch hello.txt'
# 查看 web 组主机的版本号
ansible web -m command -a 'chdir=/etc cat centos-release'
web1 | CHANGED | rc=0 >>
CentOS Linux release 7.8.2003 (Core)
web2 | CHANGED | rc=0 >>
CentOS Linux release 7.8.2003 (Core)
# 在 web 组主机上安装数据库
ansible web -m command -a 'yum install -y mariadb'
```

无法使用变量的参数演示：

```shell
ansible web -m command -a 'echo $HOSTNAME'
web1 | CHANGED | rc=0 >>
$HOSTNAME
web2 | CHANGED | rc=0 >>
$HOSTNAME
```

### 2.5.2 Shell 模块

相当于 Command 的升级版，能够支持更多的参数，可以将默认的 command 模块更改为 shell 模块，修改`/etc/ansible/ansible.cfg `
```conf
# default module name for /usr/bin/ansible
#module_name = command
```

**示例：**

```shell
# 能够使用变量
ansible web -m shell -a 'echo $HOSTNAME'
web1 | CHANGED | rc=0 >>
web1.web.com
web2 | CHANGED | rc=0 >>
web2.web.com
# 重定向也支持
ansible web -m shell -a 'echo hello > hello.txt'
web2 | CHANGED | rc=0 >>

web1 | CHANGED | rc=0 >>
ansible web -m shell -a 'cat hello.txt'
web1 | CHANGED | rc=0 >>
hello
web2 | CHANGED | rc=0 >>
hello
```

### 2.5.3 Script 模块

**功能：**在远程主机上运行 ansible 服务器上的脚本

```shell
# 创建脚本，功能是显示主机名
cat test.sh 
#!/bin/bash

echo "my host name is $HOSTNAME."
# 在 web 组的主机上执行
ansible web -m script -a "/root/test.sh" 
web1 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to web1 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to web1 closed."
    ], 
    "stdout": "my host name is web1.web.com.\r\n", 
    "stdout_lines": [
        "my host name is web1.web.com."
    ]
}
web2 | CHANGED => {
    "changed": true, 
    "rc": 0, 
    "stderr": "Shared connection to web2 closed.\r\n", 
    "stderr_lines": [
        "Shared connection to web2 closed."
    ], 
    "stdout": "my host name is web2.web.com.\r\n", 
    "stdout_lines": [
        "my host name is web2.web.com."
    ]
}
```

### 2.5.4 Copy 模块

**功能：**从 ansible 服务器主控端复制文件到远程主机（支持文件和文件夹）

```shell
ansible web -m copy -a "src=/root/copy_test dest=/root/"
web1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/root/copy_test", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
    "mode": "0644", 
    "owner": "root", 
    "secontext": "system_u:object_r:admin_home_t:s0", 
    "size": 0, 
    "src": "/root/.ansible/tmp/ansible-tmp-1610459738.61-27460-268899158778869/source", 
    "state": "file", 
    "uid": 0
}
web2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709", 
    "dest": "/root/copy_test", 
    "gid": 0, 
    "group": "root", 
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e", 
    "mode": "0644", 
    "owner": "root", 
    "secontext": "system_u:object_r:admin_home_t:s0", 
    "size": 0, 
    "src": "/root/.ansible/tmp/ansible-tmp-1610459738.63-27462-255661350308151/source", 
    "state": "file", 
    "uid": 0
}
ansible web -a "ls /root/"
web2 | CHANGED | rc=0 >>
copy_test
web1 | CHANGED | rc=0 >>
copy_test
```

### 2.5.5 Fetch 模块

**功能：**从远程主机提取文件至 ansible 的主控端，copy 相反，目前不支持目录

```shell
ansible web -m fetch -a "src=/etc/ssh/sshd_config dest=/root/"
web2 | CHANGED => {
    "changed": true, 
    "checksum": "f2b054756f78acb1169cbc8b8f347a69630cb8bc", 
    "dest": "/root/web2/etc/ssh/sshd_config", 
    "md5sum": "40d961cd3154f0439fcac1a50bd77b96", 
    "remote_checksum": "f2b054756f78acb1169cbc8b8f347a69630cb8bc", 
    "remote_md5sum": null
}
web1 | CHANGED => {
    "changed": true, 
    "checksum": "f2b054756f78acb1169cbc8b8f347a69630cb8bc", 
    "dest": "/root/web1/etc/ssh/sshd_config", 
    "md5sum": "40d961cd3154f0439fcac1a50bd77b96", 
    "remote_checksum": "f2b054756f78acb1169cbc8b8f347a69630cb8bc", 
    "remote_md5sum": null
}

tree
.
├── web1
│   └── etc
│       └── ssh
│           └── sshd_config
└── web2
    └── etc
        └── ssh
            └── sshd_config

6 directories, 3 files
```

### 2.5.6 File 模块

**功能：**设置文件属性（可以创建文件，设置文件属组或者权等）

```shell
# 在 web 组主机创建 file_test 文件
ansible web -m file -a 'path=/root/file_test.txt state=touch'
web2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "dest": "/root/file_test.txt", 
    "gid": 0, 
    "group": "root", 
    "mode": "0644", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:admin_home_t:s0", 
    "size": 0, 
    "state": "file", 
    "uid": 0
}
web1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": true, 
    "dest": "/root/file_test.txt", 
    "gid": 0, 
    "group": "root", 
    "mode": "0644", 
    "owner": "root", 
    "secontext": "unconfined_u:object_r:admin_home_t:s0", 
    "size": 0, 
    "state": "file", 
    "uid": 0
}

ansible web -a 'ls /root/'
web1 | CHANGED | rc=0 >>
file_test.txt
web2 | CHANGED | rc=0 >>
file_test.txt

# 更改文件权限和属主
ansible web -m file -a 'path=/root/file_test.txt owner=test mode=777'

# 创建软链接
ansible web -m file -a 'src=/root/file_test.txt dest=/root/file.link state=link'
```

### 2.5.7 Unarchive 模块

**功能：**解包解压缩

实现有两种用法：

1. 将 ansible 主机上的压缩包传到远程主机后解压缩至特定目录，设置 copy=yes
2. 将远程主机上的某个压缩包解压缩到指定路径下，设置 copy=no

常见参数：

- copy：默认为 yes，当 copy=yes，拷贝的文件是从 ansible 主机复制到远程主机上，如果设置为 copy=no，会在远程主机上寻找 src 源文件
- remote_src：和 copy 功能一样且互斥，yes 表示在远程主机，不在 ansible 主机，no 表示文件在 ansible 主机上
- src：源路径，可以是a nsible 主机上的路径，也可以是远程主机上的路径，如果是远程主机上的路径，则需要设置 copy=no
- dest：远程主机上的目标路径
- mode：设置解压缩后的文件权限

```shell
# 将 ansible 主机的 /etc/ 打包压缩，并解压到 web 组主机的 /tmp 目录下
tar zcvf etc.tar.gz /etc/
ls
etc.tar.gz
ansible web -m unarchive -a 'src=/root/etc.tar.gz dest=/tmp'
```

### 2.5.8 Hostname 模块

**功能：**管理主机名

```shell
 ansible <host> -m hostname -a "name=hostname"
 ansible <ip> -m hostname -a 'name=hostname'
```

### 2.5.9 Cron 模块

**功能：**计划任务

支持时间：**minute，hour，day，month，weekday**

```shell
#在 web 组主机上创建 cron 任务
ansible web -m cron -a 'hour=0 minute=0 name="say hello" job=/root/cron_test.sh'
# 检查是否创建任务成功
ansible web -a 'cat /var/spool/cron/root'
web2 | CHANGED | rc=0 >>
#Ansible: say hello
0 0 * * * /root/cron_test.sh
web1 | CHANGED | rc=0 >>
#Ansible: say hello
0 0 * * * /root/cron_test.sh
# 禁用计划任务
ansible web -m cron -a 'hour=0 minute=0 name="say hello" job=/root/cron_test.sh disable=yes'
# 再启用计划任务
ansible web -m cron -a 'hour=0 minute=0 name="say hello" job=/root/cron_test.sh disable=no'
# 删除计划任务
ansible web -m cron -a 'name="say hello" state=absent'
```

### 2.5.10 Yum 模块

**功能：**管理软件包

```shell
# 安装软件包
ansible web -m yum -a 'name=httpd'
# 查看是否安装成功
ansible web -a 'rpm -qa|grep httpd'
web2 | CHANGED | rc=0 >>
httpd-2.4.6-97.el7.centos.x86_64
web1 | CHANGED | rc=0 >>
httpd-2.4.6-97.el7.centos.x86_64
# 删除软件包
ansible web -m yum -a 'name=httpd state=absent'
```

### 2.5.11 User 模块

**功能：**管理用户

```shell
# 创建用户,comment:用户描述 home：指定home目录 group：指定组
ansible web -m user -a 'name=web_user comment="web user" home=/web/web_user group=root'

ansible all -m user -a 'name=nginx comment=nginx uid=88 group=nginx groups="root,daemon" shell=/sbin/nologin system=yes create_home=no  home=/data/nginx non_unique=yes'

# 删除用户,remove:删除home目录
ansible web -m user -a 'name=web_user state=absent remove=yes'
```

### 2.5.12 Lineinfile 模块

ansible 在使用 sed 进行替换时，经常会遇到需要转义的问题，而且 ansible 在遇到特殊符号进行替换时，存在问题，无法正常进行替换。其实在 ansible 自身提供了两个模块：**lineinfile 模块**和**replace 模块**，可以方便的进行替换

**功能：**相当于sed，可以修改文件内容

```shell
# 关闭 web 组主机 selinux
ansible web -a 'cat /etc/selinux/config'
web1 | CHANGED | rc=0 >>

SELINUX=enforcing
SELINUXTYPE=targeted 

web2 | CHANGED | rc=0 >>

SELINUX=enforcing
SELINUXTYPE=targeted 

ansible web -m lineinfile -a "path=/etc/selinux/config regexp='^SELINUX=' line='SELINUX=disabled'"
web2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "backup": "", 
    "changed": true, 
    "msg": "line replaced"
}
web1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "backup": "", 
    "changed": true, 
    "msg": "line replaced"
}
ansible web -a 'cat /etc/selinux/config'web1 | CHANGED | rc=0 >>

SELINUX=disabled
SELINUXTYPE=targeted 

web2 | CHANGED | rc=0 >>

SELINUX=disabled
SELINUXTYPE=targeted 

# 删除 /etc/fstab 以 # 开头的行
ansible web -m lineinfile -a 'dest=/etc/fstab state=absent regexp="^#"'
web1 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "backup": "", 
    "changed": true, 
    "found": 7, 
    "msg": "7 line(s) removed"
}
web2 | CHANGED => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "backup": "", 
    "changed": true, 
    "found": 7, 
    "msg": "7 line(s) removed"
}
```

### 2.5.13 Replace 模块

该模块有点类似于sed命令，主要也是基于正则进行匹配和替换

```shell
# 注释所有以 UUID 开头的行
ansible all -m replace -a "path=/etc/fstab regexp='^(UUID.*)' replace='#\1'"  
# 将注释的行去掉
ansible all -m replace -a "path=/etc/fstab regexp='^#(.*)' replace='\1'"
```

### 2.5.14 Setup 模块

**功能：** setup 模块来收集主机的系统信息，这些 facts 信息可以直接以变量的形式使用，但是如果主机较多，会影响执行速度，可以使用 `gather_facts: no` 来禁止 Ansible 收集 facts 信息

```shell
# 收集主机的所有信息
ansible web -s setup

# 收集指定信息
# 内存信息
ansible web -s setup -a 'filter=ansible_memtotal_mb'
web1 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 7818, 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
web2 | SUCCESS => {
    "ansible_facts": {
        "ansible_memtotal_mb": 7818, 
        "discovered_interpreter_python": "/usr/bin/python"
    }, 
    "changed": false
}
```

# 三、 Playbook

## 3.1 Playbook 简介


## 3.2 YAML

### 3.2.1 YAML 语言

<font color=green>YAML: YAML Ain't Markup Language</font>

YAML 是 "YAML Ain't a Markup Language"（YAML 不是一种标记语言）的递归缩写。在开发的这种语言时，YAML 的意思其实是："Yet Another Markup Language"（仍是一种标记语言）。

YAML 的语法和其他高级语言类似，并且可以简单表达清单、散列表、标量等数据形态。它使用空白符号缩进和大量依赖外观的特色，特别适合用来表达或编辑数据结构、各种配置文件、倾印调试内容、文件大纲（例如：许多电子邮件标题格式和YAML非常接近）。

<font color=red>YAML 的配置文件后缀为 .yml</font>

### 3.2.2 YAML 语法

- 大小写敏感
- 使用缩进表示层级关系
- 缩进不允许使用tab，只允许空格
- 缩进的空格数不重要，只要相同层级的元素左对齐即可
- '#'表示注释

### 3.2.3 数据类型

YAML 支持以下几种数据类型：

- 对象：键值对的集合，又称为映射（mapping）/ 哈希（hashes） / 字典（dictionary）
- 数组：一组按次序排列的值，又称为序列（sequence） / 列表（list）
- 纯量（scalars）：单个的、不可再分的值

学习网站：[YAML 入门教程-菜鸟教程](https://www.runoob.com/w3cnote/yaml-intro.html)

## 3.3 Playbook 核心元素

- Hosts 执行的远程主机列表
- Tasks 任务集
- Variables 内置变量或自定义变量在playbook中调用
- Templates 模板，可替换模板文件中的变量并实现一些简单逻辑的文件
- Handlers 和 notify 结合使用，由特定条件触发的操作，满足条件方才执行，否则不执行
- Tags 标签 指定某条任务执行，用于选择运行 playbook 中的部分代码。ansible 具有幂等性，因此会自动跳过没有变化的部分，即便如此，有些代码为测试其确实没有发生变化的时间依然会非常地长。此时，如果确信其没有变化，就可以通过 tags 跳过此些代码片断

### 3.3.1 Host 组件

Hosts：playbook 中的每一个 play 的目的都是为了让特定主机以某个指定的用户身份执行任务。hosts 用于指定要执行指定任务的主机，须事先定义在主机清单中

```shell
# 主机清单
web1.web.com
web2.web.com
web1.web.com:web2.web.com
192.168.1.50
192.168.1.*
# 主机组清单
web:db      #或，两个组的并集
web:&db     #与，两个组的交集
web:!db     #在 web 组，而不在 db 组
```

示例：

```yaml
- hosts: web:db
```

### 3.3.2 Remote_user 组件

remote_user: 可用于 Host 和 task 中。也可以通过指定其通过 sudo 的方式在远程主机上执行任务，其可用于 play 全局或某任务；此外，甚至可以在 sudo 时使用 sudo_user 指定 sudo 时切换的用户

示例：

```yaml
- hosts: web
  remote_user: root

  tasks:
    - name: test connection
      ping: 
      remote_user: ecarry
      sudo: yes     #默认 sudo 为 root
```

### 3.3.3 Task 列表和 Action 组件

play 的主体部分是 task list，task list 中有一个或多个 task,各个 task 按次序逐个在 hosts 中指定的所有主机上执行，即在所有主机上完成第一个 task 后，再开始第二个task。

task 的目的是使用指定的参数执行模块，而在模块参数中可以使用变量。模块执行是幂等的，这意味着多次执行是安全的，因为其结果均一致。

每个 task 都应该有其 name，用于 playbook 的执行结果输出，建议其内容能清晰地描述任务执行步骤。如果未提供 name，则 action 的结果将用于输出。

#### task 两种格式：
1. action: module arguments
2. module: arguments 建议使用

注意：shell 和 command 模块后面跟命令，而非 key=value

示例：

```yaml
---
- hosts: web
  remote_user: root
  tasks:
    - name: install httpd
      yum: name=httpd
    - name: start httpd
      systemd: name=httpd state=started enable=yes
```

### 3.3.4 其他组件

某任务的状态在运行后为 changed 时，可通过 notify 通知给相应的 handlers，任务可以通过 tags 打标签，可在 ansible-playbook 命令上使用 -t 指定进行调用

### 3.3.5 ShellScript 和 Playbook 对比

使用  ShellScript 安装 apache

```shell
#!/bin/bash
# yum 安装 apache
yum install --quite -y httpd

# 复制已配置好的配置文件
cp /tmp/httpd.conf /etc/httpd/httpd.conf
cp /tmp/vhosts.conf /etc/httpd/conf.d/

# 启动 apache，并设置开机自启
systemctl enable  --now httpd
```

使用 Playbook 实现

```yaml
---
- hosts: web
  remote_user: root
  tasks:
    - name: "install apache"
      yum: name=httpd
    - name: "cp conf"
      copy: src=/tmp/httpd.conf dest=/etc/httpd/httpd.conf
    - name: "cp conf"
      copy: src=/tmp/vhosts.conf dest=/etc/httpd/conf.d/
    - name: "start httpd"
      systemd: name=httpd state=started enabled=yes
```

### 3.3.6 Playbook 命令

格式

`ansible-playbook <filename.yml> ...[options]`

常用选项：

- -C --check          #只检测可能会发生的改变，但不真正执行操作
- --list-hosts        #列出运行任务的主机
- --list-tags         #列出tag
- --list-tasks        #列出task
- --limit 主机列表     #只针对主机列表中的主机执行
- -v -vv  -vvv        #显示过程

示例：

```shell
# 检测
ansible-playbook playbook_install_apache.yml -C

PLAY [web] ********************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
ok: [web2]
ok: [web1]

TASK [install apache] *********************************************************************************************************
changed: [web2]
changed: [web1]

TASK [start httpd] ************************************************************************************************************
changed: [web1]
changed: [web2]

PLAY RECAP ********************************************************************************************************************
web1                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

# 执行
ansible-playbook playbook_install_apache.yml

PLAY [web] ********************************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************************
ok: [web2]
ok: [web1]

TASK [install apache] *********************************************************************************************************
changed: [web1]
changed: [web2]

TASK [start httpd] ************************************************************************************************************
changed: [web1]
changed: [web2]

PLAY RECAP ********************************************************************************************************************
web1                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

## 3.4 Playbook 基础实战

### 3.4.1 使用 Playbook 创建 mysql 账户

```yml
---
- hosts: db
  remote_user: root
# 禁用 setup 收集信息，提高速度
  gather_facts: no

  tasks:
    - name: "add group for mysql"
      group: name=mysql system=yes
    - name: "add user for mysql"
      user: name=mysql system=yes shell=/sbin/nologin system=yes group=mysql home=/data/mysql create_home=no 
```

检查是否创建成功

```shell
ansible db -a 'getent passwd mysql'
192.168.8.144 | CHANGED | rc=0 >>
mysql:x:998:996::/data/mysql:/sbin/nologin
192.168.8.145 | CHANGED | rc=0 >>
mysql:x:998:996::/data/mysql:/sbin/nologin
```

### 3.4.2 使用 Playbook 安装 mysql

```yml
---
- hosts: db
  remote_user: root
# 跳过检测信息，提高速度
  gather_facts: no

  tasks:
    - name: "add group for mysql"
      group: name=mysql system=yes
    - name: "add user for mysql"
      user: name=mysql system=yes shell=/sbin/nologin system=yes group=mysql home=/data/mysql create_home=no 
```

## 3.5 Playbook handlers 和 notify

Handlers 本质是 task list，类似于 Mysql 中的触发器触发行为，其中的 task 与前述的 task 并没有本质上的不同，主要用于当关注的资源发生变化时，才会采取一定的操作。而 Notify 对应的 action 可用于在每个 play 的最后被触发，这样可以避免多次有改变发生时每次都执行指定的操作，仅在所有的变化发生完后一次性地执行指定操作。


监听 nginx 配置文件更改，重启 nginx 服务

```yml
---
- hosts: web
  remote_user: root
  gather_facts: no

  tasks:
    - name: install epel
      yum: name=epel-release state=present
    - name: install nginx
      yum: name=nginx state=present
    - name: copy config
      copy: src=/root/nginx.conf dest=/etc/nginx/
      # 监听文件是否改动，改动则重启 nginx，名字需要与 handlers name 相同
      notify: restart nginx
    - name: start nginx
      systemd: name=nginx state=started enabled=yes
  
  handlers:
    - name: restart nginx
      systemd: name=nginx state=restarted
    # 检测 nginx 进程是否启动，使用 kill 发送 0 信号，0 信号代表 不发送任何信号，但是系统会进行错误检查。所以经常用来检查一个进程是否存在，存在返回0；不存在返回1;
    - name: check nginx process
      shell: killall -0 nginx &> /tmp/nginx.log
```

安装到 web 组主机 `ansible-playbook playbook_install_nginx.yml`

查看 web 主机端口：

```shell
netstat -antulp |grep 80
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      20281/nginx: master 
tcp6       0      0 :::80                   :::*                    LISTEN      20281/nginx: master 
```

修改配置文件，将端口改为8888，再次执行 `ansible-playbook playbook_install_nginx.yml`

```shell
ansible-playbook playbook_install_nginx.yml 

PLAY [web] *********************************************************************************************************************************************************

TASK [install epel] ************************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [install nginx] ***********************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [copy config] *************************************************************************************************************************************************
changed: [web2]
changed: [web1]

TASK [start nginx] *************************************************************************************************************************************************
ok: [web1]
ok: [web2]

RUNNING HANDLER [restart nginx] ************************************************************************************************************************************
changed: [web1]
changed: [web2]

PLAY RECAP *********************************************************************************************************************************************************
web1                       : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

查看 web 主机端口已经变为 8888：

```shell
netstat -antulp|grep 8888
tcp        0      0 0.0.0.0:8888            0.0.0.0:*               LISTEN      4559/nginx: master  
tcp6       0      0 :::8888                 :::*                    LISTEN      4559/nginx: master
```

## 3.6 Playbook 的 tags 组件

在 playbook 中，可以利用 tags 组件，为特定 task 指定标签，当执行 playbook 时，可以只执行特定 tags 的 task，而非整个 playbook 文件

示例：

`vi playbook_install_nginx.yml`
```yml
---
- hosts: web
  remote_user: root
  gather_facts: no

  tasks:
    - name: install epel
      yum: name=epel-release state=present
    - name: install nginx
      yum: name=nginx state=present
    - name: copy config
      copy: src=/root/nginx.conf dest=/etc/nginx/
      # 监听文件是否改动，改动则重启 nginx，名字需要与 handlers name 相同
      notify: restart nginx
      tags: conf
    - name: start nginx
      systemd: name=nginx state=started enabled=yes
  
  handlers:
    - name: restart nginx
      systemd: name=nginx state=restarted
    # 检测 nginx 进程是否启动，使用 kill 发送 0 信号，0 信号代表 不发送任何信号，但是系统会进行错误检查。所以经常用来检查一个进程是否存在，存在返回0；不存在返回1;
    - name: check nginx process
      shell: killall -0 nginx &> /tmp/nginx.log
```

列出 playbook 中的标签：

```shell
ansible-playbook --list-tags playbook_install_nginx.yml 

playbook: playbook_install_nginx.yml

  play #1 (web): web    TAGS: []
      TASK TAGS: [conf]
```

更改配置文件，将端口配置为8081，执行标签

```shell
ansible-playbook -t conf playbook_install_nginx.yml 

PLAY [web] *********************************************************************************************************************************************************

TASK [copy config] *************************************************************************************************************************************************
changed: [web2]
changed: [web1]

RUNNING HANDLER [restart nginx] ************************************************************************************************************************************
changed: [web1]
changed: [web2]

PLAY RECAP *********************************************************************************************************************************************************
web1                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

查看主机端口：

```shell
netstat -antulp |grep 8081
tcp        0      0 0.0.0.0:8081            0.0.0.0:*               LISTEN      6189/nginx: master  
tcp6       0      0 :::8081                 :::*                    LISTEN      6189/nginx: master  
```

## 3.7 Playbook 使用变量

**变量名：**仅能由字母、数字和下划线组成，且只能以字母开头

变量的定义：

`variable=value`

示例：

`http_port=80`

变量调用方式：

`{{ variable_name }}`

**变量来源：**

1. ansible 的 setup facts 远程主机的所有变量都可直接调用，如 {{ ansible_distribution }} linux 发行版
2. 通过命令行指定变量，优先级最高 `ansible-playbook -e varname=value`
3. 在playbook文件中定义

```yml
vars:
  - var1: value1
  - var2: value2
```
4. 在独立的变量 YAML 文件中定义
```yml
- host
  vars_files:
    - vars.yml
```

5. 在 /etc/ansible/hosts 中定义

主机（普通）变量：主机组中主机单独定义，优先级高于公共变量
组（公共）变量：针对主机组中所有主机定义统一变量

6. 在role中定义




### 3.7.1 使用 setup 中的变量

使用 setup 中的变量来命名文件名:

<font color=red>注意：要使用 setup 中的变量，需要 `gather_facts: yes`</font>

```yml
---
- hosts: web
  remote_user: root

  tasks:
    - name: create nodename log file in /var/log
      file: name=/var/log/{{ ansible_nodename }}.log state=touch mode=700
```

```shell
ansible-playbook playbook_setup_var.yml

PLAY [web] *********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [create nodename log file in /var/log] ************************************************************************************************************************
changed: [web1]
changed: [web2]

PLAY RECAP *********************************************************************************************************************************************************
web1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

```shell
ls /var/log/ |grep web
web1.web.com.log
```

### 3.7.2 在 playbook 中定义变量


```yml
---
- hosts: web
  remote_user: root
  # 定义变量
  vars:
    - username: ecarry
    - group: ecarry

  tasks:
    - name: create group ecarry
    # 引用变量
      group: name={{ group }} state=present
    - name: create user ecarry
      user: name={{ username }} state=present
```

执行

```shell
ansible-playbook playbook_create_user.yml 

PLAY [web] *********************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************
ok: [web1]
ok: [web2]

TASK [create group ecarry] *****************************************************************************************************************************************
ok: [web2]
ok: [web1]

TASK [create user ecarry] ******************************************************************************************************************************************
ok: [web1]
ok: [web2]

PLAY RECAP *********************************************************************************************************************************************************
web1                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=3    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

检查是否创建成功：

```shell
ansible web -m shell -a 'cat /etc/passwd |grep ecarry'
web2 | CHANGED | rc=0 >>
ecarry:x:1000:1000:ecarry:/home/ecarry:/bin/bash
web1 | CHANGED | rc=0 >>
ecarry:x:1000:1000:ecarry:/home/ecarry:/bin/bash
```

### 3.7.3 在 playbook 命令行中使用变量

写一个安装包的 playbook，在 ansible-playbook 命令行中使用 `ansible-playbook -e varname=pkgname` 定义变量
```yml
cat playbook_install_package.yml 
---
- hosts: web
  remote_user: root
  
  tasks:
    - name: install packages
      yum: name={{ pkname }} state=present 
```

在 web 组主机安装 vim

```shell
ansible-playbook -e pkname=vim playbook_install_package.yml 

PLAY [web] **********************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************
ok: [web1]
ok: [web2]

TASK [install packages] *********************************************************************************************************
changed: [web2]
changed: [web1]

PLAY RECAP **********************************************************************************************************************
web1                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web2                       : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
```

### 3.7.4 使用变量文件

创建变量文件 `pak_var.yml`：

```yml
# variable file
package_name: nginx
service_name: nginx
```

playbook 文件 `playbook_install_package.yml`:

```yml
---
# isntall package and config service
- hosts: web
  remote_user: root
  vars_files: 
    - pak_var.yml 

  tasks:
    - name: install epel
      yum: name=epel-release
    - name: install packages
      yum: name={{ package_name }} state=present 
    - name: start service
      systemd: name={{ service_name }} enabled=yes state=started

```

### 3.7.5 主机清单文件中定义变量

#### 主机变量：

在 inventory 主机清单文件中为指定的主机定义变量以便于在playbook中使用
```
[web]
web1.web.com http_port=80 
web2.web.com http_port=8080
```

#### 组（公共）变量

在 inventory 主机清单文件中赋予给指定组内所有主机上的在 playbook 中可用的变量，如果和主机变是同名，优先级低于主机变量
```
[web]
web1.web.com
web2.web.com

[web:vars]
ntp_server=ntp.magedu.com
nfs_server=nfs.magedu.com
```

## 3.8 Playbook template

模板是一个文本文件，可以做为生成文件的模版，并且模板文件中还可嵌套 jinja 语法

### 3.8.1 jinja 语言




### 3.8.2 template

**功能：**可以根据和参考模块文件，动态生成相类似的配置文件

**template文件**必须存放于 **templates 目录**下，且命名为 **.j2** 结尾


利用 template 文件 `nginx.conf.j2` 配置 nginx：

```j2
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

# 从 ansible setup 获取主机 CPU 信息，为 nginx 配置进程数
worker_processes {{ ansible_processor_vcpus }};
```

playbook 文件 `install_nginx.yml`：

```yml
---
# install nginx and config with template
- hosts: web
  remote_user: root

  tasks:
    - name: install epel
      yum: name=epel-release state=present
    - name: install nginx
      yum: name=nginx state=present
    - name: template config nginx
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      notify: restart nginx
    - name: start nginx
      systemd: name=nginx state=started enabled=yes

  handlers:
    - name: restart nginx
      systemd:  name=nginx state=restarted
```

template 支持算数运算：

```j2
# vim nginx.conf.j2 
worker_processes {{ ansible_processor_vcpus**2 }};    
worker_processes {{ ansible_processor_vcpus+2 }}; 
```

### 3.8.3 template 的流程控制

### 3.8.3 template 使用 when


# 四、 Roles

**角色**是 ansible 自1.2版本引入的新特性，用于**层次性**、**结构化**地组织 playbook。roles 能够根据层次型结构自动装载变量文件、tasks 以及handlers 等。要使用 roles 只需要在 playbook 中使用 include 指令即可。简单来讲，roles 就是通过分别将**变量**、**文件**、**任务**、模**板**及**处理器**放置于单独的目录中，并可以便捷地 include 它们的一种机制。角色一般用于基于主机构建服务的场景中，但也可以是用于构建守护进程等场景中

**roles：**多个角色的集合，可以将多个的 role（如：mysql、nginx等），分别放至roles目录下的独立子目录中

## 4.1 Roles 的目录结构

![](/img/ansible/ansible_4.png)
![](/img/ansible/ansible_3.png)

## 4.2 Roles 各目录作用

roles/project/ :项目名称,有以下子目录

- files/ ：存放由 copy 或 script 模块等调用的文件
- templates/：template 模块查找所需要模板文件的目录
- tasks/：定义 task，role 的基本元素，至少应该包含一个名为 main.yml 的文件；其它的文件需要在此文件中通过 include 进行包含
- handlers/：至少应该包含一个名为 main.yml 的文件；其它的文件需要在此文件中通过 include 进行包含
- vars/：定义变量，至少应该包含一个名为 main.yml 的文件；其它的文件需要在此文件中通过 include 进行包含
- meta/：定义当前角色的特殊设定及其依赖关系,至少应该包含一个名为 main.yml 的文件，其它文件需在此文件中通过 include 进行包含
- default/：设定默认变量时使用此目录中的 main.yml 文件，比 vars 的优先级低

## 4.3 创建 Roles

1. 创建以 roles 命名的目录
2. 在 roles 目录中分别创建以各角色名称命名的目录，如 nginx 等
3. 在每个角色命名的目录中分别创建 files、handlers、meta、tasks、templates 和 vars 目录；用不到的目录可以创建为空目录，也可以不创建
4. 在 playbook 文件中，调用各角色

示例：创建 nginx roles

```bash
tree nginx/
nginx/
├── files
├── tasks
│   ├── install_nginx.yml
│   ├── main.yml
│   └── restart.yml
└── vars
    └── main.yml

3 directories, 4 files
```

## 4.4 Playbook 调用 Roles

### 4.4.1 方法1

```yml
---
- hosts: web
  remote_user: root
  roles:
    - nginx
    - mysql
```

## 4.5 Roles 实战

### 4.5.1 实现 nginx 角色

