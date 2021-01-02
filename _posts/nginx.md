---
layout: post
title: Nginx 简单配置
tags: [Nginx]
index_img: https://th.wallhaven.cc/small/k9/k9vygm.jpg
banner_img: https://w.wallhaven.cc/full/k9/wallhaven-k9vygm.png
categories: [Nginx]
date: 2020-10-21 09:59:00
---

# 一、 基础概念

Nginx 是一个高性能的 **HTTP** 和**反向代理服务器**，特点是占用**内存少**，**并发能力强**，事实上nginx的并发能力确实在同类型的网页服务器中表现较好。

Nginx 相对于 Apache 优点：
- 高并发响应性能非常好，官方 Nginx 处理静态文件并发 5w/s
- 负载均衡及反向代理性能非常强
- 系统内存和 CPU 占用率低
- 可以对后端服务进行将康检查
- 支持 PHP cgi 方式和 FastCGI 方式
- 可作为缓存服务器、邮件代理服务器
- 配置代码简洁且容易上手

## 1.1 静态页面

**静态页面**通常指不与数据库发生交互的页面，是一种基于 w3c 规范的一种网页书写格式，是一种统一协议语言，所以称之为静态网页。静态页面被设计好之后，一般很少去修改，不随着浏览器参数改变而内容改变，需注意的是动态的图片也是属于静态文件。从 SEO 角度来讲，HTML 页面更有利于搜索引擎的爬行和收录。常见的静态页面以 .html、 .gif、 .jpg、 .jpeg、 .bmp、 .png、 .ico、 .txt、 .js、 .css 等结尾。

## 1.2 动态页面

**动态页面**通常指与数据库发生交互的页面，内容展示丰富，功能非常强大，实用性广。从 SEO 角度来讲，搜索引擎很难全面的爬行和收录动态网页，因为动态网页会随着数据库的更新、参数的变更而发生改变，常见的动态页面以 .jsp、.php、.do、.asp、.cgi、.apsx 等结尾。

# 二、 工作原理

Nginx WEB 服务器最主要的就是各个模块的工作，模块从结构上分为**核心模块**、**基础模块**和**第三方模块**，其中三类模块分别为：
- 核心模块：**HTTP** 模块、**EVENT** 模块和 **MAIL** 模块等
- 基础模块：HTTP Access 模块、HTTP FastCGI 模块、HTTP Proxy 模块和 HTTP Rewrite 模块
- 第三方模块：HTTP Upstream Request Hash 模块、Notice 模块、HTTP Access Key 模块、Limit_req 模块和 Upstream check module 等

Nginx 的模块从功能上分为：
- <font color=red>Handlers（处理器模块）：此类模块直接处理请求，并进行输出内容和修改 headers 信息等操作，Handlers 处理器模块一般只能有一个</font>
- <font color=red>Filters（过滤器模块）：此类模块主要是对其他模块输出的内容进行修改操作，最后有 Nginx 输出</font>
- Proxys（代理类模块）：此类模块是 Nginx 的 HTTP Upstream 之类的模块，这些模块主要与后端一些服务比如 FastCGI 等进行交互，实现服务代理和负载均衡等功能

```
# user  nobody;
worker_processes  1;

# error_log  logs/error.log;
# error_log  logs/error.log  notice;
# error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    # log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                   '$status $body_bytes_sent "$http_referer" '
    #                   '"$http_user_agent" "$http_x_forwarded_for"';

    # access_log  logs/access.log  main;

    sendfile        on;
    # tcp_nopush     on;

    # keepalive_timeout  0;
    keepalive_timeout  65;

    # gzip  on;

    server {
        listen       80;
        server_name  localhost;

        # charset koi8-r;

        # access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        # error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
```

## 2.1 Nginx 运行原理

Nginx  由 **Nginx 内核**和**模块**组成，其中内核的设计非常微小和简洁，完成的工作也非常简单，仅仅通过查找配置文件将客户端的请求映射到一个 <font color=red>location block</font>，而 location 是 Nginx 配置中的一个指令，用于访问的 URL 匹配，而在这个 location 中所配置的每个指令将会启动不同的模块去完成相应的工作。

**location block**
```
#访问根 /
location / {
            root   html;            # 以 /usr/local/nginx/ 为基础的目录
            index  index.html index.htm;
        }
```

根页面：

![](/img/nginx/nginx_1.jpg)

```
# = 精确匹配
location = /50x.html {
            root   html; #以/usr/local/nginx/为基础的目录
        }
```

精准匹配 50x 页面：
![](/img/nginx/nginx_1.jpg)




echo 'export PATH=/usr/local/node/bin:$PATH' >> /etc/profile