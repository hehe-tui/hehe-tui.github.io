---
title: MySQL完全备份与恢复
date: 2021-1-12
updated:
description:
cover: https://img.imgdb.cn/item/5ffd6c773ffa7d37b3f157d8.jpg
tag:
  - MySQL
categories:
  - 数据库
---
## 数据库备份的分类
* 从物理与逻辑的角度，备份可以分为物理备份和逻辑备份。
（1）物理备份:对数据库操作系统的物理文件（如数据文件、日志文件等）的备份。
物理备份又可分为脱机备份（冷备份）和联机备份（热备份）。
1、冷备份:是在关闭数据库的时候进行的
2、热备份:数据库处于运行状态，这种备份方法依赖于数据库的日志文件
3、温备份:数据库锁定表格（不可写入但可读）的状态下进行的
（2）逻辑备份:对数据库逻辑组件（如表等数据库对象）的备份
* 从数据库的备份策略角度，备份可分为完全备份、差异备份和增量备份
（1）完全备份:每次对数据进行完整的备份
对整个数据库的备份、数据库结构和文件结构的备份，保存的是备份完成时刻的数据月异备份与增量备份的基础。
优点:备份与恢复操作简单方便
缺点:数据存在大量的重复；占用大量的空间；备份与恢复时间长
（2）差异备份:备份那些自从上次完全备份之后被修改过的所有文件
备份的时间节点是从上次完整备份起，备份数据量会越来越大。恢复数据时，只需恢复上次的完全备份与最近的一次差异备份。
（3）增量备份:只有那些在上次完全备份或者增量备份后被修改的文件才会被备份
以上次完整备份或上次的增量备份的时间为时间点，仅备份这之间的数据变化，因而备份的数据量小，占用空间小，备份速度快。但恢复时，需要从上一次的完整备份起到最后一次增量备份依次恢复，如中间某次的备份数据损坏，将导致数据的丢失。
##  举个栗子
### 安装环境
#### 查看linux的版本
```javascript
[root@mysql ~]# cat /etc/redhat-release
CentOS Linux release 7.8.2003 (Core)
```
#### CMake源码安装
```javascript
[root@mysql ~]# yum -y install gcc-c++ pcre pcre-devel zlib zlib-devel openssl openssl-devel mlocate bzip2 lrzsz ncurses-devel
[root@mysql ~]# wget https://github.com/Kitware/CMake/releases/download/v3.19.2/cmake-3.19.2.tar.gz
[root@mysql ~]# tar -zxvf cmake-3.19.2.tar.gz -C /usr/local/
[root@mysql ~]# cd /usr/local/cmake-3.19.2/
[root@mysql cmake-3.19.2]# ./configure
[root@mysql cmake-3.19.2]# gmake && make install
[root@mysql cmake-3.19.2]# cmake --version
cmake version 3.19.2

CMake suite maintained and supported by Kitware (kitware.com/cmake).
```
#### gcc升级
```javascript
[root@mysql ~]# yum -y install centos-release-scl devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
[root@mysql ~]# scl enable devtoolset-9 bash
[root@mysql ~]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
[root@mysql ~]# source /etc/profile
```
#### MySQL源码安装
```javascript
[root@mysql ~]# yum -y remove mari*
[root@mysql ~]# wget https://dev.mysql.com/get/Downloads/MySQL-8.0/mysql-boost-8.0.22.tar.gz
[root@mysql ~]# tar -zxvf mysql-boost-8.0.22.tar.gz -C /usr/local/
[root@mysql ~]# cd /usr/local/mysql-8.0.22/
[root@mysql mysql-8.0.22]# groupadd mysql
[root@mysql mysql-8.0.22]# useradd -r -g mysql -s /sbin/nologin mysql
[root@mysql mysql-8.0.22]# mkdir -p /usr/local/mysql
[root@mysql mysql-8.0.22]# mkdir -p /data/mysql/
[root@mysql mysql-8.0.22]# chown -R mysql.mysql /usr/local/mysql/
[root@mysql mysql-8.0.22]# chown -R mysql.mysql /data/mysql/
[root@mysql mysql-8.0.22]# chmod -R 755 /data/mysql/
[root@mysql mysql-8.0.22]# chmod -R 755 /usr/local/mysql/
[root@mysql mysql-8.0.22]# cmake . -DCMAKE_INSTALL_PREFIX=/usr/local/mysql -DMYSQL_DATADIR=/data/mysql -DSYSCONFDIR=/etc -DMYSQL_TCP_PORT=3306 -DWITH_BOOST=/usr/local/mysql-8.0.22/boost -DDEFAULT_CHARSET=utf8 -DDEFAULT_COLLATION=utf8_general_ci -DENABLED_LOCAL_INFILE=ON -DWITH_INNODB_MEMCACHED=ON -DWITH_INNOBASE_STORAGE_ENGINE=1 -DWITH_FEDERATED_STORAGE_ENGINE=1 -DWITH_BLACKHOLE_STORAGE_ENGINE=1 -DWITH_ARCHIVE_STORAGE_ENGINE=1 -DWITHOUT_EXAMPLE_STORAGE_ENGINE=1 -DWITH_PERFSCHEMA_STORAGE_ENGINE=1 -DFORCE_INSOURCE_BUILD=1
[root@mysql mysql-8.0.22]# make && make install
[root@mysql mysql-8.0.22]# cd bin/
[root@mysql bin]# ./mysqld --initialize-insecure --user=mysql --basedir=/usr/local/mysql --datadir=/data/mysql
[root@mysql bin]# vim /etc/my.cnf
[client]
port=3306
socket=/tmp/mysql.sock
#default-character-set=utf8
#user=root
#password=123456
[mysqld]
server-id=1
#skip-grant-tables
port=3306
user=mysql
max_connections=200
socket=/tmp/mysql.sock
basedir=/usr/local/mysql
datadir=/data/mysql
pid-file=/data/mysql/mysql.pid
init-connect='SET NAMES utf8'
#character-set-server=utf8
default-storage-engine=INNODB
log_error=/data/mysql/mysql-error.log
slow_query_log_file=/data/mysql/mysql-slow.log
[mysqldump]
quick
max_allowed_packet=16M
[root@mysql bin]# echo "PATH=/usr/local/mysql/bin:$PATH" >> /etc/profile
[root@mysql bin]# source /etc/profile
[root@mysql bin]# cp /usr/local/mysql-8.0.22/support-files/mysql.server /etc/init.d/mysqld
[root@mysql bin]# vim /etc/init.d/mysqld
···
basedir=/usr/local/mysql
datadir=/data/mysql
···
[root@mysql bin]# chmod +x /etc/init.d/mysqld
[root@mysql bin]# /etc/init.d/mysqld start
[root@mysql bin]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 12
Server version: 8.0.22 Source distribution

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
MySQL初步搭建完成
### 完全备份与恢复
```javascript
[root@mysql ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 15
Server version: 8.0.22 Source distribution

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> create database auth;
Query OK, 1 row affected (0.00 sec)

mysql> use auth;
Database changed
mysql> create table user(name char(10)not null,ID int(48));
Query OK, 0 rows affected, 1 warning (0.02 sec)

mysql> insert into user values('liangzai','1');
Query OK, 1 row affected (0.01 sec)

mysql> select * from user;
+----------+------+
| name     | ID   |
+----------+------+
| liangzai |    1 |
+----------+------+
1 row in set (0.00 sec)
```
####  mysqldump备份
MySQL自带的备份工具，相当方便对MySQl进行备份。通过该命令工具可以将制定的库、表或全部的库导出为SQL脚本，在需要恢复时可进行数据恢复。
* 对单个库进行完全备份
格式：mysqldump -u 用户名 -p [密码] [选项] [数据库名] > /备份路径/备份文件名
```javascript
[root@mysql ~]# mkdir backup
[root@mysql ~]# mysqldump -uroot -p auth > backup/auth-$(date +%Y%m%d).sql
[root@mysql ~]# echo $?
[root@mysql ~]# ls backup/
auth-20210112.sql
```
* 对多个库进行完全备份
格式：mysqldump -u 用户名 -p [密码] [选项] --databases库名1 [库名2] > /备份路径/备份文件名
```javascript
[root@mysql ~]# mysqldump -u root -p --databases mysql auth > backup/mysql+auth-$(date +%Y%m%d).sql
[root@mysql ~]# ls backup/
mysql+auth-20210112.sql
```
* 对所有库进行完全备份
格式: mysqldump -u 用户名 -p [密码] [选项] --all-databases 备份路径/备份文件名
```javascript
[root@mysql ~]# mysqldump -u root -p --opt --all-databases > backup/mysql_all.$(date +%Y%m%d).sql
# 加快备份速度,当备份数据量大时使用
[root@mysql ~]# ls backup/
mysql_all.20210112.sql
```
* 对表进行完全备份
格式: mysqldump -u 用户名 -p [密码] [选项] 数据库名 表名 > /备份路径/备份文件名
```javascript
[root@mysql ~]# mysqldump -u root -p auth user > backup/auth_user-$(date +%Y%m%d).sql
[root@mysql ~]# ls backup/
auth_user-20210112.sql
```
* 对表结构的备份
格式: mysqldump -u 用户名 p [密码] -d 数据名 表名 > /备份路径/备份文件名
```javascript
[root@mysql ~]# mysqldump -u root -p -d mysql user > backup/des_mysql_user-$(date +%Y%m%d).sql
[root@mysql ~]# ls backup/
des_mysql_user-20210112.sql
```
####  数据恢复
#####  source命令
执行 source 备份路径
```javascript
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| auth               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)

mysql> drop database auth;
Query OK, 1 row affected (0.02 sec)

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)

mysql> source backup/auth-20210112.sql

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| auth               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```
#####  mysql命令
mysql -u 用户名 -p [密码] < 库备份脚本的路径
```javascript
[root@mysql ~]# mysql -u root -p -e 'show databases;'
Enter password: 
+--------------------+
| Database           |
+--------------------+
| auth               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
[root@mysql ~]# mysql -u root -p -e 'drop database auth;'
Enter password: 
[root@mysql ~]# mysql -u root -p -e 'show databases;'
Enter password: 
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
[root@mysql ~]# mysql -u root -p < backup/mysql_all.20210112.sql 
Enter password: 
[root@mysql ~]# mysql -u root -p -e 'show databases;'
Enter password: 
+--------------------+
| Database           |
+--------------------+
| auth               |
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
```
mysql -u 用户名 -p [密码] 库名 <表备份脚本的路径
```javascript
[root@mysql ~]# mysql -u root -p -e 'drop table auth.user;'
Enter password: 
[root@mysql ~]# mysql -u root -p -e 'select * from auth.user;'
Enter password: 
ERROR 1146 (42S02) at line 1: Table 'auth.user' doesn't exist
[root@mysql ~]# mysql -u root -p auth < backup/auth_user-20210112.sql 
Enter password: 
[root@mysql ~]# mysql -u root -p -e 'select * from auth.user;'
Enter password: 
+----------+------+
| name     | ID   |
+----------+------+
| liangzai |    1 |
+----------+------+
```
未完待续...
