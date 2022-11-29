---
title: Docker安装部署ElK教程
date: 2020-12-3
updated:
description: 
cover: https://pic.imgdb.cn/item/6385a12316f2c2beb12e69cd.jpg
tag:
  - Elasticsearch 
  - Logstash 
  - Kibana
categories:
  - Docker
---
## 简介
### 核心组成
ELK由**Elasticsearch**，**Logstash**和**Kibana**三部分组件组成；

Elasticsearch是一个开源分布式搜索引擎，它的特点是：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

Logstash是一个完全开源的工具，它可以对您的日志进行收集，分析，将其存储供以后使用Kibana是一个开源和免费的工具，它可以为Logstash和ElasticSearch提供的日志分析友好的Web界面，可以帮助您汇总，分析和搜索重要数据日志。

### 四大组件
Logstash：logstash服务器端

Elasticsearch：存储类别日志

Kibana：网络化接口

Logstash转发器：logstash客户端端通过木材服务器网络协议发送日志到logstash服务器

### ELK工作流程
在需要收集日志的所有服务上部署logstash，作为logstash代理（logstash shipper）用于监控并过滤收集日志，将过滤后的内容发送到Redis，然后logstash indexer将日志收集在一起并提供全文搜索服务ElasticSearch，可以用ElasticSearch进行自定义搜索通过Kibana来结合自定义搜索进行页面展示。

## Docker安装ELK
参考[ELK官网](https://www.elastic.co/)
### 拉取镜像
```javascript
[root@jjh ~]# docker pull docker.elastic.co/elasticsearch/elasticsearch:7.10.0
[root@jjh ~]# docker pull docker.elastic.co/logstash/logstash:7.10.0
[root@jjh ~]# docker pull docker.elastic.co/kibana/kibana:7.10.0
[root@jjh ~]# docker images
REPOSITORY                                      TAG                 IMAGE ID            CREATED             SIZE
docker.elastic.co/logstash/logstash             7.10.0              bc71baf6997e        3 weeks ago         843 MB
docker.elastic.co/kibana/kibana                 7.10.0              da7fcd592595        3 weeks ago         1 GB
docker.elastic.co/elasticsearch/elasticsearch   7.10.0              37190fe5beea        3 weeks ago         774 MB
```
### 运行容器
```javascript
[root@jjh ~]# docker run -d --name es -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.10.0
[root@jjh ~]# docker run --name es_logstash docker.elastic.co/logstash/logstash:7.10.0
[root@jjh ~]# docker run --name es_kibana -p 5601:5601 -d -e ELASTICSEARCH_URL=http://localhost:9200 docker.elastic.co/kibana/kibana:7.10.0
[root@jjh ~]# docker ps
CONTAINER ID        IMAGE                                                  COMMAND                  CREATED             STATUS              PORTS                                            NAMES
f017ffdeb0d5        docker.elastic.co/kibana/kibana:7.10.0                 "/usr/local/bin/du..."   4 hours ago         Up 4 hours          0.0.0.0:5601->5601/tcp                           es_kibana
b7249be98850        docker.elastic.co/logstash/logstash:7.10.0             "/usr/local/bin/do..."   4 hours ago         Up 3 hours          5044/tcp, 9600/tcp                               es_logstash
1242f863cf94        docker.elastic.co/elasticsearch/elasticsearch:7.10.0   "/tini -- /usr/loc..."   4 hours ago         Up 3 hours          0.0.0.0:9200->9200/tcp, 0.0.0.0:9300->9300/tcp   es
```
###  修改配置文件
#### Elasticsearch
进入容器，修改 elasticsearch.yml 文件
```javascript
[root@1242f863cf94 elasticsearch]# cat config/elasticsearch.yml 
cluster.name: "docker-cluster"
network.host: 0.0.0.0
# 加入跨域配置
http.cors.enabled: true
http.cors.allow-origin: "*"
```
访问  http://localhost:9200  ,显示如下：
```javascript
{
  "name" : "1242f863cf94",
  "cluster_name" : "docker-cluster",
  "cluster_uuid" : "svVoSL31SwCdpvize03tGA",
  "version" : {
    "number" : "7.10.0",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "51e9d6f22758d0374a0f3f5c6e8f3a7997850f96",
    "build_date" : "2020-11-09T21:30:33.964949Z",
    "build_snapshot" : false,
    "lucene_version" : "8.7.0",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
```
####  Logstash
进入容器，修改 logstash.yml 文件
```javascript
bash-4.2$ cat config/logstash.yml 
http.host: "0.0.0.0"
xpack.monitoring.elasticsearch.hosts: "http://localhost:9200"
```
#### Kibana
进入容器，修改 logstash.yml 文件
```javascript
bash-4.4$ cat config/kibana.yml 
# ** THIS IS AN AUTO-GENERATED FILE **
#Default Kibana configuration for docker target
server.name: kibana
server.host: "0"
elasticsearch.hosts: "http://localhost:9200"
monitoring.ui.container.elasticsearch.enabled: true
```
访问  http://localhost:5601  ,显示如下：
![](https://pic.downk.cc/item/5fc8e611394ac5237869fc1a.png)
记得修改IP地址，不然你会看到访问页面出现如下几个大字：
"kibana server is not ready yet"
### ElasticSearch-Head（可选）
参考[ElasticSearch-Head](https://github.com/mobz/elasticsearch-head)
为什么要安装ElasticSearch-Head呢，原因是需要有一个管理界面进行查看ElasticSearch相关信息，用于监控 Elasticsearch 状态的客户端插件，包括数据可视化、执行增删改查操作等。
```javascript
[root@jjh ~]# docker pull mobz/elasticsearch-head:5
[root@jjh ~]# docker run -d --name es_admin -p 9100:9100 mobz/elasticsearch-head:5
```
访问  http://localhost:9100  ,显示如下：
![](https://pic.downk.cc/item/5fc8ede4394ac523786f1e09.png)
记得修改页面左上角IP地址，否则集群健康值无法测量

