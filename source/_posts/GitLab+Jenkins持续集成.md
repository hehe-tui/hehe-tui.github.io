---
title: GitLab+Jenkins持续集成
date: 2020-11-28
updated:
description:  
cover: https://pic.downk.cc/item/5fc1f86115e7719084aa791f.jpg
tag:
  - Jenkins 
  - GitLab
categories:
  - Docker
---
**预安装Docker环境**
**记得更改国内镜像源**
## Docker安装Jenkins
参考[官方Docker使用说明](https://hub.docker.com/r/jenkins/jenkins/)
### 拉取 Jenkins镜像
```javascript
[root@jjh ~]# docker pull jenkins/jenkins
```
### 创建本地Jenkins配置文件目录
```javascript
[root@jjh ~]# mkdir /var/jenkins_home
[root@jjh ~]# chmod 777 /var/jenkins_home  ##注意这里必须配置本地卷的权限，否则启动失败
```
### 运行Jenkins镜像
```javascript
[root@jjh ~]#  docker run -d -v /var/jenkins_home:/var/jenkins_home -p 8080:8080 --name jenkins docker.io/jenkins/jenkins
7917d5151c9d1b66c2fcbec4e32bd573565fb4c4ff6ef51a1546f883d7ce3a22
[root@jjh ~]# docker ps
CONTAINER ID        IMAGE                       COMMAND                  CREATED             STATUS              PORTS                               NAMES
7917d5151c9d        docker.io/jenkins/jenkins   "/sbin/tini -- /us..."   21 seconds ago      Up 20 seconds       0.0.0.0:8080->8080/tcp, 50000/tcp   jenkins
```
#-d：后台运行
#-p：将容器内部端口向外映射
#--name：命名容器名称
#-v：将容器内数据文件夹或者日志、配置等文件夹挂载到宿主机指定目录
## Docker安装GitLab
### 拉取 GitLab镜像
```javascript
[root@jjh ~]# docker pull gitlab/gitlab-ce
```
### 创建本地GitLab配置文件目录
通常会将 GitLab 的配置 (etc) 、 日志 (log) 、数据 (data) 放到容器之外， 便于日后升级， 因此请先准备这三个目录。
```javascript
[root@jjh ~]# mkdir -p /var/gitlab/etc
[root@jjh ~]# mkdir -p /var/gitlab/log
[root@jjh ~]# mkdir -p /var/gitlab/data
```
### 运行GitLab镜像并修改配置文件
```javascript
[root@jjh ~]# docker run -d -p 443:443 -p 222:22 -p 8090:80  --name gitlab -v /var/gitlab/etc:/etc/gitlab -v /var/gitlab/log:/var/log/gitlab -v /var/gitlab/data:/var/opt/gitlab docker.io/gitlab/gitlab-ce
```
在gitlab上创建项目的时候，生成项目的URL访问地址是按容器的hostname来生成的，也就是容器的id。作为gitlab服务器，我们需要一个固定的URL访问地址，于是需要配置gitlab.rb
```javascript
[root@jjh ~]#  vim /var/gitlab/etc/gitlab.rb
# 配置http协议所使用的访问地址,不加端口号默认为80
external_url 'http://localhost'

# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = 'localhost'
gitlab_rails['gitlab_shell_ssh_port'] = 222 # 此端口是run时22端口映射的222端口
```
## 搭建GitLab+Jenkins持续集成环境
### 创建GitLab项目组
打开 http://localhost:8090 ，登录进去（详见下面），点击new group创建一个项目组，信息可以自己看着填
![](https://pic.downk.cc/item/5fc26697d590d4788a82467b.png)
### 获取GitLab个人访问令牌
打开GitLab，点击“setting”——“Account”，复制“Private token”备用，如下所示：
![](https://pic.downk.cc/item/5fc2718cd590d4788a86eadf.png)
### 创建Jenkins Job
打开 http://localhost:8080 ,“item name”可以随便起，然后点击“构建一个自由风格的软件项目”
![](https://pic.downk.cc/item/5fc271c1d590d4788a870281.png)
### Jenkins安装Git GitLab插件
切换到“可选插件”，分别搜索“GitLab Plugin”和“Git Plugin”,然后点击“直接安装”。如果在“可选插件”里没有搜到，可能默认你已经安装了，可以在“已安装”里查看
![](https://pic.downk.cc/item/5fc271f8d590d4788a871adf.png)
### 配置Jenkins设定系统
配置GitLab，”Connection Name”随便填，“Git Host URL”填GitLab的访问地址，然后点“Add”——“jenkins”，如下所示：
![](https://pic.downk.cc/item/5fc2725ad590d4788a874229.png)
设置完了，要测试一下能否连接成功，点击“test connection”,要看到返回“Success”才行
### 配置Job
#### 配置Job的源码管理
选择“源码管理”，选择“Git”,然后去GitLab中复制项目地址，粘贴到“Repository URL”,然后点击“credentials”后面的“Add”按钮
![](https://pic.downk.cc/item/5fc272bed590d4788a876d73.png)
之前连接存储库一直失败，后来发现是端口出错，这个要注意一下
#### 配置Job的构建触发器
选择“构建触发器”，勾选“Pull SCM”，这个选项会每隔一段时间检查一下GitLab仓库中代码是否有更新，有的话就执行构建操作。日程表如何设置，在这个输入框下面有说明。
![](https://pic.downk.cc/item/5fc27310d590d4788a879849.png)
点击进阶生成秘密令牌
![](https://pic.downk.cc/item/5fc27352d590d4788a87b838.png)
![](https://pic.downk.cc/item/5fc27374d590d4788a87c5c8.png)
返回gitlab界面贴上令牌和job的url
![](https://pic.downk.cc/item/5fc27391d590d4788a87d318.png)
#### 配置Job的构建脚本
在build栏目里，选择“jenkins execute shell”,然后输入你项目的构建命令（这依赖于你的项目，如Maven的maven build，gulp的gulp xxx 等等）
![](https://pic.downk.cc/item/5fc273f1d590d4788a87fbfe.png)
jenkins支持多种构建脚本，可以自己试一下
#### 将构建状态推送回git
点击“进阶”，填写“build name”
![](https://pic.downk.cc/item/5fc27410d590d4788a8809f2.png)
点击“应用”，点击“保存”。
至此，所有工作已经完成，现在你提交代码到GitLab，jenkins会每隔一段时间帮你运行一次构建命令，这样大家的代码自动集成到一起，出了错的话很快就知道了。
