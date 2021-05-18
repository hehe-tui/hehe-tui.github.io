---
title: HAProxy+Nginx负载均衡介绍与搭建
date: 2020-12-28
updated:
description:
cover: https://img.imgdb.cn/item/6002932d3ffa7d37b3d81018.jpg
tag:
  - HAProxy
  - Nginx
categories:
  - 负载均衡
---
## HAProxy简介
HAProxy提供高可用性、负载均衡以及基于TCP和HTTP的应用代理，支持虚拟主机，它是免费、快速并且可靠的一种负载均衡解决方案。适合处理高负载站点的七层数据请求。类似的代理服务可以屏蔽内部真实服务器，防止内部服务器遭受攻击。
### HAProxy主要优点:
* HAProxy是支持虚拟主机的，通过frontend指令来实现
* 能够补充Nginx的一些缺点比如Session的保持，Cookie 的引导等工作
* 支持url检测后端的服务器出问题的检测会有很好的帮助。
* 它跟LVS一样，本身仅仅就只是款负载均衡软件;单纯从效率上来讲HAProxy 更会比Nginx有更出色的负载均衡速度，在并发处理上也是优于Nginx的。
* HAProxy可以对MySQL读进行负载均衡，对后端的MySQL节点进行检测和负载均衡，不过在后端的MySQ slaves数量超过10台时性能不如LVS,所以更推荐LVS+Keepalived。
* 能对请求的url和header中的信息做匹配，有比Ivs有更好的7层实现
##  HAProxy+Nginx负载均衡安装
### HAProxy安装

|服务器名|操作系统|IP|
|:--------:|:--------:|:--------:|
|HAProxy|CentOS 7|192.168.0.36|
|Nginx1|CentOS 7|192.168.0.42|
|Nginx2|CentOS 7|192.168.0.43|

#### 准备lua环境
由于CentOS7 之前版本自带的lua版本比较低并不符合HAProxy要求的lua最低版本(5.3)的要求，因此需要编译安装较新版本的lua环境
参考[lua官方网站](http://www.lua.org/)
```javascript
[root@haproxy ~]# lua -v
Lua 5.1.4  Copyright (C) 1994-2008 Lua.org, PUC-Rio
[root@haproxy ~]# yum -y install gcc readline-devel 
[root@haproxy ~]# cd /usr/local/src
[root@haproxy src]# wget http://www.lua.org/ftp/lua-5.4.2.tar.gz
[root@haproxy src]# tar -zxvf lua-5.4.2.tar.gz 
[root@haproxy src]# cd lua-5.4.2/
[root@haproxy lua-5.4.2]# make linux test
[root@haproxy lua-5.4.2]# src/lua -v
Lua 5.4.2  Copyright (C) 1994-2020 Lua.org, PUC-Rio
```
####  源码安装HAProxy
参考[HAProxy官网](https://www.haproxy.com/)
```javascript
[root@haproxy ~]# yum -y install gcc openssl-devel pcre-devel systemd-devel
[root@haproxy ~]# ls
haproxy-2.2.6.tar.gz
[root@haproxy ~]# tar -zxvf haproxy-2.2.6.tar.gz -C /usr/local/
[root@haproxy ~]# cd /usr/local/haproxy-2.2.6/
[root@haproxy haproxy-2.2.6]# make  ARCH=x86_64 TARGET=linux-glibc  USE_PCRE=1 USE_OPENSSL=1 USE_ZLIB=1  USE_SYSTEMD=1  USE_LUA=1 LUA_INC=/usr/local/src/lua-5.4.2/src/  LUA_LIB=/usr/local/src/lua-5.4.2/src/ 
[root@haproxy haproxy-2.2.6]# make install PREFIX=/usr/local/haproxy
```
#### 修改配置文件
[root@haproxy haproxy-2.2.6]# mkdir /etc/haproxy
[root@haproxy haproxy-2.2.6]# vim /etc/haproxy/haproxy.cfg
```javascript
global
     log 127.0.0.1 local2 info
     chroot /usr/local/haproxy
     user haproxy
     group haproxy
     daemon           ## 守护进程模式    可以使用非守护默认
     maxconn 4000     ## 最大的连接数

defaults              ## 默认配置
     log global       ## 应用全局部分的日志配置
     mode http        ## 模式为http
     option httplog
     option dontlognull
     timeout connect 5000     ## 连接超时时间
     timeout client 50000
     timeout server 50000     ## 客户端和服务器超时时间

frontend http_front
     bind *:80
     stats uri /haproxy-status
     stats auth    jjh:123456
     default_backend http_back

backend http_back
     balance roundrobin
     option httpchk GET /index.html
     option forwardfor header X-Forwarded-For
     server nginx1 192.168.0.42:80 check inter 2000 rise 3 fall 3 weight 30   ## 服务器节点的地址、名称、端口 、检查间隔时间3000毫秒、健康检查次数2次认为失败
     server nginx2 192.168.0.43:80 check inter 2000 rise 3 fall 3 weight 30 
```
#### 启动 HAProxy
```javascript
[root@haproxy1 haproxy-2.2.6]# cp haproxy /usr/sbin/
[root@haproxy2 haproxy-2.2.6]# useradd -r haproxy
[root@haproxy2 haproxy-2.2.6]# cp ./examples/haproxy.init /etc/init.d/haproxy
[root@haproxy2 haproxy-2.2.6]# chmod 755 /etc/init.d/haproxy 
[root@haproxy2 haproxy-2.2.6]# /etc/init.d/haproxy start
Reloading systemd:                                         [  OK  ]
Starting haproxy (via systemctl):                          [  OK  ]   
```
访问 http://localhost/haproxy-status
用户名/密码（haproxy.cfg配置文件）   jjh/123456
![](https://pic.downk.cc/item/5fe95d1b3ffa7d37b36af6f5.jpg)
###  Nginx安装
参考[Nginx官方网站](https://nginx.com/) 
点击[Nginx下载](https://nginx.org/)
####  依赖包安装
```javascript
[root@nginx ~]# yum -y install pcre-devel zlib-devel gcc gcc-c++ make
```
####  源码安装Nginx
```javascript
[root@nginx ~]# ls
nginx-1.18.0.tar.gz
[root@nginx ~]# tar -zxvf nginx-1.18.0.tar.gz -C /usr/local/
[root@nginx ~]# useradd -M -s /sbin/nologin nginx
[root@nginx1 ~]# cd /usr/local/nginx-1.18.0/
[root@nginx1 nginx-1.18.0]# ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module && make && make install
[root@nginx1 nginx-1.18.0]# ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
[root@nginx1 nginx-1.18.0]# echo "shuaige/liangzai(nginx内容不同以作区分)" > /usr/local/nginx/html/index.html
```
####  Nginx 启动
```javascript
[root@nginx1 nginx-1.18.0]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```
####  重启报错及解决方法
```javascript
[root@nginx1 nginx-1.18.0]# nginx -s reload
nginx: [error] invalid PID number "" in "/usr/local/nginx/logs/nginx.pid"
[root@nginx1 nginx-1.18.0]# /usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
[root@nginx1 nginx-1.18.0]# nginx -s reload
```
###  验证
```javascript
[root@jjh ~]# curl 192.168.0.36
shuaige
[root@jjh ~]# curl 192.168.0.36
liangzai
```
可以看到初步搭建成功

