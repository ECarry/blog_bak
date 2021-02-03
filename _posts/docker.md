---
layout: post
title: Docker 简单上手
tags: [Docker]
index_img: https://th.wallhaven.cc/small/r7/r7jolj.jpg
banner_img: https://w.wallhaven.cc/full/r7/wallhaven-r7jolj.png
categories: [Docker]
date: 2020-10-13 15:21:46
---

# 一、 Docker 简介

## 1.1 是什么

- 使用最广泛的开源容器引擎
- 一种操作系统级别的虚拟化技术
- 依赖于 Linux 内核特性：**Namespace**（<font color=red>资源隔离</font>）和 **Cgroups**（<font color=red>资源限制</font>）
- 一个简单的应用程序打包工具

## 1.2 基本组成

- Docker Client：客户端
- Docker Daemon：守护进程
- Docker Images：镜像
- Docker Container：容器
- Docker Registry：镜像仓库

![](https://docs.docker.com/engine/images/architecture.svg)
  
## 1.3 容器 VS VM

![](/img/Docker/Docker_1.jpg)

||容器|VM|
|---|---|---|
|启动速度|秒级|分钟级|
|运行性能|接近原生|5%左右损失|
|磁盘占用|MB|GB|
|数量|成百上千|一般几十台|
|隔离性|进程隔离|系统级（彻底）|
|操作系统|只支持 Linux|所有|
|封装进度|只打包项目代码和依赖关系，共享宿主机内核|完整的操作系统|

## 1.4 应用场景

- 应用程序打包和发布
- 应用程序隔离
- 持续的集成
- 部署微服务
- 快速搭建测试环境
- 提供 PaaS（Platform as a Service），指平台即服务

# 二、 Docker 安装

## 2.1 CentOS 7 安装（使用 yum 安装）

### 官方安装手册

地址：https://docs.docker.com/engine/install/centos/

### 阿里源安装手册（推荐）

使用[阿里源](https://developer.aliyun.com/mirror/docker-ce?spm=a2c6h.13651102.0.0.3e221b11q3kwWA)安装 Docker


删除旧版本的 docker：

```
$ sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

安装步骤：

```
# step 1: 安装必要的一些系统工具
sudo yum install -y yum-utils device-mapper-persistent-data lvm2
# Step 2: 添加软件源信息
sudo yum-config-manager --add-repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# Step 3: 替换成 aliyun 软件源
sudo sed -i 's+download.docker.com+mirrors.aliyun.com/docker-ce+' /etc/yum.repos.d/docker-ce.repo
# Step 4: 更新 yum 软件包索引并安装Docker-CE
sudo yum makecache fast
sudo yum -y install docker-ce
# Step 5: 开启Docker服务
sudo service docker start

# 注意：
# 官方软件源默认启用了最新的软件，您可以通过编辑软件源的方式获取各个版本的软件包。例如官方并没有将测试版本的软件源置为可用，您可以通过以下方式开启。同理可以开启各种测试版本等。
# vim /etc/yum.repos.d/docker-ee.repo
#   将[docker-ce-test]下方的enabled=0修改为enabled=1
#
# 安装指定版本的Docker-CE:
# Step 1: 查找Docker-CE的版本:
# yum list docker-ce.x86_64 --showduplicates | sort -r
#   Loading mirror speeds from cached hostfile
#   Loaded plugins: branch, fastestmirror, langpacks
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            docker-ce-stable
#   docker-ce.x86_64            17.03.1.ce-1.el7.centos            @docker-ce-stable
#   docker-ce.x86_64            17.03.0.ce-1.el7.centos            docker-ce-stable
#   Available Packages
# Step2: 安装指定版本的Docker-CE: (VERSION例如上面的17.03.0.ce.1-1.el7.centos)
# sudo yum -y install docker-ce-[VERSION]
```

校验安装：

```
[root@test ~]# docker version
Client: Docker Engine - Community
 Version:           19.03.13
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        4484c46d9d
 Built:             Wed Sep 16 17:03:45 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.13
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       4484c46d9d
  Built:            Wed Sep 16 17:02:21 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.3.7
  GitCommit:        8fba4e9a7d01810a393d5d25a3621dc101981175
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

运行 **HELLO WORLD**:

```
[root@test ~]# docker run hello-world
#由于本地没有 hello-world 这个镜像，所以会下载一个 hello-world 的镜像，并在容器中运行
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:4cf9c47f86df71d48364001ede3a4fcd85ae80ce02ebad74156906caff5378bc
Status: Downloaded newer image for hello-world:latest

#上面安装，下面运行
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

**run** 启动流程：

![](/img/Docker/Docker_2.jpg)

## 2.2 获取镜像

### Docker Hub 官方镜像

官方镜像地址：https://hub.docker.com/explore

### Docker 镜像加速

国内从 DockerHub 拉取镜像有时会遇到困难，此时可以配置镜像加速器。Docker 官方和国内很多云服务商都提供了国内加速器服务，例如：

- 网易：https://hub-mirror.c.163.com/
- 阿里云：https://<你的ID>.mirror.aliyuncs.com

阿里云镜像获取地址：https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors ，登录后可见加速地址。

![](/img/Docker/Docker_3.jpg)

通过修改 `/etc/docker/daemon.json` 文件配置镜像加速器：

```
#-p 确保目录名称存在，不存在的就建一个。
sudo mkdir -p /etc/docker
#下面加速二选一，如果效果不理想，试着切换
#163加速
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  {"registry-mirrors": ["http://hub-mirror.c.163.com"] }
}
EOF
#阿里源加速
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ID.mirror.aliyuncs.com"]
}
EOF

cat /etc/docker/daemon.json 
{
  "registry-mirrors": ["https://ID.mirror.aliyuncs.com"]
}
sudo systemctl daemon-reload
sudo systemctl restart docker
```

# 三、 Docker 常用命令

## 3.1 帮助命令

### 3.1.1 docker version

```
[root@test ~]# docker version
Client: Docker Engine - Community
 Version:           19.03.13
 API version:       1.40
 Go version:        go1.13.15
 Git commit:        4484c46d9d
 Built:             Wed Sep 16 17:03:45 2020
 OS/Arch:           linux/amd64
 Experimental:      false

Server: Docker Engine - Community
 Engine:
  Version:          19.03.13
  API version:      1.40 (minimum version 1.12)
  Go version:       go1.13.15
  Git commit:       4484c46d9d
  Built:            Wed Sep 16 17:02:21 2020
  OS/Arch:          linux/amd64
  Experimental:     false
 containerd:
  Version:          1.3.7
  GitCommit:        8fba4e9a7d01810a393d5d25a3621dc101981175
 runc:
  Version:          1.0.0-rc10
  GitCommit:        dc9208a3303feef5b3839f4323d9beb36df0a9dd
 docker-init:
  Version:          0.18.0
  GitCommit:        fec3683
```

### 3.1.2 docker info
### 3.1.3 docker --help

## 3.2 镜像命令

|指令|描述|
|---|---|
|ls|列出镜像|
|build|构建镜像来自 Dockerfile|
|history|查看镜像历史|
|inspect|显示一个或多个镜像详细信息|
|pull|从镜像仓库拉去镜像|
|push|推送一个或多个镜像到镜像仓库|
|rm|移除一个或者多个镜像|
|prune|移除未使用的镜像。没有被标记或被任何容器引用的|
|tag|创建一个引用源镜像标记目标镜像|
|export|导出容器文件系统到tar归档文件|
|import|导入容器文件系统tar归档文件创建镜像|
|save|保存一个或多个镜像到一个tar归档文件|
|load|加载镜像来自tar归档或标准输入|


### 3.2.1 docker images

显示本地镜像：
```
[root@test ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               latest              992e3b7be046        6 days ago          133MB
hello-world         latest              bf756fb1ae65        9 months ago        13.3kB
```
#### options:
- -a ：列出本地所有镜像
- -q ：只显示镜像 ID
- --digests ：显示镜像的摘要信息
- --no-trunc ：显示完整的镜像信息

### 3.2.2 docker search 

`docker search 指定镜像名`

```
[root@test ~]# docker search nginx
NAME                               DESCRIPTION                                     STARS               OFFICIAL            AUTOMATED
nginx                              Official build of Nginx.                        13838               [OK]                
jwilder/nginx-proxy                Automated Nginx reverse proxy for docker con…   1893                                    [OK]
richarvey/nginx-php-fpm            Container running Nginx + PHP-FPM capable of…   790                                     [OK]
linuxserver/nginx                  An Nginx container, brought to you by LinuxS…   127                                     
jc21/nginx-proxy-manager           Docker container for managing Nginx proxy ho…   96                                      
tiangolo/nginx-rtmp                Docker image with Nginx using the nginx-rtmp…   95                                      [OK]
```
#### options:

docker search [options] 镜像名字

- --no-trunc ：显示完整的镜像描述
- -s ：列出收藏数不小于指定值的镜像
- --automated ：只列出 automated build 类型镜像

### 3.2.3 docker pull

`docker pull 镜像名[:TAG]` ：下载镜像，不加 TAG 默认下载 latest

```
[root@test ~]# docker pull nginx
Using default tag: latest
latest: Pulling from library/nginx
```

### 3.2.4 docker image rm 

`docker image rm 镜像ID` ：删除镜像

- docker image rm -f 镜像ID ：删除单个
- docker image rm -f 镜像1ID 镜像2ID ：删除多个镜像
- docker image rm -f $(docker images -qa) ：删除所有镜像

## 3.3 容器命令

<font color=red>有镜像才能够创建容器</font>，下载 CentOS 镜像演示：`docker pull centos`

### 3.3.1 新建并启动容器

`docker run [options] image [command] [arg]`

#### options
- --name="new name" ：为容器指定一个名称
- -d,-detach ：后台运行容器，并返回容器ID，也即启动守护式容器
- -e,-env ：设置环境变量
- -i,-interactive ：以交互模式运行容器，通常与 -t 同时使用
- -t,-tty ：为容器重新分配一个伪终端，通常与 -i 同时使用
- -P ：发布容器所有 EXPOSE 的端口到宿主机随机端口
- -p ：发布容器端口到主机
  - ip:hostPort:containerPort
  - ip::containerPort
  - hostPort:containerPort
  - containerPort
- -h ：设置容器主机名
- -ip string ：指定容器 ip，只能用于自定义网络
- -network ：连接容器到一个网络
- -mount mount ：将文件系统附加到容器
- -v，-volume list ：绑定挂载一个卷
- -restart string ：容器退出时重启策略，默认 no，可选值：[always|on-failure]


```
#使用镜像centos:latest以交互模式启动一个容器，在容器内执行/bin/bash命令
[root@test ~]# docker run -it centos /bin/bash
[root@7808a5311ae1 /]# cat /etc/redhat-release 
CentOS Linux release 8.2.2004 (Core)
```

### 3.3.2 列出当前运行的容器

`docker ps [options]`

#### options
- -a ：列出当前所有正在运行的容器和历史上运行过的
- -l ：显示最新创建的容器
- -q ：静默模式，只显示容器编号
- --no-trunc ：不截断输出

### 3.3.3 退出容器

#### 容器停止并退出

`exit`

#### 容器不停止退出

`ctrl+P+Q`

### 3.3.4 启动容器

`docker start 容器ID或者容器名`

### 3.3.5 重启容器

`docker restart 容器ID或者容器名`

### 3.3.6 停止容器

`docker stop 容器ID或者容器名`

### 3.3.7 强制停止容器

`docker kill 容器ID或者容器名`

### 3.3.8 删除已停止的容器

`docker rm 容器ID`

一次性删除多个容器：

- `docker rm -f $(docker ps -a -q)`
- `docker ps -a -q | xargs docker rm`

## 3.4 比较重要的命令

### 3.4.1 启动守护式容器

`docker run -d 容器名`


使用镜像centos:latest以后台模式启动一个容器`docker run -d centos`

**问题**：
然后 `docker ps -a` 进行查看, 会发现容器已经退出。很重要的要说明的一点: <font color=red>Docker容器后台运行，就必须有一个前台进程。</font>容器运行的命令如果不是那些<font color=red>一直挂起的命令</font>（比如运行top，tail），就是会自动退出的。
这个是 docker 的机制问题，比如你的 web 容器，我们以 nginx 为例，正常情况下，我们配置启动服务只需要启动响应的 service 即可。例如 `service nginx start`，但是，这样做，nginx 为后台进程模式运行，就导致 docker 前台没有运行的应用，这样的容器后台启动后，会立即自杀因为他觉得他没事可做了。所以，最佳的解决方案是,将你要运行的程序以前台进程的形式运行

### 3.4.2 查看容器日志

`docker logs -f -t --tail 容器ID`
- -t ：加入时间戳
- -f ：跟随最新的日志打印
- --tail 数字 显示最后几条

### 3.4.3 查看容器内运行的进程

`docker top 容器ID`

### 3.4.4 查看容器内部细节

`docker inspect 容器ID`

### 3.4.5 进入正在运行的容器并以命令行交互

#### docker exec

`docker exec -it 容器ID /bin/bash`，在容器中打开终端，并且可以启动新的进程

```
[root@test ~]# docker ps -a 
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                      PORTS               NAMES
150fcfbacdfa        centos              "/bin/bash"              3 minutes ago       Up 3 minutes                                    flamboyant_driscoll
[root@test ~]# docker exec -it 150fcfbacdfa /bin/bash
[root@150fcfbacdfa /]# ps -ef
UID        PID  PPID  C STIME TTY          TIME CMD
root         1     0  0 12:08 pts/0    00:00:00 /bin/bash
root        14     0  0 12:13 pts/1    00:00:00 /bin/bash
root        27    14  0 12:13 pts/1    00:00:00 ps -ef
```

#### docker attach

`docker attach 容器ID`，直接进入容器启动命令终端，不会启动新的进程

### 3.4.6 从容器内拷贝文件到主机上

`docker cp 容器ID:容器内路径 目的主机路径`

```
#在容器内创建 hello 文件
[root@b7394537e333 ~]# pwd
/root
[root@b7394537e333 ~]# ls
hello
#将容器内 /root/hello 文件拷贝到主机 /tmp 下
[root@test ~]# docker cp b7394537e333:/root/hello /tmp/
[root@test ~]# ls /tmp/
hello
```

# 四、 Docker 镜像

## 4.1 镜像是什么

- 一个分层存储的文件
- 一个软件的环境
- 一个镜像可以创建 N 个容器
- 一种标准化的交付
- 一个不包含 Linux 内核而又精简的 Linux 操作系统

镜像不是一个单一的文件，而是有多层结构。我们可以通过 `docker history <ID/NAME>` 查看镜像中各层内容及大小，每层对应着 Dockerfile 中的一条指令。Docker 镜像默认存储在 `/var/lib/docker/\<storage-driver\>中。

```
[root@docker ~]# docker history nginx
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
7e4d58f0e5f3        4 weeks ago         /bin/sh -c #(nop)  CMD ["nginx" "-g" "daemon…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  STOPSIGNAL SIGTERM           0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  EXPOSE 80                    0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENTRYPOINT ["/docker-entr…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop) COPY file:0fd5fca330dcd6a7…   1.04kB              
<missing>           4 weeks ago         /bin/sh -c #(nop) COPY file:1d0a4127e78a26c1…   1.96kB              
<missing>           4 weeks ago         /bin/sh -c #(nop) COPY file:e7e183879c35719c…   1.2kB               
<missing>           4 weeks ago         /bin/sh -c set -x     && addgroup --system -…   63.4MB              
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV PKG_RELEASE=1~buster     0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV NJS_VERSION=0.4.3        0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  ENV NGINX_VERSION=1.19.2     0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  LABEL maintainer=NGINX Do…   0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop)  CMD ["bash"]                 0B                  
<missing>           4 weeks ago         /bin/sh -c #(nop) ADD file:e7407f2294ad23634…   69.2MB    
```

### 4.1.1 UnionFS

UnionFS（联合文件系统）：Union文件系统（UnionFS）是一种**分层**、**轻量级**并且**高性能**的文件系统，它支持对文件系统的修改作为<font color=red>一次提交来一层层的叠加</font>，同时可以将不同目录挂载到同一个虚拟文件系统下(unite several directories into a single virtual filesystem)。Union 文件系统是 Docker 镜像的基础。镜像可以通过分层来进行继承，基于基础镜像（没有父镜像），可以制作各种具体的应用镜像。

**特性**：一次同时加载多个文件系统，但从外面看起来，只能看到一个文件系统，联合加载会把各层文件系统叠加起来，这样最终的文件系统会包含所有底层的文件和目录

镜像文件分层结构的好处--**共享资源**。比如：有多个镜像都从相同的 base 镜像构建而来，那么宿主机只需在磁盘上保存一份base镜像，同时内存中也只需加载一份 base 镜像，就可以为所有容器服务了。而且镜像的每一层都可以被共享。

### 4.1.2 Docker 镜像加载原理

docker的镜像实际上由一层一层的文件系统组成，这种层级的文件系统UnionFS。

bootfs(boot file system)主要包含 **bootloader** 和 **kernel** , **bootloader** 主要是引导加载 kernel, Linux 刚启动时会加载 bootfs 文件系统，<font color=red>在 Docker 镜像的最底层是 bootfs</font>。这一层与我们典型的 Linux/Unix 系统是一样的，包含 boot 加载器和内核。当 boot 加载完成之后整个内核就都在内存中了，此时内存的使用权已由 bootfs 转交给内核，此时系统也会卸载 bootfs。

rootfs (root file system) ，在 bootfs 之上。包含的就是典型 Linux 系统中的 /dev, /proc, /bin, /etc 等标准目录和文件。rootfs 就是各种不同的操作系统发行版，比如 Ubuntu，Centos 等等。 

![](/img/Docker/Docker_5.png)

## 4.2 镜像特点

Docker 镜像都是只读的，当容器启动时，一个新的可写层被加载到镜像的顶部。这一层通常被称为“容器层”，“容器层”下都叫“镜像层”。

## 4.3 镜像与容器的联系

![](/img/Docker/Docker_4.jpg)

如图，容器其实是在镜像的最上面加了一层读写层，在运行容器里文件改动时，会先从镜像里要写的文件复制到容器自己的文件系统中（读写层）。如果容器删除了，最上面的读写层也就删除了，改动也就丢失了。所以无论多少个容器共享一个镜像，所做的写操作都是从镜像的文件系统中复制过来操作的，<font color=red>并不会修改镜像的源文件</font>，这种方式<font color=red>提高磁盘利用率</font>。
若想持久化这些改动，可以通过 `docker commit` 将容器保存成一个新镜像。
- 一个镜像创建多个容器
- 镜像增量式存储
- 创建的容器里面修改不会影响到镜像

## 4.4 镜像 commit 操作

`docker commit [options]` ：提交容器副本使之成为一个新的镜像

- -m ：提交的描述信息
- -a ：作者

`docker commit -m "xxxx" -a "xxx" 容器ID new_image_name:[tag] ` ：将容器打包成新的镜像

# 五、 Docker 容器

## 5.1 容器资源限制

|选项|描述|
|---|---|
|-m,-memory|容器可以使用的最大内存|
|-memory-swap|允许交换到磁盘的内存量|
|-memory-swappiness=<0-100>|容器使用SWAP分区交换的百分比（0-100，默认为-1）|
|-oom-kill-disable|禁用OOM Killer|
|-cpus|可使用的CPU数量|
|-cpuset-cpus|限制容器使用特定的CPU核心，如（0-3）|
|-cpu-shares|CPU共享（相对权重）|

### 5.1.1 内存限额

允许容器最多使用 500MB 内存和 100MB 的 Swap，并禁用 OOM Killer：

`docker run -d --name nginx01 --memory="500m" --memory-swap="600m --oom-kill-disable nginx`

### 5.1.2 CPU 限制

允许容器使用两个 CPU：

`docker run -d --name nginx02 --cpus="2" nginx`

允许容器最多使用 50% 的 CPU：

`docker run -d --name nginx03 --cpus=".5" nginx`

### 5.1.3 正在运行的容器限制

`docker update`

## 5.2 将数据从宿主机挂载到容器中的三种方式

Docker提供三种方式将数据从宿主机挂载到容器中：
- volumes：Docker 管理宿主机文件系统的一部分（/var/lib/docker/volumes）。保存数据的最佳方式。
- bind mounts：将宿主机上的任意位置的文件或者目录挂载到容器中。
- tmpfs：挂载存储在主机系统的内存中，而不会写入主机的文件系统。如果不希望将数据持久存储在任何位置，可以使用 tmpfs，同时避免写入容器可写层提高性能。

![](/img/Docker/Docker_6.jpg)

### 5.2.1 Volume

**管理卷**

- `docker volume create volume_name` ：创建一个数据卷
- `docker volume ls` ：列出已创建的数据卷
- `docker volume inspect volume_mame` ：显示所创建数据卷的信息
- `docker volume rm volume_name` ：删除一个或者多个数据卷


#### 创建数据卷

```
[root@test volumes]# docker volume create nginx_vol
nginx_vol
[root@test volumes]# docker volume ls
DRIVER              VOLUME NAME
local               nginx_vol
#查看所创建的数据卷
[root@test volumes]# docker volume inspect nginx_vol
[
    {
        "CreatedAt": "2020-10-13T20:55:56+08:00",
        "Driver": "local",
        "Labels": {},
        "Mountpoint": "/var/lib/docker/volumes/nginx_vol/_data",
        "Name": "nginx_vol",
        "Options": {},
        "Scope": "local"
    }
]
#数据卷在宿主机上的目录
[root@test volumes]# pwd
/var/lib/docker/volumes
[root@test volumes]# tree
.
└── nginx_vol
    └── _data
```

**容器中重要数据可以存放到数据卷当中，当容器删除，存放在数据卷中的数据将不会丢失，且能够实现数据共享**

#### 将数据卷挂载到将要创建的容器中

创建容器 nginx01，并将数据卷 nginx_vol 挂在到容器中：

```
[root@test volumes]# docker run -d --name nginx01 --mount src=nginx_vol,dst=/usr/share/nginx/html -p 2333:80 nginx
[root@test _data]# docker inspect nginx01
        "Mounts": [
            {
                "Type": "volume",
                "Name": "nginx_vol",
                "Source": "/var/lib/docker/volumes/nginx_vol/_data",
                "Destination": "/usr/share/nginx/html",
                "Driver": "local",
                "Mode": "z",
                "RW": true,
                "Propagation": ""
            }
        ],
[root@test volumes]# tree
.
└── nginx_vol
    └── _data
        ├── 50x.html
        └── index.html
```

修改主页内容，显示 Hello Nginx!，打开网页

![](/img/Docker/Docker_7.jpg)

创建容器 nginx02，并将数据卷 nginx_vol 挂在到容器中：

```
[root@test _data]# docker run -d --name nginx02 --mount src=nginx_vol,dst=/usr/share/nginx/html -p 2334:80 nginx
```

打开网页，显示和 nginx01 一样

![](/img/Docker/Docker_8.jpg)

#### 特点
- 多个运行容器之间共享数据，多个容器可以同时挂载相同的卷。
- 当容器停止或被移除时，该卷依然存在。
- 当明确删除卷时，卷才会被删除。
- 将容器的数据存储在远程主机或其他存储上（间接）
- 将数据从一台 Docker 主机迁移到另一台时，先停止容器，然后备份卷的目录（/var/lib/docker/volumes/）

### 5.2.2 Bind Mounts

用卷创建一个容器：

- `docker run -d -it --name=nginx-test --mount type=bind,src=/app/wwwroot,dst=/usr/share/nginx/html nginx`
- `docker run -d -it --name=nginx-test -v /app/wwwroot:/usr/share/nginx/html nginx`

验证绑定：

- `docker inspect nginx-test`

清理：

- `docker stop nginx-test`
- `docker rm nginx-test`

**注意：**
1. 如果源文件/目录没有存在如果挂载目标在容器中非空目录，则该目录现有内容将被隐藏。
2. 不会自动创建，会抛出一个错误。

#### 特点
- 从主机共享配置文件到容器。默认情况下，挂载主机 `/etc/resolv.conf` 到每个容器，提供 DNS 解析。
- 在 Docker 主机上的开发环境和容器之间共享源代码。例如，可以将 Maven target 目录挂载到容器中，每次在 Docker 主机上构建 Maven 项目时，容器都可以访问构建的项目包。
- 当 Docker 主机的文件或目录结构保证与容器所需的绑定挂载一致时

## 5.3 容器网络

### 5.3.1 docker0 veth-pair 

Docker 使用的是 Linux 的桥接，宿主机中是一个 Docker 容器的网桥 docker0

![](/img/Docker/docker_14.png)

成对的虚拟设备接口，它们都是成对出现的，当运行容器后，查看网络设备：

![](/img/Docker/docker_12.jpg)

容器内地址：

![](/img/Docker/docker_13.jpg)

**Docker 中的所有的网络接口都是虚拟的，虚拟的转发效率高**，只要删除容器，对应的网桥一对就会自动删除

### 5.3.2 link




# 六、 DockerFile

## 6.1 DockerFile 是什么

Dockerfile 是用来构建 Docker 镜像的构建文件，是由一系列命令和参数构成的脚本

构建步骤：
1. 编写 Dockerfile 文件
2. `docker build`
3. `docker run`

文件结构：

CentOS Dockerfile:

```
FROM scratch
ADD centos-7-x86_64-docker.tar.xz /

LABEL \
    org.label-schema.schema-version="1.0" \
    org.label-schema.name="CentOS Base Image" \
    org.label-schema.vendor="CentOS" \
    org.label-schema.license="GPLv2" \
    org.label-schema.build-date="20200809" \
    org.opencontainers.image.title="CentOS Base Image" \
    org.opencontainers.image.vendor="CentOS" \
    org.opencontainers.image.licenses="GPL-2.0-only" \
    org.opencontainers.image.created="2020-08-09 00:00:00+01:00"

CMD ["/bin/bash"]

```

基础内容：
1. 每条保留字指令都必须为大写字母且后面要跟随至少一个参数
2. 指令按照从上到下顺序执行
3. #表示注释
4. 每条指令都会创建一个新的镜像层，并对镜像进行提交

## 6.2 Dockerfile 指令

|指令|描述|
|---|---|
|FROM|构建新镜像是基于哪个镜像|
|MAINTAINER</br>LABEL|镜像维护者姓名或邮箱地址|
|RUN|构建镜像时运行的Shell命令|
|COPY|拷贝文件或目录到镜像中|
|ENV|设置环境变量|
|USER|为RUN、CMD和ENTRYPOINT执行命令指定运行用户|
|EXPOSE|声明容器运行的服务端口|
|HEALTHCHECK|容器中服务健康检查|
|WORKDIR|为RUN、CMD、ENTRYPOINT、COPY和ADD设置工作目录|
|ENTRYPOINT|运行容器时执行，如果有多个ENTRYPOINT指令，最后一个生效|
|CMD|运行容器时执行，如果有多个CMD指令，最后一个生效|

## 6.3 Dockerfile 与 Docker容器、镜像的关系

从应用软件的角度来看，Dockerfile、Docker镜像与Docker容器分别代表软件的三个不同阶段，

- Dockerfile是软件的原材料
- Docker镜像是软件的交付品
- Docker容器则可以认为是软件的运行态。

Dockerfile面向开发，Docker镜像成为交付标准，Docker容器则涉及部署与运维，三者缺一不可，合力充当Docker体系的基石。

![](/img/Docker/Docker_9.png)

1. Dockerfile，需要定义一个Dockerfile，Dockerfile定义了进程需要的一切东西。Dockerfile涉及的内容包括执行代码或者是文件、环境变量、依赖包、运行时环境、动态链接库、操作系统的发行版、服务进程和内核进程(当应用进程需要和系统服务和内核进程打交道，这时需要考虑如何设计namespace的权限控制)等等;
2. Docker镜像，在用Dockerfile定义一个文件之后，docker build时会产生一个Docker镜像，当运行 Docker镜像时，会真正开始提供服务;
3. Docker容器，容器是直接提供服务的。

## 6.4 Build 镜像

Usage:`docker build [option] PATH |URL| -[flags]`

#### option:
- -t,--tag list ：镜像名称
- -f，--file string :指定 Dockerfile 文件位置

## 6.5 构建 Nginx 基础镜像

### 6.5.1 编写 Dockerfile 

```
FROM centos:7
MAINTAINER ecarry
#安装 nginx 依赖
RUN yum install -y gcc gcc-c++ make \
    openssl-devel pcre-devel gd-devel \
    iproute net-tools wget && \
    yum clean all && \
    rm -rf /var/cache/yum/*
#源码编译 nginx，为nginx设置安装目录和启用的模块
RUN wget http://nginx.org/download/nginx-1.19.3.tar.gz && \
    tar zxf nginx-1.19.3.tar.gz && \
    cd nginx-1.19.3 && \
    ./configure --prefix=/usr/local/nginx \
    --with-http_ssl_module \
    --with-http_stub_status_module && \
    make -j 4 && make install && \
    cd / && rm -rf nginx-* && \
    ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

ENV PATH $PATH:/usr/local/nginx/sbin
WORKDIR /usr/local/nginx
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```

**参数说明：**
- --prefix ：用于指定nginx编译后的安装目录
- --add-module ：为添加的第三方模块，此次添加了fdfs的nginx模块
- --with..._module ：表示启用的nginx模块


### 6.5.2 构建镜像

`docker build -t nginx:v1 -f dockerfile .`

- -t ：指定镜像名称
- -f ：指定编写的 dockerfile 文件
- . ：代表 dockerfile 文件里上下文

```
[root@docker ~]# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx               v1                  a4906a1a6692        39 minutes ago      335MB
```

## 6.6 构建 php 基础镜像

### 6.6.1 编写 Dockerfile 

```
FROM centos:7
MAINTAINER ecarry
RUN yum install epel-release -y && \
    yum install -y gcc gcc-c++ make gd-devel libxml2-devel \
    libcurl-devel libjpeg-devel libpng-devel openssl-devel \
    libmcrypt-devel libxslt-devel libtidy-devel autoconf \
    iproute net-tools telnet wget curl && \
    yum clean all && \
    rm -rf /var/cache/yum/*

RUN wget https://www.php.net/distributions/php-5.6.40.tar.gz && \
    tar zxf php-5.6.40.tar.gz && \
    cd php-5.6.40 && \
    ./configure --prefix=/usr/local/php \
    --with-config-file-path=/usr/local/php/etc \
    --enable-fpm --enable-opcache \
    --with-mysql --with-mysqli --with-pdo-mysql \
    --with-openssl --with-zlib --with-curl --with-gd \
    --with-jpeg-dir --with-png-dir --with-freetype-dir \
    --enable-mbstring --with-mcrypt --enable-hash && \
    make -j 4 && make install && \
    cp php.ini-production /usr/local/php/etc/php.ini && \
    cp sapi/fpm/php-fpm.conf /usr/local/php/etc/php-fpm.conf && \
    sed -i "90a \daemonize = no" /usr/local/php/etc/php-fpm.conf && \
    mkdir /usr/local/php/log && \
    cd / && rm -rf php* && \
    ln -sf /usr/share/zonginfo/Asia/Shanghai /etc/localtime

ENV PATH $PATH:/usr/local/php/sbin:/usr/local/php/bin
WORKDIR /usr/local/php
EXPOSE 9000
CMD ["php-fpm"]
```

### 6.5.2 构建镜像

`docker build -t php:v1 -f dockerfile_php .`



## 6.6 关系图

![](/img/Docker/Docker_10.png)

## 6.7 搭建 LNMP 网站平台

![](/img/Docker/Docker_11.jpg)

### 6.7.1 创建自定义网络

`docker network create lnmp`

### 6.7.2 创建 MySQL 容器

```
[root@docker ~]# docker run -d \
   --name lnmp_mysql \
   --net lnmp \
   --mount src=mysql_vol,dst=/var/lib/mysql \
   -e MYSQL_ROOT_PASSWORD=root123 \
   -e MYSQL_DATABASE=wordpress mysql:5.7 --character-set-server=utf8
```

### 6.7.3 创建 PHP 容器

```
user  nobody;
worker_processes  2;
worker_rlimit_nofile 65535;

error_log logs/error.log notice;

pid        /var/run/nginx.pid;


events {
    use epoll;
    worker_connections  4096;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    access_log  off;

    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
        

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;
    clinet_max_body_size 64m;

    #gzip  on;

    server {
        listen       80;
        server_name  www.ecarry.com;
        index index.php index.html;

        #charset koi8-r;

        access_log  logs/www.ecarry.com_access.log;
        error_log log/www.ecarry.com_error.log;

        location / {
            root   /wwwroot;
        }

        location ~* \.php$ {
            root   /wwwroot;
            fastcgi_pass lnmp_php:9000;
            fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
            include fastcgi_params;
        }
    }
}
```