---
title: Samba介绍与安装
date:
updated:
description:
cover: https://pic.imgdb.cn/item/609d2cced1a9ae528f426b91.jpg
tag:
  - Samba
categories:
  - 共享服务
---
## Samba简介
Samba是在Linux和UNIX系统上实现SMB协议的一个免费软件，由服务器及客户端程序构成。可以在各个操作系统之间实现资源共享。
Samba包含smbd和 nmbd两个关键程序，可以实施四种基本的现代CIFS服务：
* 文件和打印服务
* 认证与授权
Smbd 提供“共享模式”和“用户模式”的身份验证和授权，可以通过要求密码来保护共享文件和打印服务。
共享模式：可以将密码分配给共享目录或打印机，然后将这个单一密码提供给允许使用共享的每个人。
用户模式：每个用户都有自己的用户名和密码，系统管理员可以分别授予或拒绝访问。
* 名称解析
名称解析有两种形式：广播和点对点
客户端将其NetBIOS名称和IP地址发送到NBNS服务器，该服务器将信息保存在简单的数据库中，当客户端要与另一个客户端通话时，它将另一个客户端的名称发送到NBNS服务器。如果名称在列表中，则NBNS递回IP地址。不同子网中的客户端可以共享同一台NBNS服务器。
* 服务公告（浏览）
在LAN上，参与的计算机进行选举，以决定其中的哪些计算机将成为本地主浏览器（LMB）。然后，“优胜者”通过声明一个特殊的NetBIOS名称来标识自己。LMB的工作是保留可用服务的列表。
域主浏览器（DMB）可以在路由网络上跨NT域协调浏览列表。使用NBNS，LMB将找到其DMB，以交换和合并浏览列表。
[Samba官网](https://wiki.samba.org/)
##  Samba安装
###  环境搭建
|服务器名|操作系统|IP|
|--|--|--|
|Samba|CentOS7|192.168.100.101|

####  源码安装 Samba
```javascript 
[root@samba ~]# yum install -y epel-release 
[root@samba ~]# yum install -y python3 python3-devel perl-Parse-Yapp libtasn1-devel libunistring-devel zlib-devel gmp-devel libffi-devel libldap2-dev lmdb openldap-devel m4 gcc lmdb-devel flex wget
[root@samba ~]# wget https://gmplib.org/download/gmp/gmp-6.2.1.tar.xz && xz -d gmp-6.2.1.tar.xz && tar -xvf gmp-6.2.1.tar && cd gmp-6.2.1
[root@localhost gmp-6.2.1]# ./configure
[root@localhost gmp-6.2.1]# make && make check && make install
[root@samba gmp-6.2.1]# export GMP_CFLAGS="-I/usr/local/include" GMP_LIBS="-L/usr/local/lib -lgmp"

[root@samba ~]# wget https://ftp.gnu.org/gnu/nettle/nettle-3.7.2.tar.gz && tar -xzvf nettle-3.7.2.tar.gz && cd nettle-3.7.2
[root@samba nettle-3.7.2]# ./configure --prefix=/usr --enable-static --enable-mini-gmp
[root@samba nettle-3.7.2]# make && make install
[root@samba nettle-3.7.2]# chmod -v 755 /usr/lib64/lib{hogweed,nettle}.so && install -v -m755 -d /usr/share/doc/nettle-3.7.2 && install -v -m644 nettle.html /usr/share/doc/nettle-3.7.2
[root@samba nettle-3.7.2]# nettle-hash --version
## 查看 nettle 版本
nettle-hash (nettle 3.7.2)

[root@samba ~]# wget https://ftp.gnu.org/gnu/libtasn1/libtasn1-4.16.0.tar.gz && tar -xzvf libtasn1-4.16.0.tar.gz && cd libtasn1-4.16.0
[root@samba libtasn1-4.16.0]# export CFLAGS="-std=c99"
[root@samba libtasn1-4.16.0]# ./configure --prefix=/usr --disable-static
[root@samba libtasn1-4.16.0]# make && make install
[root@samba libtasn1-4.16.0]# export CFLAGS=""

[root@samba ~]# wget ftp://sourceware.org/pub/libffi/libffi-3.3.tar.gz && tar xf libffi-3.3.tar.gz && cd libffi-3.3
[root@samba libffi-3.3]# ./configure --prefix=/usr
[root@samba libffi-3.3]# make && make install
[root@samba libffi-3.3]# export LD_LIBRARY_PATH=/usr/libffi-3.3:$LD_LIBRARY_PATH

[root@samba ~]# wget ftp://ftp.gnu.org/gnu/libidn/libidn2-2.3.0.tar.gz && tar -xzvf libidn2-2.3.0.tar.gz && cd libidn2-2.3.0
[root@samba libidn2-2.3.0]# ./configure --prefix=/usr --disable-static
[root@samba libidn2-2.3.0]# make && make install
[root@samba libidn2-2.3.0]# export LIBIDN2_CFLAGS="-I/usr/include" LIBIDN2_LIBS="-L/usr/lib -lidn2"

[root@samba ~]# wget https://www.gnupg.org/ftp/gcrypt/gnutls/v3.7/gnutls-3.7.1.tar.xz && xz -d gnutls-3.7.1.tar.xz && tar -xvf gnutls-3.7.1.tar && cd gnutls-3.7.1
[root@samba gnutls-3.7.1]# ./configure --prefix=/usr --docdir=/usr/share/doc/gnutls-3.7.1 --without-p11-kit 
[root@samba gnutls-3.7.1]# make && make install
[root@samba gnutls-3.7.1]# ldconfig && gnutls-cli -v
gnutls-cli 3.7.1
Copyright (C) 2000-2020 Free Software Foundation, and others, all rights reserved.
This is free software. It is licensed for use, modification and
redistribution under the terms of the GNU General Public License,
version 3 or later <http://gnu.org/licenses/gpl.html>

Please send bug reports to:  <bugs@gnutls.org>

[root@samba ~]# wget https://download.samba.org/pub/samba/stable/samba-4.14.3.tar.gz && tar xf samba-4.14.3.tar.gz && cd samba-4.14.3
[root@samba samba-4.14.3]# ln -sf /usr/lib/pkgconfig/gnutls.pc /usr/lib64/pkgconfig/gnutls.pc
[root@samba samba-4.14.3]# ln -sf /usr/lib/libgnutls.so /usr/lib64/libgnutls.so
[root@samba samba-4.14.3]# ln -sf /usr/lib/libgnutls.so.30 /usr/lib64/libgnutls.so.30
[root@samba samba-4.14.3]# ln -sf /usr/lib/pkgconfig/libidn2.pc  /usr/lib64/pkgconfig/libidn2.pc
[root@samba samba-4.14.3]# ./configure --disable-python --without-ad-dc --without-json --without-libarchive  --without-acl-support --without-pam --with-shared-modules='!vfs_snapper'
[root@samba samba-4.14.3]# make && make install
```
####  验证
```javascript
[root@samba ~]# vim /etc/ld.so.conf
···
/usr/local/samba/lib
## 加入一行/usr/local/samba/lib
···
[root@samba ~]# ldconfig
## 执行ldconfig命令让配置生效
[root@samba ~]# vim /usr/local/samba/etc/smb.conf
## 创建配置文件
[root@samba ~]# ln –s  /usr/local/samba/etc/smb.conf   /usr/local/samba/lib/smb.conf 
[root@samba ~]# /usr/local/samba/bin/testparm
## 如果没有任何错误，说明samba已经安装成功
Load smb config files from /usr/local/samba/etc/smb.conf
Loaded services file OK.
Weak crypto is allowed
Server role: ROLE_STANDALONE

Press enter to see a dump of your service definitions
```
###  共享模式
####  修改配置文件
```javascript
[root@samba ~]# vim /usr/local/samba/etc/smb.conf
...
[global]
        workgroup = SAMBA          
        ## 工作组名称
        security = user            
        ## 设置用户访问samba服务器的验证方式
        ## 1. share：用户访问Samba Server不需要提供用户名和口令, 安全性能较低。
        ## 2. user：Samba Server共享目录只能被授权的用户访问,由Samba Server负责检查账号和密码的正确性。账号和密码要在本Samba Server中建立。
        ## 3. server：依靠其他Windows NT/2000或Samba Server来验证用户的账号和密码,是一种代理验证。此种安全模式下,系统管理员可以把所有的Windows用户和口令集中到一个NT系统上,使用 Windows NT进行Samba认证, 远程服务器可以自动认证全部用户和口令,如果认证失败,Samba将使用用户级安全模式作为替代的方式。
        ## 4. domain：域安全级别,使用主域控制器(PDC)来完成认证。
        ## passdb backend = tdbsam    
        ## 定义用户后台类型
        ## printing = cups
        ## 设置共享打印机的配置文件
        ## printcap name = cups
        ## Samba共享打印机的类型。
        ## 现在支持的打印系统有：bsd, sysv, plp, lprng, aix, hpux, qnx
        ## load printers = yes
        ## 打印共享功能
        ## cups options = raw
        ## 打印机选项
        map to guest = Bad User 
        ## 添加此项，开启匿名用户访问
[public]
##添加的share文件
        path=/samba/public
        ##路径
        public=yes
        ##公共访问
        browseable=yes
        ##能够访问
        writable=yes
        ##允许有写的权限
        create mask=0644
        ##设置权限
        directory mask=0755
...
[root@samba ~]# mkdir -p /samba/public
[root@samba ~]# chmod 777 -R /samba/public
```
####  启动服务
```javascript
[root@samba ~]# /usr/local/samba/sbin/nmbd start
[root@samba ~]# /usr/local/samba/sbin/smbd start
```
####  测试
![](https://pic.imgdb.cn/item/6094aae7d1a9ae528fabac04.png)
![](https://pic.imgdb.cn/item/6094aa99d1a9ae528fa9022e.png)
创建一个测试文件
```javascript
[root@samba ~]# cd /samba/public
[root@samba public]# ll
总用量 0
-rw-r--r--. 1 nobody nobody 0 5月   6 17:50 test.txt
```
回到Linux服务器可以看到我们共享的文件是匿名访问的
###  用户模式
####  修改配置文件
```javascript
[root@samba ~]# vim /usr/local/samba/etc/smb.conf
...
[global]
        workgroup = SAMBA
        security = user
        passdb backend = tdbsam
        ## printing = cups
        ## printcap name = cups
        ## load printers = yes
        ## cups options = raw
        ## map to guest = Bad User 
        ## 注释匿名用户访问
[private]
##添加的share文件
        path=/samba/private
        ## 路径
        ## public=yes
        ## 公共访问
        browseable=yes
        ## 能够访问
        ## writable=yes
        ##允许有写的权限
        create mask=0644
        ## 设置权限
        directory mask=0755
        valid users=shuaige liangzai    
        ## 允许访问的用户
        write list=shuaige               
        ## 允许写入的用户   
        ## hosts deny=xx.xx.xx.xx  
        ## 拒绝xx.xx.xx.xx访问    
...
[root@samba ~]# mkdir -p /samba/private
[root@samba ~]# chmod 777 -R /samba/private/
```
####  添加用户并设置密码
```javascript
[root@jjh samba]# useradd shuaige
## 创建用户
[root@jjh samba]# /usr/local/samba/bin/smbpasswd -a shuaige
## 给用户设置密码
New SMB password:
Retype new SMB password:
Added user shuaige.
[root@samba public]# useradd liangzai
[root@samba public]# /usr/local/samba/bin/smbpasswd -a liangzai
New SMB password:
Retype new SMB password:
Added user liangzai.
[root@samba public]# /usr/local/samba/bin/pdbedit -L
## 列出smb用户列表
shuaige:1000:
liangzai:1001:
```
####  启动服务
```javascript
[root@samba ~]# /usr/local/samba/sbin/nmbd start
[root@samba ~]# /usr/local/samba/sbin/smbd start
```
####  测试
![](https://pic.imgdb.cn/item/6094aab6d1a9ae528faa1603.png)
![](https://pic.imgdb.cn/item/6094aa89d1a9ae528fa86085.png)
清除登录记录
![](https://pic.imgdb.cn/item/6094aaadd1a9ae528fa9b800.png)
![](https://pic.imgdb.cn/item/6094aaa1d1a9ae528fa94b47.png)
![](https://pic.imgdb.cn/item/6094aa84d1a9ae528fa8330f.png)
可以看到用户liangzai没有写权限
```javascript
[root@samba bin]# cd /samba/private/
[root@samba private]# ll
总用量 0
-rw-r--r--. 1 shuaige shuaige 0 5月   6 18:40 shuaige.txt
```
未完待续...
