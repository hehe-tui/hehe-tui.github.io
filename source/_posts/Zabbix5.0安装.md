---
title: Zabbix5.0安装
date: 2020-12-5
updated:
description: 
cover: https://pic.downk.cc/item/5fc1f80b15e7719084aa672f.jpg
tag:
  - Zabbix
categories:
  - 监控
---
详细安装说明详见[zabbix官网](https://www.zabbix.com/cn)
官网上有yum安装的方式，如果是学习使用可以使用yum方式，方便简单。建议在生产环境使用源码方式安装。以下，演示为yum方式安装zabbix服务端与客户端。
### 准备LAMP
```javascript
[root@jjh ~]# yum -y install httpd mariadb-server mariadb php
[root@jjh ~]# systemctl start httpd.service mariadb.service
[root@jjh ~]# systemctl enable httpd.service mariadb.service
```
### 安装数据库
```javascript
[root@jjh ~]# rpm -Uvh https://repo.zabbix.com/zabbix/5.0/rhel/7/x86_64/zabbix-release-5.0-1.el7.noarch.rpm
[root@jjh ~]# yum clean all
```
### 安装Zabbix服务器和代理
```javascript
[root@jjh ~]# yum install zabbix-server-mysql zabbix-agent -y
```
### 安装Zabbix前端
```javascript
[root@jjh ~]# yum install centos-release-scl -y
```
### 修改配置文件
```javascript
[root@jjh ~]# vim /etc/yum.repos.d/zabbix.repo 
[zabbix-frontend]
...
enabled=1
...
```
### 安装Zabbix前端软件包
```javascript
[root@jjh ~]# yum install zabbix-web-mysql-scl zabbix-apache-conf-scl -y
```
### 创建数据库并导入初始数据
```javascript
[root@jjh ~]# mysql_secure_installation #初始化数据库信息
[root@jjh ~]# mysql -uroot -p
MariaDB [(none)]> create database zabbix character set utf8 collate utf8_bin;
MariaDB [(none)]> grant all privileges on zabbix.* to zabbix@localhost identified by 'zabbix';
MariaDB [(none)]> quit;
```
### 导入初始架构和数据
```javascript
[root@jjh ~]# zcat /usr/share/doc/zabbix-server-mysql-*/create.sql.gz |mysql -uzabbix -pzabbix zabbix
```
### 查看和编辑配置文件
```javascript
[root@jjh ~]# vim /etc/zabbix/zabbix_server.conf 
DBName=zabbix
DBUser=zabbix
DBPassword=zabbix	#添加此行
AlertScriptsPath=/usr/lib/zabbix/alertscripts	#自定义动作脚本存放路径
[root@jjh zabbix]# vim /etc/opt/rh/rh-php72/php-fpm.d/zabbix.conf
php_value[date.timezone] = Asia/Shanghai
```
### 启动服务
```javascript
[root@jjh zabbix]# systemctl restart zabbix-server zabbix-agent httpd rh-php72-php-fpm
[root@jjh zabbix]# systemctl enable zabbix-server zabbix-agent httpd rh-php72-php-fpm 
```
### web界面安装
浏览器访问：http://localhost/zabbix
数据库密码：zabbix
![](https://pic.downk.cc/item/5fcb9026394ac52378a1cfc8.png)
用户名：Admin
密码：zabbix
![](https://pic.downk.cc/item/5fcc725b394ac52378375174.png)
如果主机可用性为未知，可以查看下日志
可能是数据库密码错误导致没连上，在配置文件中修改下密码就OK了
