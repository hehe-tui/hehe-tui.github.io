---
title: Redis介绍与安装
date: 2020-12-20
updated:
description:
cover: https://pic.imgdb.cn/item/60a8c2126ae4f77d35485f8a.jpg
tag:
  - Redis
categories:
  - 数据库
---
## Redis简介
Redis是一个开源(BSD许可),内存数据结构存储，用作数据库、缓存和消息代理。它 支持数据结构很多，如strings, hashes, lists, sets 等等。Redis 具有内置复制，Lua 脚本，LRU 清除缓存，事务和不同级别的磁盘持久性，并通过Redis Sentinel 提供高可用性，以及使用 Redis Cluster自动分区。

Redis是当前比较热门的NOSQL系统之一， 它是一个key-value存储系统。和Memcache类似，但很大程度补偿了Memcache的不足，它支持存储的value类型相对更多，包括string 、list、set、zset和hash。这些数据类型都支持push/pop、add/remove 及取交集并集和差集及 更丰富的操作。在此基础上，Redis 支持各种不同方式的排序。

和Memcache 一样，Redis数据都是缓存在计算机内存中，不同的是，Memcache只能 将数据缓存到内存中，无法自动定期写入硬盘，这就表示 一旦断电或重启，内存清空，数据丢失。所以Memcache的应用场景适用于缓存无需持久化的数据。而Redis不同的是它会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，实现数据的持久化。

参考[Redis官网](https://redis.io/)
##  Redis安装
###  环境安装
```javascript
[root@jjh ~]# yum -y install gcc-c++ centos-release-scl
[root@jjh ~]# yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
# 升级gcc版本到9.0以上
[root@jjh ~]# scl enable devtoolset-9 bash
# 需要注意的是scl命令启用只是临时的，退出shell或重启就会恢复原系统gcc版本。如果要长期使用gcc 9.0的话：
[root@jjh ~]# echo "source /opt/rh/devtoolset-9/enable" >>/etc/profile
# 这样退出shell重新打开就是新版的gcc了，以下其他版本同理，修改devtoolset版本号即可
```
### 下载Redis 6.0.9
```javascript
[root@jjh ~]# wget https://download.redis.io/releases/redis-6.0.9.tar.gz
[root@jjh ~]# tar -zxvf redis-6.0.9.tar.gz -C /usr/local/
[root@jjh ~]# cd /usr/local/redis-6.0.9/
[root@jjh ~]# ln -s /usr/local/redis-6.0.9/ /usr/local/redis
[root@jjh redis-6.0.9]# yum -y install tcl
[root@jjh redis-6.0.9]#  make
[root@jjh redis-6.0.9]#  make test
···
\o/ All tests passed without errors!

Cleanup: may take some time... OK
···
[root@jjh redis-6.0.9]# make install
[root@jjh redis-6.0.9]# mkdir conf bin
[root@jjh redis-6.0.9]# cp redis.conf sentinel.conf conf/
[root@jjh redis-6.0.9]# find src/ -perm 755 -type f -exec cp {} bin/ \;
[root@jjh redis-6.0.9]# ls bin/
redis-benchmark  redis-check-aof  redis-check-rdb  redis-cli  redis-sentinel  redis-server
[root@jjh redis-6.0.9]# echo 'PATH=$PATH:/usr/local/redis/bin/' >> /etc/profile
[root@jjh redis-6.0.9]# bash /etc/profile
[root@jjh redis-6.0.9]# redis-server
15486:C 20 Dec 2020 15:53:14.105 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15486:C 20 Dec 2020 15:53:14.105 # Redis version=6.0.9, bits=64, commit=00000000, modified=0, pid=15486, just started
15486:C 20 Dec 2020 15:53:14.105 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
                _._                                                  
           _.-``__ ''-._                                             
      _.-``    `.  `_.  ''-._           Redis 6.0.9 (00000000/0) 64 bit
  .-`` .-```.  ```\/    _.,_ ''-._                                   
 (    '      ,       .-`  | `,    )     Running in standalone mode
 |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
 |    `-._   `._    /     _.-'    |     PID: 15486
  `-._    `-._  `-./  _.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |           http://redis.io        
  `-._    `-._`-.__.-'_.-'    _.-'                                   
 |`-._`-._    `-.__.-'    _.-'_.-'|                                  
 |    `-._`-._        _.-'_.-'    |                                  
  `-._    `-._`-.__.-'_.-'    _.-'                                   
      `-._    `-.__.-'    _.-'                                       
          `-._        _.-'                                           
              `-.__.-'                                               

15486:M 20 Dec 2020 15:53:14.106 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
15486:M 20 Dec 2020 15:53:14.106 # Server initialized
15486:M 20 Dec 2020 15:53:14.106 # WARNING overcommit_memory is set to 0! Background save may fail under low memory condition. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
15486:M 20 Dec 2020 15:53:14.106 # WARNING you have Transparent Huge Pages (THP) support enabled in your kernel. This will create latency and memory usage issues with Redis. To fix this issue run the command 'echo madvise > /sys/kernel/mm/transparent_hugepage/enabled' as root, and add it to your /etc/rc.local in order to retain the setting after a reboot. Redis must be restarted after THP is disabled (set to 'madvise' or 'never').
15486:M 20 Dec 2020 15:53:14.106 * Ready to accept connections
```
### 再开一个终端查看下端口
```javascript
[root@jjh ~]# netstat -antp|grep :6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      15486/redis-server  
tcp6       0      0 :::6379                 :::*                    LISTEN      15486/redis-server  
```
### 修改配置文件
```javascript
[root@jjh redis-6.0.9]# vim conf/redis.conf 
···
68 bind 0.0.0.0
89 protected-mode no
224 daemonize yes
255 loglevel warning
260 logfile "/usr/local/redis/redis.log"
283 always-show-logo no
365 dir /usr/local/redis/
···
[root@jjh redis-6.0.9]# redis-server /usr/local/redis/conf/redis.conf 
[root@jjh redis-6.0.9]# netstat -antp|grep :6379
tcp        0      0 0.0.0.0:6379            0.0.0.0:*               LISTEN      15836/redis-server  
```
### 客户端测试
```javascript
[root@jjh redis-6.0.9]# redis-cli
127.0.0.1:6379> set NAME jjh
OK
127.0.0.1:6379> get NAME
"jjh"
```
##  Redis 持久化
redis支持两种持久化方式，一种是Snapshotting (快照)也是默认方式，另一种是 Append-only file (缩写aof)的方式。
###  Snapshotting
快照是默认的持久化方式。这种方式是就是将内存中数据以快照的方式写入到二进制文件中，默认的文件名为dump.rdb。可以通过配置设置自动做快照持久化的方式。我们可以配置redis在n秒内如果超过m个key被修改就自动做快照，下面是默认的快照保存配置。

* save 900 1        #900秒内如果超过1个key被修改，则发起快照保存
* save 300 10      #300秒内容如超过10个key被修改，则发起快照保存
* save 60 10000  #60秒内容如超过100000个key被修改，则发起快照保存
```javascript
[root@jjh ~]# cat /usr/local/redis/conf/redis.conf 
···
307 save 900 1
308 save 300 10
309 save 60 10000

342 dbfilename dump.rdb
···
```
如果需要恢复数据，只需要将备份文件（dump.rdb）移动到 redis 安装目录并启动服务即可。
想知道工作目录在哪
```javascript
[root@jjh ~]# redis-cli
127.0.0.1:6379> config get dir
1) "dir"
2) "/usr/local/redis-6.0.9"
```
save操作是在主线程中保存快照的，由于redis是用一个主线程来处理，这种方式会阻塞所有 client 请求，所以不推荐使用。另一点需要注意的是，每次快照持久化都是将内存数据完整写入到磁盘， 并不是增量的只同步脏数据。如果数据量大且写操作比较多，必然会引起大量的磁盘 io 操作，可能会严重影响性能。

另外由于快照方式存在一定间隔时间，如果 redis 意外 down 掉，就会丢失最后一次快照后的所有修改。
### Append-only file
aof 比快照方式有更好的持久化性，在使用 aof 持久化方式时，redis会将每一个收到的写命令都通过 write 函数追加到文件中(默认是appendonly.aof)。 当 redis 重启时会通过重新执行文件中保存的写命令来在内存中重建整个数据库的内容。

由于 os 会在内核中缓存 write 做的修改，所以可能不是立即写到磁盘上。这样aof方式的持久化也还是有可能会丢失部分修改。不过我们可以通过配置文件告诉redis我们想要通过 fsync 函数强制 os 写入到磁盘的时机。有三种方式如下(默认是:每秒 fsync 一次）

appendonly yes    //启用 aof 持久化方式
* appendfsync always      //每次收到写命令就立即强制写入磁盘，最慢的，但是保证完全的持久化，不推荐使用
* appendfsync everysec   //每秒钟强制写入磁盘次，在性能和持久化方面做了很好的折中，推荐
* appendfsync no             //完全依赖os,性能最好持久化没保证

aof 的方式也同时带来了另一个问题：持久化文件会变的越来越大。例如我们调用 incr test 命令100次，文件中必须保存全部的100条命令，其实有99条都是多余的。因为要恢复数据库的状态其实文件中保存一条 set test 100就够了。为了压缩 aof 的持久化文件。redis 提供了 bgrewriteaof 命令。收到此命令 redis 将使用与快照类似的方式将内存中的数据以命令的方式保存到临时文件中，最后替换原来的文件。
```javascript
[root@jjh ~]# cat /usr/local/redis/conf/redis.conf 
···
307 #save 900 1
308 #save 300 10
309 #save 60 10000

1089 appendonly yes
1093 appendfilename "appendonly.aof"
1119 appendfsync everysec
···
```
##  Redis 主从复制
192.168.0.36 jjh
192.168.0.42 jjh2
需要注意，主从复制的开启，完全是在从节点发起的，不需要我们在主节点做任何事情
从节点开启主从复制，有3种方式：
* 配置文件
在从服务器的配置文件中加入：slaveof <masterip> <masterport>
* 启动命令
redis-server启动命令后加入 --slaveof <masterip> <masterport>
* 客户端命令
Redis服务器启动后，直接通过客户端执行命令：slaveof <masterip> <masterport>，则该Redis实例成为从节点

上述3种方式是等效的，下面以客户端命令的方式为例，看一下当执行了slaveof后，Redis主节点和从节点的变化
```javascript
[root@jjh2 ~]# redis-cli
127.0.0.1:6379> slaveof 192.168.0.36 6379
OK
```
在 master 上插入数据
```javascript
[root@jjh ~]# redis-cli
127.0.0.1:6379> set file 1.txt
OK
127.0.0.1:6379> get file
"1.txt"
```
查看是否成功
```javascript
[root@jjh2 ~]# redis-cli
127.0.0.1:6379> get file
"1.txt"
```
##  Redis Sentinel (哨兵)
Redis Sentinl 为 Redis 提供高可用性。实际上，这意味使用 Sentinel 可以创建一个Redis 部署，可以在没有人为干预的情况下抵御某些类型的故障。Redts Sentinel 还提供其他附属任务，如监控、通知、并充当客户端的配置提供程序。

Sentinel 功能：
* 监控：Sentinel 会不断检查主实例和从属实例是否按预期工作。
* 通知：Sentinel可以通过API通知系统管理员，另一台计算机程序，其中一个受监控的Redis实例出现问题。
* 自动故障转移：如果主服务器未按预期工作，Sentinel 可以启动故障转移过程，其中从服务器被提升为主服务器，其他其他服务器将重新配置为使用新主服务器，并且使用 Redis 服务器的应用程序会通知有关新服务器的地址连接。
* 配置提供商：Sentinel 充当客户端服务发现的权限来源：客户端连接到Sentinel ，以便询问负责给定服务的当前 Redis 主服务器的地址。如果发生故障转移，Sentinel 将报告新地址。

主从节点配置 sentinel.conf 文件
```javascript
[root@jjh redis-6.0.9]# cat conf/sentinel.conf 
17 protected-mode no      //设置为非保护模式
26 daemonize yes          //设置为守护线程方式运行
68 sentinel monitor mymaster 192.168.0.36 6379 2   //设置监听的master服务器(服务名，IP，端口，投票数)
165 sentinel failover-timeout mymaster 5000  //故障转移时间，设置为5秒，默认为3分钟
```
启动 sentinel 服务，查看主从节点的状态
```javascript
[root@jjh redis-6.0.9]# redis-server /usr/local/redis/conf/sentinel.conf --sentinel
[root@jjh redis-6.0.9]# redis-cli
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:2
slave0:ip=192.168.0.43,port=6379,state=online,offset=478558,lag=1
slave1:ip=192.168.0.42,port=6379,state=online,offset=478419,lag=1
master_replid:6efce840d54c12d3e558966ec3adc6e41cfd422c
master_replid2:01b5782acd9d0e503fded1438c05ec6c7ca58201
master_repl_offset:478558
second_repl_offset:8420
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:478558

[root@jjh2 ~]# redis-cli
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.0.36
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:522776
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:6efce840d54c12d3e558966ec3adc6e41cfd422c
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:522776
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:48185
repl_backlog_histlen:474592

[root@jjh3 redis-6.0.9]# redis-cli
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.0.36
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:490443
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:6efce840d54c12d3e558966ec3adc6e41cfd422c
master_replid2:01b5782acd9d0e503fded1438c05ec6c7ca58201
master_repl_offset:490443
second_repl_offset:8420
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:490443
```
关闭主节点
```javascript
[root@jjh redis-6.0.9]# redis-cli
127.0.0.1:6379> shutdown
not connected> 
```
查看从节点的状态
```javascript
[root@jjh2 ~]# redis-cli
127.0.0.1:6379> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=192.168.0.43,port=6379,state=online,offset=549683,lag=1
master_replid:f0287a0dd102e1283d53d362a53d13bbe7666771
master_replid2:6efce840d54c12d3e558966ec3adc6e41cfd422c
master_repl_offset:549822
second_repl_offset:532438
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:48185
repl_backlog_histlen:501638

[root@jjh3 ~]# redis-cli
127.0.0.1:6379> info replication
# Replication
role:slave
master_host:192.168.0.42
master_port:6379
master_link_status:up
master_last_io_seconds_ago:1
master_sync_in_progress:0
slave_repl_offset:561568
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:f0287a0dd102e1283d53d362a53d13bbe7666771
master_replid2:6efce840d54c12d3e558966ec3adc6e41cfd422c
master_repl_offset:561568
second_repl_offset:532438
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:561568
```
可以看到主节点已经自动切换，说明 sentinel.conf 配置已经成功。
