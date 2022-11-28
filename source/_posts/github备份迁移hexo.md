---
title: GitHub备份迁移Hexo
sticky: 100
date: 2020-11-12
updated:
description:  备个份，说不定哪天就丢了。。。
cover: https://pic.imgdb.cn/item/6384a20416f2c2beb1d0c290.jpg
tag:
  - Hexo
categories:
  - GitHub
---
## 创建备份文件
先新建一个hexo文件夹，作为分支的工作目录，用于保存将要备份的文件和文件夹
```javascript
[root@jjh ~]# mkdir hexo
```
把GitHub上的Hexo仓库clone到hexo文件夹中
```javascript
[root@jjh ~]# git clone https://github.com/hehe-tui/hehe-tui.github.io hexo
```
删除**除了.git文件夹**的其它所有文件和文件夹，主要是为了得到版本管理的.git。下面命令不会删除隐藏文件和文件夹。
```javascript
[root@jjh ~]# cd hexo
[root@jjh hexo]# rm -r *
```
创建.gitignore文件
```javascript
[root@jjh hexo]# touch .gitignore
```
最后把需要备份的文件和文件夹都复制到hexo文件夹下，hexo的目录结构应该如下
```javascript
[root@jjh hexo]# ll -a
total 32
drwxr-xr-x  6 root root 4096 Oct 27 09:57 .
dr-xr-x---. 8 root root 4096 Oct 27 10:00 ..
-rw-r--r--  1 root root 3210 Oct 27 09:56 _config.yml
drwxr-xr-x  8 root root 4096 Oct 27 10:02 .git
-rw-r--r--  1 root root    0 Oct 27 09:56 .gitignore
-rw-r--r--  1 root root  842 Oct 27 09:57 package.json
drwxr-xr-x  2 root root 4096 Oct 27 09:56 scaffolds
drwxr-xr-x  6 root root 4096 Oct 27 09:57 source
drwxr-xr-x  4 root root 4096 Oct 27 09:56 themes
```
如果使用的主题是从GitHub克隆的，那么主题文件夹下有Git管理文件，需要将它们移除，否则上传github会为空目录。我使用的是hexo-butterfly，需要移除的文件如下
```javascript
[root@jjh butterfly]# rm -rf .git*
```

## 创建分支
创建一个叫hexo的分支
```javascript
[root@jjh hexo]# git checkout -b hexo
```
保存所有文件到暂存区
```javascript
[root@jjh hexo]# git add --all
```
提交变更
```javascript
[root@jjh hexo]# git commit -m "创建hexo分支"
```
推送到GitHub，并用--set-upstream与origin创建关联，将hexo设置为默认分支
```javascript
[root@jjh hexo]# git push --set-upstream origin hexo
```

##  迁移博客
预先安装环境
将hexo分支克隆下来
```javascript
[root@jjh ~]# git clone -b hexo https://github.com/hehe-tui/hehe-tui.github.io
```
然后安装Hexo依赖
```javascript
npm install
```
启动服务，访问 http://localhost:4000 判断备份是否成功。

