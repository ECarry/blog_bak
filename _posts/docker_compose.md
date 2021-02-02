---
layout: post
title: Docker Compose
tags: [Docker]
index_img: https://th.wallhaven.cc/small/r7/r7jolj.jpg
banner_img: https://w.wallhaven.cc/full/r7/wallhaven-r7jolj.png
categories: [Docker]
date: 2021-02-02 20:35:24
---

# About 

> Compose is a tool for defining and running multi-container Docker applications. With Compose, you use a YAML file to configure your application’s services. Then, with a single command, you create and start all the services from your configuration. To learn more about all the features of Compose, see the list of features.
Compose works in all environments: production, staging, development, testing, as well as CI workflows. You can learn more about each case in Common Use Cases.
Using Compose is basically a three-step process:
1.Define your app’s environment with a Dockerfile so it can be reproduced anywhere.
2.Define the services that make up your app in docker-compose.yml so they can be run together in an isolated environment.
3.Run docker-compose up and Compose starts and runs your entire app.

Compose 是用于**定义**和**运行**多容器 Docker 应用程序的工具。通过 Compose，您可以使用 YML 文件来配置应用程序需要的所有服务。然后，使用一个命令，就可以从 YML 文件配置中创建并启动所有服务。
Compose 使用的三个步骤：
- 使用 `Dockerfile` 定义应用程序的环境。
- 使用 `docker-compose.yml` 定义构成应用程序的服务，这样它们可以在隔离环境中一起运行。
- 最后，执行 `docker-compose up` 命令来启动并运行整个应用程序。

# 一、 安装

## 1.1 linux 下安装

### 下载

`sudo curl -L "https://github.com/docker/compose/releases/download/1.28.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose`

### 赋权

`sudo chmod +x /usr/local/bin/docker-compose`

### 创建软链

`sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose`

## 一、 Docker Compose YAML 编写规则


官方实例：https://docs.docker.com/compose/compose-file/compose-file-v3/

```yaml
# 3层

version: '' #版本，对应 Docker Engine release，地址 https://docs.docker.com/compose/compose-file

services: #服务
  web:
    #服务配置
    images:
    build:
    network:
    depends_on: #依赖，启动顺序
    ...
  redis:
  mysql:

# 其他配置 网络/卷、全局规则
volume:
```

示例，使用 docker-compose 一键搭建 WordPress 博客：

```yaml
version: '3.3'

services:
  db:
    image: mysql:5.7
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: root123
      MYSQL_DATABASES: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress

  wordpress:
    depends_on:
      - db
    image: wordpress:latest
    ports:
      - "8888:80"
    restart: always
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress

volumes:
  db_data: {}
```

