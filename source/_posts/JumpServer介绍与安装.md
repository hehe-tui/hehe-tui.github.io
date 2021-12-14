---
title: JumpServer介绍与安装
date: 2021-1-25
updated:
description:
cover: https://pic.imgdb.cn/item/60a91b366ae4f77d350df72d.jpg
tag:
  - JumpServer
categories:
  - 堡垒机
---
## JumpServer简介
* JumpServer 是全球首款完全开源的堡垒机, 使用 GNU GPL v2.0 开源协议, 是符合 4A 的专业运维审计系统。
* JumpServer 使用 Python / Django 进行开发, 遵循 Web 2.0 规范, 配备了业界领先的 Web Terminal 解决方案, 交互界面美观、用户体验好。
* JumpServer 采纳分布式架构, 支持多机房跨区域部署, 中心节点提供 API, 各机房部署登录节点, 可横向扩展、无并发访问限制。
* JumpServer 现已支持管理 SSH、 Telnet、 RDP、 VNC 协议资产。
###  特点
* 开源：零门槛，线上快速获取和安装
* 分布式：轻松支持大规模并发访问
* 无插件：仅需浏览器，极致的 Web Terminal 使用体验
* 多云支持：一套系统，同时管理不同云上面的资产
* 云端存储：审计录像云端存储，永不丢失
* 多租户：一套系统，多个子公司和部门同时使用； 多应用支持: 数据库，Windows远程应用，Kubernetes

参考[JumpServer官网](https://docs.jumpserver.org/zh/master/)
## JumpServer安装
推荐使用外置 数据库 和 Redis, 方便日后扩展升级

|服务器名|操作系统|IP|
|:--------:|:--------:|:--------:|
|JumpServer | CentOS 7|192.168.0.36|
|MariaDB|CentOS 7|192.168.0.63|
|Redis|CentOS 7|192.168.0.64|

* Redis >= 5.0.0
* MySQL >= 5.7
* MariaDB >= 10.2
###  MariaDB安装
参考[MariaDB官网](https://downloads.mariadb.org/mariadb/repositories/#distro=CentOS&distro_release=centos7-amd64--centos7&mirror=tuna&version=10.5)
```javascript
[root@mariadb ~]# vim /etc/yum.repos.d/MariaDB.repo 
[mariadb]
名称= MariaDB
baseurl = http://yum.mariadb.org/10.5/centos7-amd64
gpgkey = https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
gpgcheck = 1
[root@mariadb ~]# yum install MariaDB-server MariaDB-client -y
[root@mariadb ~]# systemctl start mariadb
[root@mariadb ~]# mysql -u root -p
Enter password: 
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 10.5.8-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> alter user 'root'@'localhost' identified by 'password';
Query OK, 0 rows affected (0.001 sec)
# 设个密码
MariaDB [(none)]> create database jumpserver default charset 'utf8';
Query OK, 1 row affected (0.000 sec)

MariaDB [(none)]> grant all on jumpserver.* to 'jumpserver'@'%'identified by 'password';
Query OK, 0 rows affected (0.001 sec)

MariaDB [(none)]> flush privileges;
Query OK, 0 rows affected (0.000 sec)
```
###  Redis安装
参考[Redis官网](https://redis.io/)
之前的文章有说过Redis的安装，此次略
```javascript
[root@redis ~]# vim /usr/local/redis-6.0.9/conf/redis.conf
···
requirepass password
···
# 设置密码后无密码可以登陆，但是无法操作
```
###  JumpServer安装
```javascript
[root@jumpserver ~]# cd /opt
[root@jumpserver opt]# wget https://github.com/jumpserver/installer/releases/download/v2.7.0/jumpserver-installer-v2.7.0.tar.gz
[root@jumpserver opt]# tar -xf jumpserver-installer-v2.7.0.tar.gz
[root@jumpserver opt]# cd jumpserver-installer-v2.7.0
[root@jumpserver jumpserver-installer-v2.7.0]# export DOCKER_IMAGE_PREFIX=docker.mirrors.ustc.edu.cn
[root@jumpserver jumpserver-installer-v2.7.0]# cat config-example.txt
# 说明
#### 这是项目总的配置文件, 会作为环境变量加载到各个容器中
#### 格式必须是 KEY=VALUE 不能有空格等

# Compose项目设置
COMPOSE_PROJECT_NAME=jms
COMPOSE_HTTP_TIMEOUT=3600
DOCKER_CLIENT_TIMEOUT=3600
DOCKER_SUBNET=192.168.250.0/24

## IPV6
DOCKER_SUBNET_IPV6=2001:db8:10::/64
USE_IPV6=0

### 持久化目录, 安装启动后不能再修改, 除非移动原来的持久化到新的位置
VOLUME_DIR=/opt/jumpserver

## 是否使用外部MYSQL和REDIS
USE_EXTERNAL_MYSQL=0
USE_EXTERNAL_REDIS=0

## Nginx 配置，这个Nginx是用来分发路径到不同的服务
HTTP_PORT=8080
HTTPS_PORT=8443
SSH_PORT=2222

## LB 配置, 这个Nginx是HA时可以启动负载均衡到不同的主机
USE_LB=0
LB_HTTP_PORT=80
LB_HTTPS_PORT=443
LB_SSH_PORT=2223

## Task 配置
USE_TASK=1

## XPack
USE_XPACK=0

# Koko配置
CORE_HOST=http://core:8080
ENABLE_PROXY_PROTOCOL=true

# Core 配置
### 启动后不能再修改，否则密码等等信息无法解密
SECRET_KEY=
BOOTSTRAP_TOKEN=
LOG_LEVEL=INFO
# SESSION_COOKIE_AGE=86400
# SESSION_EXPIRE_AT_BROWSER_CLOSE=false

## MySQL数据库配置
DB_ENGINE=mysql
DB_HOST=mysql
DB_PORT=3306
DB_USER=root
DB_PASSWORD=
DB_NAME=jumpserver

## Redis配置
REDIS_HOST=redis
REDIS_PORT=6379
REDIS_PASSWORD=

### Keycloak 配置方式
### AUTH_OPENID=true
### BASE_SITE_URL=https://jumpserver.company.com/
### AUTH_OPENID_SERVER_URL=https://keycloak.company.com/auth
### AUTH_OPENID_REALM_NAME=cmp
### AUTH_OPENID_CLIENT_ID=jumpserver
### AUTH_OPENID_CLIENT_SECRET=
### AUTH_OPENID_SHARE_SESSION=true
### AUTH_OPENID_IGNORE_SSL_VERIFICATION=true

# Guacamole 配置
JUMPSERVER_SERVER=http://core:8080
JUMPSERVER_KEY_DIR=/config/guacamole/data/key/
JUMPSERVER_RECORD_PATH=/config/guacamole/data/record/
JUMPSERVER_DRIVE_PATH=/config/guacamole/data/drive/
JUMPSERVER_ENABLE_DRIVE=true
JUMPSERVER_CLEAR_DRIVE_SESSION=true
JUMPSERVER_CLEAR_DRIVE_SCHEDULE=24

# Mysql 容器配置
MYSQL_ROOT_PASSWORD=
MYSQL_DATABASE=jumpserver

[root@jumpserver jumpserver-installer-v2.7.0]# ./jmsctl.sh install
···
7. 配置 MySQL
是否使用外部 MySQL? (y/n)  (默认为 n): y
请输入 mysql 的主机地址 (无默认值): 192.168.0.63
请输入 mysql 的端口 (默认为 3306): 
请输入 mysql 的数据库 (默认为 jumpserver): jumpserver
请输入 mysql 的用户名 (无默认值): jumpserver
请输入 mysql 的密码 (无默认值): password
完成

8. 配置 Redis
是否使用外部 Redis? (y/n)  (默认为 n): y
请输入 Redis 的主机地址 (无默认值): 192.168.0.64
请输入 Redis 的端口 (默认为 6379): 6379
请输入 Redis 的密码 (无默认值): password
完成
···
```
访问 http://localhost:8080/
![](https://img.imgdb.cn/item/600e60913ffa7d37b3dfc3a6.jpg)
可以看到基本搭建成功
默认用户: admin  默认密码: admin
![](https://img.imgdb.cn/item/600e614f3ffa7d37b3e04288.jpg)
未完待续。。。
