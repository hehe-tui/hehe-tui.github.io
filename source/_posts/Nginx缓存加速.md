---
title: Nginx缓存加速
date: 2020-12-31
updated:
description:
cover: https://pic.downk.cc/item/5fe9a5cf3ffa7d37b3e28311.jpg
tag:
  - Nginx
categories:
  - 缓存加速
---
##  Nginx 缓存加速概述
### Nginx支持类似Squid的缓存功能
* 把URL以及相关信息当成key,用MD5编码哈希后，把数据文件保存在硬盘上。只能为指定的URL或者状态码设置过期时间，并不支持类似squid的purge命令来手动清除指定缓存页面。
* 可通过第三方的ngx_ cache_purge 来清除指定的URL缓存。
* Nginx的缓存加速功能是由proxy_cache和fastcgi_cache 两个功能模块完成的。
### Nginx缓存加速特点
* 缓存功能十分稳定，运行速度不逊于squid, 对多核CPU的利用率比其他的开源软件也要好，支持高并发请求数，能同时承受更多的访问请求。
##  Nginx 缓存加速案例

|服务器名|操作系统|IP|
|:--------:|:--------:|:--------:|
|nginx  ngx_cache_purge|CentOS 7|192.168.0.36|
|web|windows|192.168.0.54|

###  Nginx端配置
#### 源码安装Nginx
```javascript
[root@nginx ~]# yum -y install pcre-devel zlib-devel gcc gcc-c++ make
[root@nginx ~]# wget https://nginx.org/download/nginx-1.18.0.tar.gz
[root@nginx ~]# wget http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz
[root@nginx ~]# ls
nginx-1.18.0.tar.gz  ngx_cache_purge-2.3.tar.gz
[root@nginx ~]# yum -y install pcre-devel zlib-devel gcc gcc-c++ make
[root@nginx ~]# tar -zvxf nginx-1.18.0.tar.gz -C /usr/local/
[root@nginx ~]# tar -zvxf ngx_cache_purge-2.3.tar.gz -C /usr/local/
[root@nginx ~]# cd /usr/local/nginx-1.18.0/
[root@nginx nginx-1.18.0]# ./configure --prefix=/usr/local/nginx --user=nginx --group=nginx --with-http_stub_status_module --with-pcre --add-module=/usr/local/ngx_cache_purge-2.3/
[root@nginx nginx-1.18.0]# make && make install
[root@nginx nginx-1.18.0]# ln -s /usr/local/nginx/sbin/nginx /usr/local/sbin/
```
####  修改配置文件
```javascript
[root@nginx nginx-1.18.0]# cd /usr/local/nginx/conf/
[root@nginx conf]# vim nginx.conf
#user  nobody;
user nginx nginx;
# 以nginx用户和组运行

worker_processes  1;
# 启动进程数，根据物理CPU个数设置

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;
error_log  logs/error.log  crit;
# 定义错误日志，级别为crit

pid        logs/nginx.pid;

worker_rlimit_nofile 65535;
# 打开文件的最大句柄数，最好与ulimit -n的值保持一致，使用ulimit -SHn 进行设置

events {
    use epoll;
    # Nginx 使用了最新的epoll网络I/O模型
    worker_connections  65535;
    # 每个工作进程允许最大的同时连接数
}


http {
    include       mime.types;
    # mime.types 内定义各文件类型映像
    default_type  application/octet-stream;
    # 设置默认类型是二进制流，若没有设置，比如加载PHP时，是不预解析，用浏览器访问则出现下载窗口
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    # 打开系统函数sendfile()，支持下载
    tcp_nopush     on;
    # 只有先打开Linux下的tcp_cork,sendfile打开时才有效
    
    #keepalive_timeout  0;
    keepalive_timeout  65;
    # 会话保持时间，设置的低一些可以让nginx持续工作的时间更长
    
    #gzip  on;
    
    tcp_nodelay on;
    # 不要缓存数据，而是一段一段的发送——当需要及时发送数据时，就应该设置这个属性，这样发送一小块数据信息时就不能立即得到返回值
    client_body_buffer_size 512k;
    # 指定连接请求实体的缓存大小
    proxy_connect_timeout 5;
    # 代理连接超时时间，单位秒
    proxy_read_timeout 60;
    # 代理接收超时
    proxy_send_timeout 5;
    # 代理发送超时
    proxy_buffer_size 16k;
    # 代理缓存文件大小
    proxy_buffers 4 64k;
    # 代理缓存区的数量及大小，默认一个缓冲区大小与页面大小相等
    proxy_busy_buffers_size 128k;
    # 高负荷下缓存区大小
    proxy_temp_file_write_size 128k;   
    # 代理临时文件大小
    proxy_temp_path /var/cache/nginx/cache_temp;
    # 代理临时文件存放目录
    proxy_cache_path /var/cache/nginx/proxy_cache levels=1:2 keys_zone=cache_one:200m inactive=1d max_size=30g;
    # 代理缓存存放路径，第一层目录只有一个字符，是由levels=1:2设置，总共二层目录，子目录名字由二个字符组成，键值名称为cache_one（名字随意），在内存中缓存的空间大小为200MB，1天内没有被访问的缓存将自动清除，硬盘缓存空间为30GB
    # 注：proxy_temp_path与proxy_cache_path指定的路径必须在同一分区
    
    upstream backend_server {
         server 192.168.0.54:80 weight=1 max_fails=2 fail_timeout=30s;
    }
    # 上游服务器节点，权重1（数字越大权重越大），30秒内访问失败次数大于等于2次，将在30秒内停止访问此节点，30秒后计数器清零，可以重新访问。
    
    server {
        listen       80;
        server_name  localhost 192.168.0.36;
  
        #charset koi8-r;
        charset utf-8;
        
        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
            proxy_next_upstream http_502 http_504 error timeout invalid_header;
            # 如果后端服务器返回502、504、错误等错误，自动跳转到upstream负载均衡池中的另一台服务器，实现故障转义
	        proxy_cache cache_one
            proxy_cache_valid 200 304 2h;
            # 对不同的HTTP状态码设置不同的缓存时间
            proxy_cache_key $host$uri$is_args$args;
            # 以域名、URI、参数组合成Web缓存的Key值，nginx根据Key值哈希，存储缓存内容到二级缓存目录内
	        proxy_set_header Host $host;
	        proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://backend_server;
            # 指定跳转服务器池，名字要与upstream设定的相同
            expires 1d;
            # 用于清除缓存
        }
        
        location ~ /purge(/.*) {
        # 设置允许清除缓存的主机IP或网段
            allow 127.0.0.1;
	        allow 192.168.0.0/24;
	        deny all;
	        proxy_cache_purge cache_one $host$1$is_args$args;
	}
        
        location ~ .*\.(php|jsp|cgi)?$ {
        # 扩展名以php、jsp、cgi结尾的动态应用程序不缓存
    	    proxy_set_header Host $host;
            proxy_set_header X-Forwarded-For $remote_addr;
            proxy_pass http://backend_server;
	}
        access_log off;
        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
    }
}
[root@nginx conf]# useradd -M -s /sbin/nologin nginx
[root@nginx conf]# mkdir /var/cache/nginx
[root@nginx conf]# ulimit -SHn 65535
[root@nginx conf]# nginx -t
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
[root@nginx conf]# netstat -antp |grep nginx
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      32540/nginx: master 
```
### web端配置
```javascript
[root@web ~]# yum -y install httpd
[root@web ~]# vim /etc/httpd/conf/httpd.conf
···
ServerName www.example.com:80
···
[root@web ~]# echo "liangzai" >/var/www/html/index.html
[root@web ~]# systemctl start httpd
[root@web ~]# netstat -antp |grep httpd
tcp6       0      0 :::80                   :::*                    LISTEN      28694/httpd         
```
###  验证
####  测试Nginx缓存服务器
```javascript
[root@nginx conf]# ll /var/cache/nginx/proxy_cache/
total 0
```
####  浏览器访问测试
![](https://pic.downk.cc/item/5fedc3d33ffa7d37b3b2ac1b.png)
####  查看Nginx服务器缓存
```javascript
[root@nginx conf]# ls /var/cache/nginx/proxy_cache/0/
4c
[root@nginx conf]# ls /var/cache/nginx/proxy_cache/0/4c
29513b424f76af259b61d73cbe4df4c0
```
####  清除浏览器缓存
![](https://pic.downk.cc/item/5fedc3d73ffa7d37b3b2b288.png)
####  查看Nginx服务器缓存
```javascript
[root@nginx conf]# ls /var/cache/nginx/proxy_cache/0/4c

```
可以看到初步搭建成功

