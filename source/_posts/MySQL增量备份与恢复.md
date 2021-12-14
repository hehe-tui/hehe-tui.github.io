---
title: MySQL增量备份与恢复
date: 2021-1-17
updated:
description:
cover: https://pic.imgdb.cn/item/61b5b6892ab3f51d91841eef.jpg
tag:
  - MySQL
categories:
  - 数据库
---
##  MySQL增量备份概念
使用mseldump进行完全备份，备份的数据中有重复数据，备份时间与恢复时间长。而增量备份就是备份自上一次备份之后增加或改变的文件或内容。
###  增量备份的特点
* 没有重复数据，备份量不大，时间短
* 恢复麻烦
需要上次完全备份及完全备份之后所有的增量备份才能恢复，而且要对所有增量备份进行逐个反推恢复

MySQL没有提供直接的增量备份办法，可以通过MySQL提供的二进制日志(binary logs)间接实现增量备份。
###  MySQL二进制日志对备份的意义
* 二进制日志保存了所有更新或者可能更新数据库的操作
* 二进制日志在启动 MySQL服务器后开始记录，并在文件达到max_binlog_size 所设置的大小或者接收到 flush logs 命令后重新创建新的日志文件
* 只需定时执行 flush logs 方法重新创建新的日志，生成二进制文件序列，并及时把这些日志保存到安全的地方就完成了一个时间段的增量备份

要进行MySQL的增量备份，首先要开启二进制日志功能，开启MySQl的二进制日志功能。 
方法一：MySQL的配置文件的 [mysqld] 项中加入log-bin=文件存放路径/文件前缀，如 log-bin=mysql-bin 然后重启 mysqld 服务。默认此配置存在。
方法二：使用 mysqld--log-bin=文件存放路径/文件前缀，重启 mysqld 服务
###  增量恢复的方法
* 一般的恢复：备份的二进制日志内容全部恢复
格式：mysqlbinlog [--no-defaults] 增量备份文件 | mysql -u 用户名 -p 密码
* 基于时间点的恢复：便于跳过某个发生错误的时间点实现数据恢复
格式：
从日志开头截止到某个时间点的恢复：
mysqlbinlog [--no-defaults] --stop-datetime='年-月-日 小时:分钟:秒' 二进制日志 | mysq -u 用户名 -p 密码
从某个时间点到日志结尾的恢复：
mysqlbinlog [--no-defaults] --start-datetime='年-月-日 小时:分钟:秒' 二进制日志 | mysql -u 用户名 -p 密码
从某个时间点到某个时间点的恢复：
mysqlbinlog [--no-defaults] --start-datetime='年-月-日 小时:分钟:秒' --stop-datetime='年-月-日 小时:分钟:秒' 二进制日志 | mysql -u 用户名 -p 密码
* 基于位置的恢复：可能在同一时间点既有错误的操作也有正确的操作，基于位置进行恢复更加精准
格式：
 mysqlbinlog-stop-position='操作id' 二进制日志 | mysql -u 用户名 -p 密码
 mysqlbinlog-start-position='操作id' 二进制日志 | mysql -u 用户名 -p 密码
###  举个栗子
```javascript
[root@mysql ~]# /etc/init.d/mysqld start
Starting MySQL. SUCCESS! 
[root@mysql ~]# mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.22 Source distribution

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> 
```
####  中文无法输入问题
这里死活打不出中文，但是命令行是可以的，问题定位：
```javascript
[root@mysql bin]# vim /etc/my.cnf
[client]
···
#default-character-set=utf8
...
[mysqld]
...
#character-set-server=utf8
···
```
这俩要注释掉，重启后可以输入中文，尚不清楚原因是啥。。。
```javascript
mysql> mysql> create database client;
Query OK, 1 row affected (0.01 sec)

mysql> mysql> use client;
Database changed
mysql> create table user_info(身份证 char(20)not null,姓名 char(20)not null,性别 char(4),用户ID char(10) not null);
Query OK, 0 rows affected (0.02 sec)

mysql> insert into user_info values('001','靓仔','男','01');
Query OK, 1 row affected (0.01 sec)

mysql> insert into user_info values('002','帅哥','男','02');
Query OK, 1 row affected (0.00 sec)

mysql> insert into user_info values('003','妹子','女','03');
Query OK, 1 row affected (0.00 sec)

mysql> select * from user_info;
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
+-----------+--------+--------+----------+
3 rows in set (0.00 sec)
```
#### 先进行一次完全备份
```javascript
[root@mysql ~]# mkdir back
[root@mysql ~]# mysqldump -uroot -p client user_info > back/client_user_info-$(date +%F).sql
Enter password: 
[root@mysql ~]# mysqldump -uroot -p client > back/client-$(date +%F).sql
Enter password: 
[root@mysql ~]# ls back/
client-2021-01-17.sql  client_user_info-2021-01-17.sql
```
#### 进行一次日志回滚，生成新的二进制日志
```javascript
mysql> show master logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       156 | No        |
| binlog.000002 |       847 | No        |
| binlog.000003 |       360 | No        |
| binlog.000004 |       179 | No        |
| binlog.000005 |       767 | No        |
| binlog.000006 |       156 | No        |
| binlog.000007 |      1370 | No        |
+---------------+-----------+-----------+
7 rows in set (0.00 sec)

mysql> flush logs;
Query OK, 0 rows affected (0.01 sec)

mysql> show master logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       156 | No        |
| binlog.000002 |       847 | No        |
| binlog.000003 |       360 | No        |
| binlog.000004 |       179 | No        |
| binlog.000005 |       767 | No        |
| binlog.000006 |       156 | No        |
| binlog.000007 |      1414 | No        |
| binlog.000008 |       156 | No        |
+---------------+-----------+-----------+
8 rows in set (0.00 sec)
```
#### 继续录入新的数据
```javascript
mysql> use client;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> insert into user_info values('004','男票','女','04');
Query OK, 1 row affected (0.01 sec)

mysql> insert into user_info values('005','女票','男','05');
Query OK, 1 row affected (0.00 sec)

mysql> select * from user_info;
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 004       | 男票   | 女     | 04       |
| 005       | 女票   | 男     | 05       |
+-----------+--------+--------+----------+
5 rows in set (0.00 sec)
```
#### 进行增量备份
```javascript
mysql> flush logs;
Query OK, 0 rows affected (0.02 sec)

mysql> show master logs;
+---------------+-----------+-----------+
| Log_name      | File_size | Encrypted |
+---------------+-----------+-----------+
| binlog.000001 |       156 | No        |
| binlog.000002 |       847 | No        |
| binlog.000003 |       360 | No        |
| binlog.000004 |       179 | No        |
| binlog.000005 |       767 | No        |
| binlog.000006 |       156 | No        |
| binlog.000007 |      1414 | No        |
| binlog.000008 |       818 | No        |
| binlog.000009 |       156 | No        |
+---------------+-----------+-----------+
9 rows in set (0.00 sec)

mysql> mysql> show binlog events in 'binlog.000009';
# 查看新操作的日志记录
```
#### 模拟删除误操作
```javascript
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 004       | 男票   | 女     | 04       |
| 005       | 女票   | 男     | 05       |
+-----------+--------+--------+----------+
mysql> drop table client.user_info;
Query OK, 0 rows affected (0.01 sec)

mysql> select * from user_info;
ERROR 1146 (42S02): Table 'client.user_info' doesn't exist
```
####  恢复完全备份
```javascript
[root@mysql ~]# mysql -uroot -p client < back/client_user_info-2021-01-17.sql 
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
+-----------+--------+--------+----------+
```
####  恢复增量备份
```javascript
[root@mysql ~]# mysqlbinlog /data/mysql/binlog.000008 | mysql -uroot -p 
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 004       | 男票   | 女     | 04       |
| 005       | 女票   | 男     | 05       |
+-----------+--------+--------+----------+
```
9是备份后的，8才是真正备份的操作，我说咋没恢复过来。。。
#####  基于时间点的增量备份操作
```javascript
[root@mysql ~]# mysql -uroot -p -e 'drop table client.user_info;'
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
ERROR 1146 (42S02) at line 1: Table 'client.user_info' doesn't exist
[root@mysql ~]# mysql -uroot -p client < back/client_user_info-2021-01-17.sql 
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
+-----------+--------+--------+----------+
[root@mysql ~]# mysqlbinlog --no-defaults /data/mysql/binlog.000008 
# 查看日志操作记录
```
######  仅恢复男票的数据
```javascript
[root@mysql ~]# mysqlbinlog --no-defaults --stop-datetime='2021-01-17 15:59:43' /data/mysql/binlog.000008|mysql -uroot -p

Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 004       | 男票   | 女     | 04       |
+-----------+--------+--------+----------+
```
######  仅恢复女票的数据
```javascript
[root@mysql ~]# mysqlbinlog --no-defaults --start-datetime='2021-01-17 15:59:43 ' /data/mysql/binlog.000008|mysql -uroot -p
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 005       | 女票   | 男     | 05       |
+-----------+--------+--------+----------+
```
#####  基于位置的增量备份操作
```javascript
[root@mysql ~]# mysql -uroot -p -e 'drop table client.user_info;'
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
ERROR 1146 (42S02) at line 1: Table 'client.user_info' doesn't exist
[root@mysql ~]# mysql -uroot -p client < back/client_user_info-2021-01-17.sql 
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
+-----------+--------+--------+----------+
```
######  仅恢复男票的数据
```javascript
[root@mysql ~]# mysqlbinlog --no-defaults --stop-position='465' /data/mysql/binlog.000008|mysql -uroot -p
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 004       | 男票   | 女     | 04       |
+-----------+--------+--------+----------+
```
######  仅恢复女票的数据
```javascript
[root@mysql ~]# mysqlbinlog --no-defaults --start-position='434' /data/mysql/binlog.000008|mysql -uroot -p
Enter password: 
[root@mysql ~]# mysql -uroot -p -e 'select * from client.user_info;'
Enter password: 
+-----------+--------+--------+----------+
| 身份证    | 姓名   | 性别   | 用户ID   |
+-----------+--------+--------+----------+
| 001       | 靓仔   | 男     | 01       |
| 002       | 帅哥   | 男     | 02       |
| 003       | 妹子   | 女     | 03       |
| 005       | 女票   | 男     | 05       |
+-----------+--------+--------+----------+
```
未完待续。。。
