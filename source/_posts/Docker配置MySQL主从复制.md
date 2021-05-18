---
title: Docker配置MySQL主从复制
date: 2020-12-11
updated:
description:
cover: https://img.imgdb.cn/item/60028b753ffa7d37b3d4b92a.jpg
tag:
  - Docker 
  - MySQL
categories:
  - 数据库
---
## MySQL主从复制工作原理
1.MySql主库在事务提交时会把数据变更作为事件记录在二进制日志Binlog中；
2.主库推送二进制日志文件Binlog中的事件到从库的中继日志Relay Log中，之后从库根据中继日志重做数据变更操作，通过逻辑复制来达到主库和从库的数据一致性；
3.MySql通过三个线程来完成主从库间的数据复制，其中Binlog Dump线程跑在主库上，I/O线程和SQL线程跑着从库上；
4.当在从库上启动复制时，首先创建I/O线程连接主库，主库随后创建Binlog Dump线程读取数据库事件并发送给I/O线程，I/O线程获取到事件数据后更新到从库的中继日志Relay Log中去，之后从库上的SQL线程读取中继日志Relay Log中更新的数据库事件并应用
### 复制类型
#### 基于语句的复制
MySQL中的复制功能最初基于从主设备到从设备的SQl语句传播。这称为基于语句的日志记录。您可以通过启动服务器来使用此格式--binlog_format=STATEMENT

#### 基于行的复制
在基于行的日志记录中，主服务器将事件写入二进制日志，以指示各个表行的影响方式。因此，重要的是表始终使用主键来确保可以有效地识别行。您可以通过启动它来使服务器使用基于行的日志记录 --binlog_format=ROW

#### 混合类型的复制
混合日志记录。对于混合日志记录，默认情况下使用基于语句的日志记录，但在某些情况下，日志记录模式会自动切换到基于行的日志，您可以通过使用该选项启动mysqld,使MySQL显式使用混合日志记录--binlog_format=MIXED

#### 基于GTIDS复制
GTID即全局事务ID (global transaction identifier)，实际上是由UUID+TID (即transactionId)组成的。其中UUID(即server_uuid) 产生于auto.conf文件(cat /data/mysql/data/auto.cnf)，是一个MySQL实例的唯一标识。TID代表了该实例上已经提交的事务数量，并且随着事务提交单调递增，所以GTID能够保证每个MySQL实例事务的执行（不会重复执行同一个事务，并且会补全没有执行的事务）。GTID在一组复制中，全局唯一。
使用GTID需要注意: 在构建主从复制之前，在一台将成为主的实例上进行一些操作（如数据清理等），通过GTID复制，这些在主从成立之前的操作也会被复制到从服务器上，引起复制失败。也就是说通过GTID复制都是从最先开始的事务日志开始，即使这些操作在复制之前执行。

## 案例
### 拉取镜像
```javascript
[root@jjh ~]# docker pull docker 
[root@jjh ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
docker.io/mysql     latest              dd7265748b5d        2 weeks ago         545 MB
```
### 创建主从文件夹及修改相关配置
```javascript
[root@jjh ~]# mkdir -p /docker/mysql/{master,slave}/data
```
#### 添加主库配置文件
```javascript
[root@jjh ~]# cat /docker/mysql/master/my.cnf
[mysqld]
user=mysql
log-bin=mysql-bin
## 开启二进制日志功能
server-id=1
## 同一局域网内注意要唯一
character-set-server=utf8mb4
default_authentication_plugin=mysql_native_password
## MySQL8.O 更改默认的身份认证插件
table_definition_cache=400
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
```
#### 添加从库配置文件
```javascript
[root@jjh ~]# cat /docker/mysql/slave/my.cnf
[mysqld]                                                                             
user=mysql
server-id=2
character-set-server=utf8mb4
default_authentication_plugin=mysql_native_password
table_definition_cache=400
[client]
default-character-set=utf8mb4
[mysql]
default-character-set=utf8mb4
```
### 创建Mysql桥接网络，用于主从容器直接互联
```javascript
[root@jjh ~]# docker network create mysql
```
#### 创建Ｍysql主库容器
```javascript
[root@jjh ~]# docker run -d --privileged=true -p 3307:3306 -v /docker/mysql/master/my.cnf:/etc/mysql/my.cnf -v /docker/mysql/master/data:/var/lib/mysql -v /docker/mysql/master/mysql-files:/var/lib/mysql-files -e MYSQL_ROOT_PASSWORD=密码 --name mysql-master --network mysql --network-alias mysql-master docker.io/mysql 
```
#### 创建Ｍysql从库容器
```javascript
[root@jjh ~]# docker run -d --privileged=true -p 3308:3306 -v /docker/mysql/slave/my.cnf:/etc/mysql/my.cnf -v /docker/mysql/slave1/data:/var/lib/mysql -v /docker/mysql/slave/mysql-files:/var/lib/mysql-files -e MYSQL_ROOT_PASSWORD=密码 --name mysql-slave --network mysql --network-alias mysql-slave docker.io/mysql 
```
### 配置Mysql主从复制
#### 配置主服务器
```javascript
[root@jjh ~]# docker exec -it mysql-master bash
root@2150cac30563:/# mysql -u root -p
Enter password: 
密码
mysql> GRANT REPLICATION SLAVE ON *.* TO 'root'@'%';
Query OK, 0 rows affected (0.00 sec)
# 这里使用root用户进行主从复制, %为允许所有ip进行复制

mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
# 刷新权限

mysql> show master status;
+------------------+----------+--------------+------------------+-------------------+
| File             | Position | Binlog_Do_DB | Binlog_Ignore_DB | Executed_Gtid_Set |
+------------------+----------+--------------+------------------+-------------------+
| mysql-bin.000003 |      543 |              |                  |                   |
+------------------+----------+--------------+------------------+-------------------+
1 row in set (0.00 sec)
#需要记录其中的File,  Position字段内容
```
#### 配置从服务器
```javascript
[root@jjh ~]# docker exec -it mysql-slave bash
root@2150cac30563:/# mysql -u root -p
Enter password: 
密码
#输入以下内容
mysql> change master to master_host='mysql-master',master_user='root',master_password='密码',master_log_file='File列的内容',master_log_pos=position的内容(不用加引号),master_port=3306;
Query OK, 0 rows affected, 2 warnings (0.02 sec)

mysql> start slave;
Query OK, 0 rows affected, 1 warning (0.01 sec)
# 启动slave

mysql> show slave status\G;
# 查看slave状态
*************************** 1. row ***************************
               Slave_IO_State: Waiting for master to send event
                  Master_Host: mysql-master
                  Master_User: root
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000003
          Read_Master_Log_Pos: 543
               Relay_Log_File: 5671fb724964-relay-bin.000002
                Relay_Log_Pos: 324
        Relay_Master_Log_File: mysql-bin.000003
             Slave_IO_Running: Yes
            Slave_SQL_Running: Yes

```
当Slave_IO_Running和Slave SQL Ruing都是Yes时，说明配置成功了，可以在主库创个库，添加下数据，看看能否在从库中查到

