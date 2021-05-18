---
title: Codis介绍与安装
date: 2021-1-13
updated:
description:
cover: https://pic.imgdb.cn/item/609fe4e76ae4f77d353f969c.jpg
tag:
  - Codis
categories:
  - 数据库
---
##  Codis简介
Codis 是一个分布式 Redis 解决方案, 对于上层的应用来说, 连接到 Codis Proxy 和连接原生的 Redis Server 没有明显的区别 (不支持的命令列表), 上层应用可以像使用单机的 Redis 一样使用, Codis 底层会处理请求的转发, 不停机的数据迁移等工作, 所有后边的一切事情, 对于前面的客户端来说是透明的, 可以简单的认为后边连接的是一个内存无限大的 Redis 服务。
* codis-proxy 是客户端连接的 Redis 代理服务, codis-proxy 本身实现了 Redis 协议, 表现得和一个原生的 Redis 没什么区别 (就像 Twemproxy), 对于一个业务来说, 可以部署多个 codis-proxy, codis-proxy 本身是无状态的.
* codis-config 是 Codis 的管理工具, 支持包括, 添加/删除 Redis 节点, 添加/删除 Proxy 节点, 发起数据迁移等操作. codis-config 本身还自带了一个 http server, 会启动一个 dashboard, 用户可以直接在浏览器上观察 Codis 集群的运行状态.
* codis-server 是 Codis 项目维护的一个 Redis 分支, 基于 2.8.13 开发, 加入了 slot 的支持和原子的数据迁移指令. Codis 上层的 codis-proxy 和 codis-config 只能和这个版本的 Redis 交互才能正常运行.

Codis 依赖 ZooKeeper 来存放数据路由表和 codis-proxy 节点的元信息, codis-config 发起的命令都会通过 ZooKeeper 同步到各个存活的 codis-proxy

Codis 支持按照 Namespace 区分不同的产品, 拥有不同的 product name 的产品, 各项配置都不会冲突
###  特性
* 自动平衡
* 使用非常简单
* 图形化的面板和管理工具
* 支持绝大多数 Redis 命令，完全兼容twemproxy
* 支持 Redis 原生客户端
* 安全而且透明的数据移植，可根据需要轻松添加和删除节点
* 提供命令行接口
* RESTful APIs

参考[Codis官网](https://github.com/CodisLabs/codis)
##  Codis部署
### Go安装
```javascript
[root@codis ~]# yum install -y git gcc make g++ gcc-c++ automake openssl-devel zlib-*
[root@codis ~]# wget https://golang.google.cn/dl/go1.15.6.linux-amd64.tar.gz
[root@codis ~]# wget https://github.com/CodisLabs/codis/releases/download/3.2.2/codis3.2.2-go1.8.5-linux.tar.gz
[root@codis ~]# tar -zxvf go1.15.6.linux-amd64.tar.gz -C /usr/local/
[root@codis ~]# vim /etc/profile
···
export GOROOT=/usr/local/go   #设置为go安装的路径
export GOPATH=/usr/local/gopath  #默认安装包的路径
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
[root@codis ~]# source /etc/profile
[root@codis ~]# go version
go version go1.15.6 linux/amd64
```
### jdk安装
```javascript
[root@codis ~]# ls
go1.15.6.linux-amd64.tar.gz jdk-8u202-linux-x64.tar.gz
[root@codis ~]# tar -zxvf jdk-8u202-linux-x64.tar.gz -C /usr/local/
[root@codis ~]# vim /etc/profile
···
#java config
export JAVA_HOME=/usr/local/jdk1.8.0_202
export JRE_HOME=/usr/local/jdk1.8.0_202/jre
export CLASSPATH=.:${JAVA_HOME}/jre/lib/rt.jar:${JAVA_HOME}/lib/dt.jar:${JAVA_HOME}/lib/tools.jar
export PATH=$PATH:${JAVA_HOME}/bin
[root@codis ~]# source /etc/profile
[root@codis ~]# java -version
java version "1.8.0_202"
Java(TM) SE Runtime Environment (build 1.8.0_202-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.202-b08, mixed mode)
```
### ZooKeeper安装
```javascript
[root@codis ~]# wget http://mirror.bit.edu.cn/apache/zookeeper/zookeeper-3.6.2/apache-zookeeper-3.6.2-bin.tar.gz
[root@codis ~]# tar -zxvf apache-zookeeper-3.6.2-bin.tar.gz -C /usr/local/
[root@codis ~]# cd /usr/local/apache-zookeeper-3.6.2-bin/conf
[root@codis conf]# cp zoo_sample.cfg zoo.cfg
[root@codis conf]# vim zoo.cfg
···
dataDir=/usr/local/apache-zookeeper-3.6.2-bin/data
···
[root@codis conf]# cd ..
[root@codis apache-zookeeper-3.6.2-bin]# mkdir data
[root@codis apache-zookeeper-3.6.2-bin]# cd data/
[root@codis data]# vim myid
1
[root@codis data]# /usr/local/apache-zookeeper-3.6.2-bin//bin/zkServer.sh start
ZooKeeper JMX enabled by default
Using config: /usr/local/apache-zookeeper-3.6.2-bin/bin/../conf/zoo.cfg
Starting zookeeper ... STARTED
```
###  Codis安装
Codis 源代码需要下载到 $GOPATH/src/github.com/CodisLabs/codis
```javascript
[root@codis data]# cd
[root@codis ~]# mkdir -p $GOPATH/src/github.com/CodisLabs
# 目录结构一定要是,gopath的自定义，后面的目录需要新建和官网一样，不一样要报错
[root@codis ~]# cd $_ && git clone https://github.com/CodisLabs/codis.git -b release3.2
[root@codis ~]# cd $GOPATH/src/github.com/CodisLabs/codis
[root@codis codis]# make
```
#### 启动codis-dashboard
脚本启动 dashboard，并查看 dashboard 日志确认启动是否有异常
```javascript
[root@codis codis]# ./admin/codis-dashboard-admin.sh start
/usr/local/gopath/src/github.com/CodisLabs/codis/admin/../config/dashboard.toml
starting codis-dashboard ... 
[root@codis codis]# tail -100 ./log/codis-dashboard.log.2021-01-13
···
2021/01/13 14:22:28 main.go:171: [WARN] [0xc0001caea0] dashboard online failed [10]
```
执行报错
```javascript
[root@codis codis]# cd /tmp/codis/data/codis3/codis-demo/
[root@codis codis-demo]# ls
topom
[root@codis codis-demo]# rm topom 
rm: remove regular file ‘topom’? y
```
etcd已经有/ codis3 / codis-service / topom了，删除掉再启动就好
```javascript
[root@codis codis]# ./admin/codis-dashboard-admin.sh start
/usr/local/gopath/src/github.com/CodisLabs/codis/admin/../config/dashboard.toml
starting codis-dashboard ... 
[root@codis codis]# tail -100 ./log/codis-dashboard.log.2021-01-13
···
2021/01/13 14:33:33 main.go:179: [WARN] [0xc00009d680] dashboard is working ...
```
####  启动codis-proxy
脚本启动 codis-proxy，并查看 proxy 日志确认启动是否有异常
```javascript
[root@codis codis]# ./admin/codis-proxy-admin.sh start
/usr/local/gopath/src/github.com/CodisLabs/codis/admin/../config/proxy.toml
starting codis-proxy ... 
[root@codis codis]# tail -100 ./log/codis-proxy.log.2021-01-13
···
2021/01/13 14:43:08 main.go:343: [WARN] rpc online proxy seems OK
2021/01/13 14:43:09 main.go:233: [WARN] [0xc00018c580] proxy is working ...
```
####  启动codis-server
脚本启动 codis-server，并查看 redis 日志确认启动是否有异常
```javascript
[root@codis codis]# ./admin/codis-server-admin.sh start
/usr/local/gopath/src/github.com/CodisLabs/codis/admin/../config/redis.conf
starting codis-server ... 
[root@codis codis]# tail -100 /tmp/redis_6379.log
···
3006:M 13 Jan 14:46:31.533 * The server is now ready to accept connections on port 6379
```
####  启动codis-fe
脚本启动 codis-fe，并查看 fe 日志确认启动是否有异常
```javascript
[root@codis codis]# ./admin/codis-fe-admin.sh start

starting codis-fe ... 
[root@codis codis]# tail -100 ./log/codis-fe.log.2021-01-13
2021/01/13 14:49:32 main.go:101: [WARN] set ncpu = 2
2021/01/13 14:49:32 main.go:104: [WARN] set listen = 0.0.0.0:9090
2021/01/13 14:49:32 main.go:120: [WARN] set assets = /usr/local/gopath/src/github.com/CodisLabs/codis/bin/assets
2021/01/13 14:49:32 main.go:162: [WARN] set --filesystem = /tmp/codis
2021/01/13 14:49:32 main.go:216: [WARN] option --pidfile = /usr/local/gopath/src/github.com/CodisLabs/codis/bin/codis-fe.pid
```
####  通过fe添加group
通过web浏览器访问集群管理页面(fe地址:XX.XX.XX.XX:9090) 选择我们刚搭建的集群 codis-demo，在 Proxy 栏可看到我们已经启动的 Proxy， 但是 Group 栏为空，因为我们启动的 codis-server 并未加入到集群 添加 NEW GROUP，NEW GROUP 行输入 1，再点击 NEW GROUP 即可添加 Codis Server，Add Server 行输入我们刚刚启动的 codis-server 地址，添加到我们刚新建的 Group，然后再点击 Add Server 按钮即可，如下图所示：
![](https://img.imgdb.cn/item/5ffe9e333ffa7d37b3c44d22.png)
####  通过fe初始化slot
新增的集群 slot 状态是 offline，因此我们需要对它进行初始化（将 1024 个 slot 分配到各个 group），而初始化最快的方法可通过 fe 提供的 rebalance all slots 按钮来做，如下图所示，点击此按钮，我们即快速完成了一个集群的搭建。
![](https://img.imgdb.cn/item/5ffe9e393ffa7d37b3c4526c.png)
未完待续。。。
