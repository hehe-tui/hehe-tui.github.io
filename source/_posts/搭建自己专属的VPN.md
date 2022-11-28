---
title: 搭建自己专属的VPN
date: 2020-11-20
updated:
description:  谁能拒绝一个酷炫的vpn呢！
cover: https://pic.imgdb.cn/item/6384a73516f2c2beb1da9157.jpg
tag:
  - Shadowsocks
categories:
  - VPN
---
首先你需要准备一台海外服务器
## 安装Shadowsocks服务端
```
[root@jjh ~]# yum install python-setuptools && easy_install pip
[root@jjh ~]# pip install shadowsocks
```
参考[官方Shadowsocks使用说明](https://github.com/shadowsocks/shadowsocks/wiki/Shadowsocks-%E4%BD%BF%E7%94%A8%E8%AF%B4%E6%98%8E)
### 配置
修改配置文件 /etc/shadowsocks.json，如果没有可以新建
内容如下：
```
{
        "server":"0.0.0.0",
        "server_port":" 你的端口 ",
        "password":" 你的密码 ",
        "timeout":300,
        "method":"aes-256-cfb",
        "fast_open":false,
        "workers":4
}
```
或者设置多个账号
```
{
        "server":"0.0.0.0",
        "port_password":{
        "端口一":"密码",
        "端口二":"密码",
        }，
        "timeout":300,
        "method":"aes-256-cfb",
        "fast_open":false,
}
```
配置说明

 |字段|说明|
 |----|----|
 |server|ss服务监听地址|
 |server_port|ss服务监听端口|
 |local_address|本地监听地址|
 |local_port|本地服务监听地址|
 |password|密码|
 |timeout|超时时间，单位 秒|
 |method|加密方法，默认是aes-256-cfb|
 |fast_open|使用TCP_FASTOPEN，true/false|
 |workers|workers数，只支持Unix/Linux系统|

参考[Configuration via Config File](https://github.com/shadowsocks/shadowsocks/wiki/Configuration-via-Config-File)
### 启动
前台启动
```
ssserver -c /etc/shadowsocks.json
```
后台启动与停止
```
ssserver -c /etc/shadowsacks.json -d start
ssserver -c /etc/shadowsacks.json -d stop
```
设置开机自启
修改/etc/rc.local，加入如下内容
```
ssserver -c /etc/shadowsacks.json -d start
```
日志
shadowsocks的日志保存在 /var/log/shadowsocks.log

## 安装Shadowsocks客户端
```
[root@jjh ~]# yum install python-setuptools && easy_install pip
[root@jjh ~]# pip install git+https://github.com/shadowsocks/shadowsocks.git@master
```
### 配置
创建一个/etc/shadowsocks.json文件，格式如下
```
{
    "server":"服务器 IP 或是域名",
    "server_port":端口号,
    "local_address": "127.0.0.1",
    "local_port":1080,
    "password":"密码",
    "timeout":300,
    "method":"加密方式 (chacha20-ietf-poly1305 / aes-256-cfb)",
    "fast_open": false
}
```
### 启动
```
sslocal -c /etc/shadowsocks.json -d start
```
## 验证
```
[root@jjh ~]# curl --socks5 127.0.0.1:1080 http://httpbin.org/ip
{
  "origin": "xxx.xxx.xxx.xxx"
}
```
如果返回你的ss服务器ip,则测试成功。

此文章仅用于技术交流，请勿作非法之事！
