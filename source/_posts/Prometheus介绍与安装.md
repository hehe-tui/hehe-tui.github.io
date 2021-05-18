---
title: Prometheus介绍与安装
date: 2021-1-28
updated:
description:
cover: https://img.imgdb.cn/item/601267593ffa7d37b3b746d1.jpg
tag:
  - Prometheus
categories:
  - 监控
---
## Prometheus简介
Prometheus是最初在SoundCloud上构建的开源系统监视和警报工具包 。自2012年成立以来，许多公司和组织都采用了Prometheus，该项目拥有非常活跃的开发人员和用户社区。现在，它是一个独立的开源项目，并且独立于任何公司进行维护。为了强调这一点并阐明项目的治理结构，Prometheus在2016年加入了 Cloud Native Computing Foundation，这是继Kubernetes之后的第二个托管项目。
###  特点
* 一个多维数据模型，其中包含通过度量标准名称和键/值对标识的时间序列数据
* PromQL，一种灵活的查询语言 ，可利用此维度
* 不依赖分布式存储；单服务器节点是自治的
* 时间序列收集通过HTTP上的拉模型进行
* 通过中间网关支持推送时间序列
* 通过服务发现或静态配置发现目标
* 多种图形和仪表板支持模式
###  体系
![](https://img.imgdb.cn/item/601264c73ffa7d37b3b5a3b6.jpg)
## Prometheus安装
### GO安装
[GO官网](https://golang.google.cn/)
```javascript
[root@prometheus ~]# wget https://golang.google.cn/dl/go1.15.7.linux-amd64.tar.gz
[root@prometheus ~]# tar -zxvf go1.15.7.linux-amd64.tar.gz -C /usr/local/
[root@prometheus ~]# vim /etc/profile
···
export PATH=$PATH:/usr/local/go/bin
···
# 配置环境变量
[root@prometheus ~]# source /etc/profile
[root@prometheus ~]# go version
go version go1.15.7 linux/amd64
# 查看go版本
```
### Prometheus安装
[Prometheus官网](https://prometheus.io/)
```javascript
[root@prometheus ~]# wget https://github.com/prometheus/prometheus/releases/download/v2.24.1/prometheus-2.24.1.linux-amd64.tar.gz
[root@prometheus ~]# tar -zxvf prometheus-2.24.1.linux-amd64.tar.gz -C /usr/local/
[root@prometheus ~]# ln -sv /usr/local/prometheus-2.24.1.linux-amd64/ /usr/local/Prometheus
‘/usr/local/Prometheus’ -> ‘/usr/local/prometheus-2.24.1.linux-amd64/’
[root@prometheus ~]# /usr/local/Prometheus/prometheus --config.file=/usr/local/Prometheus/prometheus.yml &
# 启动promethus服务
```
打开 http://localhost:9090/ ,可查看普罗米修斯自带的监控页面
![](https://img.imgdb.cn/item/60114bb03ffa7d37b32a1a85.png)
### Grafana安装
[Grafana官网](https://grafana.com/)
```javascript
[root@prometheus ~]# wget https://dl.grafana.com/oss/release/grafana-7.3.7.linux-amd64.tar.gz
[root@prometheus ~]# tar -zxvf grafana-7.3.7.linux-amd64.tar.gz -C /usr/local/
[root@prometheus ~]# cd /usr/local/grafana-7.3.7/
[root@prometheus grafana-7.3.7]# ./bin/grafana-server web &
```
打开 http://localhost:3000/ ,验证是否成功
![](https://img.imgdb.cn/item/60114be03ffa7d37b32a2e27.png)
账号：admin
密码：admin
#### 添加prometheus数据源
点击界面的 "Add data source" 
![](https://img.imgdb.cn/item/60114ff33ffa7d37b32c3ec9.jpg)
选择Prometheus
![](https://img.imgdb.cn/item/6011501a3ffa7d37b32c55ba.jpg)
Dashboards页面选择 "Prometheus 2.0 Stats"
![](https://img.imgdb.cn/item/6011515a3ffa7d37b32cfe2c.jpg)
Settings页面填写 prometheus 地址并保存
![](https://img.imgdb.cn/item/601150863ffa7d37b32c9247.jpg)
切换到 "Prometheus 2.0 Stats" 查看整个监控页面
![](https://img.imgdb.cn/item/601151a23ffa7d37b32d1fb8.jpg)
### 添加常用监控项
####  MariaDB监控
```javascript
[root@prometheus ~]# vim /usr/local/Prometheus/prometheus.yml
···
# MySQL监控
  - job_name: 'MariaDB'
    static_configs:
    - targets: ['192.168.0.63:9104']
# 默认mysqld-exporter端口为9104
···
[root@prometheus ~]# pkill prometheus
[root@prometheus ~]# /usr/local/Prometheus/prometheus --config.file=/usr/local/Prometheus/prometheus.yml &
# 重启下 Prometheus 
```
MariaDB 安装方法此处略
```javascript
[root@mariadb ~]# wget https://github.com/prometheus/mysqld_exporter/releases/download/v0.12.1/mysqld_exporter-0.12.1.linux-amd64.tar.gz
[root@mariadb ~]# tar -zxvf mysqld_exporter-0.12.1.linux-amd64.tar.gz -C /usr/local/
[root@mariadb ~]# cd /usr/local/mysqld_exporter-0.12.1.linux-amd64/
[root@mariadb mysqld_exporter-0.12.1.linux-amd64]# vim .my.cnf
[client]
user=root
password=password
# 设置配置文件，user为数据库登录用户，password为这个用户的密码
[root@mariadb mysqld_exporter-0.12.1.linux-amd64]# /usr/local/mysqld_exporter-0.12.1.linux-amd64/mysqld_exporter --config.my-cnf="/usr/local/mysqld_exporter-0.12.1.linux-amd64/.my.cnf" &
[1] 24834
INFO[0000] Starting mysqld_exporter (version=0.12.1, branch=HEAD, revision=48667bf7c3b438b5e93b259f3d17b70a7c9aff96)  source="mysqld_exporter.go:257"
INFO[0000] Build context (go=go1.12.7, user=root@0b3e56a7bc0a, date=20190729-12:35:58)  source="mysqld_exporter.go:258"
INFO[0000] Enabled scrapers:                             source="mysqld_exporter.go:269"
INFO[0000]  --collect.global_status                      source="mysqld_exporter.go:273"
INFO[0000]  --collect.global_variables                   source="mysqld_exporter.go:273"
INFO[0000]  --collect.slave_status                       source="mysqld_exporter.go:273"
INFO[0000]  --collect.info_schema.innodb_cmp             source="mysqld_exporter.go:273"
INFO[0000]  --collect.info_schema.innodb_cmpmem          source="mysqld_exporter.go:273"
INFO[0000]  --collect.info_schema.query_response_time    source="mysqld_exporter.go:273"
INFO[0000] Listening on :9104                            source="mysqld_exporter.go:283"
# 启动mysqld-exporter
[root@mariadb ~]# mysql -uroot -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 28834
Server version: 10.5.8-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> grant all on jumpserver.* to 'root'@'%'identified by 'password';   
Query OK, 0 rows affected (0.001 sec)
```
访问 http://localhost:9090/ 
![](https://img.imgdb.cn/item/60124f963ffa7d37b3a85b8c.jpg)
Grafana 添加MariaDB数据源
![](https://img.imgdb.cn/item/601152e33ffa7d37b32dc15f.jpg)
添加需要被监控的数据库及相关信息
![](https://img.imgdb.cn/item/601153503ffa7d37b32dfd0b.jpg)
查看监控
![](https://img.imgdb.cn/item/601251be3ffa7d37b3a96852.jpg)
####  Redis监控
[redis_exporter](https://github.com/oliver006/redis_exporter)
```javascript
[root@redis ~]# git clone https://github.com/oliver006/redis_exporter.git
[root@redis ~]# cd redis_exporter
[root@redis redis_exporter]# go build .
go: github.com/gomodule/redigo@v1.8.3: Get "https://proxy.golang.org/github.com/gomodule/redigo/@v/v1.8.3.mod": dial tcp 172.217.27.145:443: i/o timeout
# 这个异常的原因是因为某些特殊原因, 我们无法下载墙外的依赖, 所以我们需要去代理服务器进行下载
[root@redis redis_exporter]# vim /etc/profile
···
export GOPROXY=https://goproxy.io/
[root@redis redis_exporter]# go build .
[root@redis redis_exporter]# ./redis_exporter --version
INFO[0000] Redis Metrics Exporter <<< filled in by build >>>    build date: <<< filled in by build >>>    sha1: <<< filled in by build >>>    Go: go1.15.7    GOOS: linux    GOARCH: amd64 
[root@redis redis_exporter]# ./redis_exporter -redis.addr 192.168.0.64:6379 -redis.password "password" & 
```
```javascript
[root@prometheus ~]# vim /usr/local/Prometheus/prometheus.yml
···
# Redis监控
  - job_name: 'Redis'
    static_configs:
    - targets: ['192.168.0.64:9121']               
# 默认redis-exporter端口为9121
···
[root@prometheus ~]# pkill prometheus
[root@prometheus ~]# /usr/local/Prometheus/prometheus --config.file=/usr/local/Prometheus/prometheus.yml &
# 重启下 Prometheus 
```
访问 http://localhost:9090/ 
![](https://img.imgdb.cn/item/601262b23ffa7d37b3b414b3.jpg)
访问 http://localhost:3000/ 
![](https://img.imgdb.cn/item/601263153ffa7d37b3b46cd4.jpg)
未完待续。。。
