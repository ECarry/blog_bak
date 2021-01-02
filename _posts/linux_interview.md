---
layout: post
title: Linux 运维工程师面试总结
tags: [Linux]
sticky: 100
index_img: https://th.wallhaven.cc/small/d5/d5gexo.jpg
banner_img: https://w.wallhaven.cc/full/d5/wallhaven-d5gexo.png
categories: [Linux]
date: 2020-11-22 20:41:42
---

| 文章名称 | linux 运维工程师面试题总结 |
|---|---|
| 作者信息 | https://www.cuiliangblog.cn/blog/about/ |
| 技术博客 | https://www.cuiliangblog.cn/ |
| 文章链接 | https://www.cuiliangblog.cn/blog/show-124/ |

# 一、 linux

1. 系统启动流程
2. linux 文件类型
3. centos6 和 7 怎么添加程序开机自启动？
4. 如何升级内核，目前最新版本号多少？
5. nginx 日志访问量前十的ip怎么统计？
6. 删除 /var/log/ 下 .log 结尾的30天前的日志文件
7. ansible 有哪些模块？功能是什么？
8. nginx 性能为什么比 apache 高？
9. 四层负载和七层负载区别是什么？
10. lvs 有哪些工作模式？哪个性能高？
11. lvs nginx haproxy keeplived 区别，优缺点？
12. 如下 url 地址，各个部分的含义
https://www.baidu.com/s?word=123&ie=utf-8
13. tomcat 各个目录含义，如何修改端口，如何修改内存数？
14. nginx 反向代理时，如何使后端获取真正的访问来源 ip？
15. nginx 负载均衡算法有哪些？
16. 如何进行压力测试？
17. curl 命令如何发送 https 请求？如何查看 response 头信息？如何发送 get 和 post 表单信息？

# 二、 mysql

1. 索引的为什么使查询加快？有啥缺点？
2. sql 语句左外连接 右外连接 内连接 全连接区别
3. mysql 数据备份方式，如何恢复？你们的备份策略是什么？
4. 如何配置数据库主从同步，实际工作中是否遇到数据不一致问题？如何解决？
5. mysql 约束有哪些？
6. 二进制日志（binlog）用途？
7. mysql 数据引擎有哪些？
8. 如何查询 mysql 数据库存放路径？
9. mysql 数据库文件后缀名有哪些？用途什么？
10. 如何修改数据库用户的密码？
11. 如何修改用户权限？如何查看？

# 三、 nosql

1. redis 数据持久化有哪些方式？
2. redis 集群方案有哪些？
3. redis 如何进行数据备份与恢复？
4. MongoDB 如何进行数据备份？
5. kafka 为何比 redis rabbitmq 快？

# 四、 docker

1. dockerfile 有哪些关键字？用途是什么？
2. 如何减小 dockerfile 生成镜像体积？
3. dockerfile 中 CMD 与 ENTRYPOINT 区别是什么？
4. dockerfile 中 COPY 和 ADD 区别是什么？
5. docker 的 cs 架构组件有哪些？
6. docker 网络类型有哪些？
7. 如何配置 docker 远程访问？
8. docker 核 心namespace CGroups 联合文件系统功能是什么？
9. 命令相关：导入导出镜像，进入容器，设置重启容器策略，查看镜像环境变量，查看容器占用资源
10. 构建镜像有哪些方式？
11. docker 和 vmware 虚拟化区别？

# 五、 kubernetes

1. k8s 的集群组件有哪些？功能是什么？
2. kubectl 命令相关：如何修改副本数，如何滚动更新和回滚，如何查看 pod 的详细信息，如何进入 pod 交互？
3. etcd 数据如何备份？
4. k8s 控制器有哪些？
5. 哪些是集群级别的资源？
6. pod 状态有哪些？
7. pod 创建过程是什么？
8. pod 重启策略有哪些？
9. 资源探针有哪些？
10. requests 和 limits 用途是什么？
11. kubeconfig 文件包含什么内容，用途是什么？
12. RBAC中role 和 clusterrole 区别，rolebinding 和 clusterrolebinding 区别？
13. ipvs 为啥比 iptables 效率高？
14. sc pv pvc 用途，容器挂载存储整个流程是什么？
15. nginx ingress 的原理本质是什么？
16. 网络类型，描述不同 node 上的 Pod 之间的通信流程
17. k8s 集群节点需要关机维护，需要怎么操作

# 六、 prometheus

1. prometheus 对比 zabbix 有哪些优势？
2. prometheus 组件有哪些，功能是什么？
3. 指标类型有哪些？
4. 在应对上千节点监控时，如何保障性能
（降低采集频率，缩小历史数据保存天数，使用集群联邦和远程存储）
5. 简述从添加节点监控到 grafana 成图的整个流程
6. 在工作中用到了哪些 exporter

# 七、 ELK

1. Elasticsearch 的数据如何备份与恢复？
2. 你们项目中使用的 logstash 过滤器插件是什么？实现哪些功能？
3. 是否用到了 filebeat 的内置 module？用了哪些？
4. kibana 如何自定义图表和仪表盘？
5. elasticsearch 分片副本是什么？你们配置的参数是多少？

# 八、 运维开发

1. 备份系统中所有镜像
2. 编写脚本，定时备份某个库，然后压缩，发送异机
（注意：①公共部分定义函数，如获取时间戳，配置报警接口②异常处理，如数据库大，检测任务是否完成。检测生成文件大小是否是空文件）
3. 批量获取所有主机的系统信息
4. django 的 mtv 模式流程
5. python 如何导出、导入环境依赖包
6. python 创建，进入，退出，查看虚拟环境
7. flask 和 django 区别，应用场景
8. flask开发一个 hello word 页面流程
9. 列举常用的 git 命令
10. git gitlab jenkins 的 CICD 流程如何配置

# 九、 日常工作

1. 在日常工作中遇到了什么棘手的问题，如何排查
（① redis 弱口令导致中挖矿病毒，排查，优化 ② k8s 中开发的程序在用户上传文件时开启进程，未及时关闭，导致节点超出最大进程数）
2. 日常故障处理流程
3. 修改线上业务配置文件流程
4. 业务 pv 多少？集群规模多少？怎么保障业务高可用？

# 十、 开放性问题

1. 你认为初级运维工程师和高级运维工程师的区别？（初级干活的，会操作，顺利完成领导安排的任务。高级优化架构，研究如何避免问题，研究新技术并引用）
2. 你认为未来运维发展方向（自动化，智能化）