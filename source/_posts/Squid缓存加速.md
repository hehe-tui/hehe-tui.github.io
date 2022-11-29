---
title: Squid缓存加速
date: 2020-12-28
updated:
description: 
cover: https://pic.imgdb.cn/item/63859d3f16f2c2beb1279261.jpg
tag:
  - Squid
categories:
  - 缓存加速
---
##  Squid简介
参考[Squid官网](http://www.squid-cache.org/)
###  工作机制
当客户机通过代理来请求Web页面时，指定的代理服务器会先检查自己的缓存，如果缓存中已经有客户机需要的页面，则直接将缓存中的页面内容反馈给客户机
如果缓存中没有客户机要访问的页面，则由代理服务器向Internet发送请求，当获得返回的Web页面后，将网页数据保存到缓存中并发送给客户机，同时记录缓存
###  代理基本类型
* 传统代理(普通代理)
内网>普通代理服务器>外网
内网客户机需配置浏览器的LAN代理IP及端口
无需设置防火墙规则及路由转发
* 透明代理
内网>透明代理服务器>外网
内网客户机无需添加配置
需要配置防火墙规则(REDIRECT 重定向)，开启路由转发功能
* 反向代理
内网服务器<反向代理服务器<外网客户机
外网客户机无需添加配置
无需添加防火墙规则
###  使用Squid代理服务器访问外网的优点
* 减少重复请求，节约带宽
* 具有ACL (Access Control List)访问控制列表功能，对客户机上网行为灵活控制
* 对内网客户机具有保护作用
##  传统代理
###  配置Squid

|服务器名|操作系统|IP|
|:--------:|:--------:|:--------:|
|squid|CentOS 7|192.168.0.36|
|nginx|CentOS 7|192.168.0.48|
|客户机|windows|192.168.0.52|

####  源码安装Squid
```javascript
[root@squid ~]# yum install -y perl autoconf automake make libxml2-devel libcap-devel libtool-ltdl-devel gcc gcc-c++
[root@squid ~]# ls
squid-4.13.tar.gz
[root@squid ~]# tar -zxvf squid-4.13.tar.gz -C /usr/local/
[root@squid ~]# cd /usr/local/squid-4.13/
[root@squid squid-4.13]# ./configure --prefix=/usr/local/squid
[root@squid squid-4.13]# make && make install
[root@squid squid-4.13]# ln -s /usr/local/squid/sbin/ /usr/local/sbin/
## 创建软链接
[root@squid squid-4.13]# useradd -M -s /sbin/nologin squid
## 创建管理进程的用户
[root@squid squid-4.13]# chown -R squid.squid /usr/local/squid/var/
## 修改文件的属主和属组
```
####  修改Squid的配置文件
```javascript
[root@squid squid-4.13]# vim /etc/squid.conf
···
cache_effective_user squid #添加指定程序用户
cache_effective_group squid #添加指定账号基本组
···
```
#### 启动Squid
```javascript
[root@squid squid-4.13]# export PATH=$PATH:/usr/local/squid/sbin/
[root@squid squid-4.13]# squid -k parse
## 检查配置文件语法
[root@squid squid-4.13]# squid           
## 启动Squid服务
[root@squid squid-4.13]# netstat -ntap | grep squid    
tcp6       0      0 :::3128                 :::*                    LISTEN      28971/(squid-1)       
## 服务启动，Squid服务处于正常监听状态
```
###  配置Nginx
nginx 的安装及修改之前有写过就不在这里详细说明了，记得打开日志
###  配置客户机
客户机的代理配置，在IE浏览器中，选择"工具"->"Internet选项",然后弹出的"Internet选项"对话框，在"连接"选项中的"局域网(LAN)设置"选项组中单击"局域网设置"按钮，弹出"局域网(LAN)设置"对话框，设置如下:
![](https://pic.downk.cc/item/5fe9a3053ffa7d37b3ddc84d.png)
访问 nginx 服务
![](https://pic.downk.cc/item/5fe9a30e3ffa7d37b3ddd7ca.png)
###  验证
```javascript
[root@nginx logs]# tailf access.log 
192.168.0.36 - - [28/Dec/2020:17:08:24 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko"
192.168.0.36 - - [28/Dec/2020:17:08:24 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko"
192.168.0.36 - - [28/Dec/2020:17:08:24 +0800] "GET / HTTP/1.1" 304 0 "-" "Mozilla/5.0 (Windows NT 10.0; WOW64; Trident/7.0; rv:11.0) like Gecko"
```
可以看到的确是代理服务器访问的，说明传统代理部署成功

