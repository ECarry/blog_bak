---
layout: post
title: 源码安装 Nginx PHP 事项
tags: [Nginx]
index_img: https://th.wallhaven.cc/small/k9/k9vygm.jpg
banner_img: https://w.wallhaven.cc/full/k9/wallhaven-k9vygm.png
categories: [Nginx]
date: 2021-02-23 09:31:08
---

# 所需依赖

## Nginx

```shell
yum install -y epel-release && \
yum install -y gc gcc gcc-c++ \
pcre-devel zlib-devel \
openssl openssl-devel zlib-devel \
libxm12 libxm12-devel bzip2-devel libmcrypt-devel \
sqlite-devel glibc glibc-devel pcre pcre-devel \
systemd-devel net-tools iotop bc zip unzip zlib-devel bash-completion \
nfs-utils automake libxslt libxslt-devel
```

## PHP

```shell
yum install epel-release -y && \
yum install -y libxm12-devel bzip2-devel \
libmcrypt-devel sqlite-devel gc gcc gcc-c++ \
glibc glibc-devel pcre pcre-devel openssl openssl-devel \
systemd-devel net-tools iotop bc zip unzip zlib-devel bash-completion \
nfs-utils automake libxm12 libxm12-devel libxslt libxslt-devel
```

# 安装配置

## Nginx
```shell
./configure --prefix=/app/nginx  --user=nginx --group=nginx --with-http_stub_status_module --with-http_ssl_module --with-file-aio --with-http_dav_module --with-stream --with-stream_ssl_module --with-stream_ssl_module --with-pcre --with-http_v2_module --with-http_gzip_static_module --with-http_sub_module
```



## PHP

```shell
./configure --prefix=/app/php --with-openssl  --with-zlib  --with-config-file-path=/ect --with-config-file-scan-dir=/etc/php.d --enable-mbstring --enable-xml --enable-sockets --enable-fpm --enable-maintainer-zts --enable-exif --disable-fileinfo --disable-mbregex --with-freetype --with-jpeg --enable-gd  --enable-gd-jis-conv
```
--with-freetype-dir --with-jpeg-dir --with-png-dir --with-libxml-dir=/usr

# Systemd service

## Nginx

`/usr/lib/systemd/system/nginx.service`

```
[Unit]
Description=nginx 1.18.0
After=network.target remote-fs.target nss-lookup.target

[Service]
Type=forking
PIDFile=/run/nginx.pid

ExecStartPre=/usr/bin/rm -f /run/nginx.pid
ExecStartPre=/app/nginx/sbin/nginx -t
ExecStart=/app/nginx/sbin/nginx
ExecReload=/bin/kill -s HUP $MAINPID
KillSignal=SIGQUIT
TimeoutStopSec=5

KillMode=process
PrivateTmp=true

[Install]
WantedBy=multi-user.target
```