---
title: Kubernetes介绍与安装
date: 2020-12-18
updated:
description:
cover: https://pic.downk.cc/item/5fdc6fba3ffa7d37b377761e.jpg
tag:
  - Kubernetes
categories:
  - Kubernetes
---
## Kubernetes 介绍
### Kubernetes 简介
Kubernetes 是 Google 在 2014 年 6 月开源的一个容器集群管理系统，使用 GO 语言开发，Kubernetes 简称 K8S。K8S 是 Google 内部一个叫 Borg 的容器集群管理系统衍生出来的，Borg已经在 Google 大规模生产运行十年之久。
K8S 主要用于自动化部署、扩展和管理容器应用，提供了资源调度、部署管理、服务发现、扩容缩容、监控等一整套功能。K8S 的目标是让部署容器化应用简单高效。2015 年 7月，Kubernetes v1.0 版本正式发布，最新版本详见[Kubernetes官网](https://kubernetes.io/zh/)
### Kubernetes 主要功能
* 数据卷
Pod 中容器之间共享数据，可以使用数据卷。
* 应用程序监控检查
容器内服务可能进程阻塞无法处理请求，可以设置监控检查策略保证应用健壮性。
* 复制应用程序实例
控制器维护着 Pod 副本数量，保证一个 Pod 或一组同类的 Pod 数量始终可用。
* 弹性伸缩
根据设定的指标（CPU 利用率）自动缩放 Pod 副本数。
* 服务发现
使用环境变量或 DNS 服务插件保证容器中程序发现 Pod 入库访问地址。
* 负载均衡
一组 Pod 副本分配一个私有的集群 IP 地址，负载均衡转发请求到后端容器。在集群内部其他 Pod 可以通过这个 cluster IP 访问应用。
* 滚动更新
更新服务不中断，一次更新一个 Pod，而不是同时删除整个服务。类似于灰度发布。
*服务编排
通过文件描述部署服务，使得应用程序部署变得更高效。
* 资源监控
node 节点组件集成 cAdvisor 资源收集工具，可通过 Heapster 汇总整个集群节点资源数据，然后存储到 InfluxDB 时序数据库，再由Grafana 展示。、
* 提供认证和授权
支持角色访问控制（RBAC 基于角色的权限访问控制 Role-Based Access Control）认证授权等策略。
### 基本对象概念
* Pod
Pod 是最小部署单元，一个 Pod 由一个或多个容器组成，Pod 中容器共享存储和网络，
在同一台 docker 主机上运行。
* Service
Service 是一个应用服务抽象，定义了 Pod 逻辑集合和访问这个 Pod 集合的策略。
Service 代理 Pod 集合对外表现是为一个访问入口，分配一个集群 IP 地址，来自这个 IP的请求将负载均衡转发后端 Pod 中的容器。
Service 通过 Label Selector 选择一组 Pod 提供服务。
* Volume
数据卷，共享 Pod 中容器使用的数据。
* Namespace
命名空间将对象逻辑上分配到不同 namespace，可以是不同的项目、用户等区分管理，并设定控制策略，从而实现多租户。
命名空间也成为虚拟集群。
* Label
标签用户区分对象（如 Pod、Service），键/值对存在；每个对象可以有多个标签，通过标签关联对象。
* ReplicaSet（RS）
下一代 Replication Controller（RC）。确保任何给定时间制定的 Pod 副本数量，并提供声明式更新等功能。
RC 与 RS 唯一区别就是 label selector 支持不同，RS 支持新的基于集合的标签，RC 仅支持基于等式的标签。推荐使用 RS，后面 RC 将可能被淘汰。
* Deployment
Deployment 是一个更高层的 API 对象，它管理 ReplicaSets，意味着可能永远都不需要直接操作 RS 对象。
* StatefulSet
StatefulSet 适合持久性的应用程序，有唯一的网络标志符（IP），持久存储，有序的部署、扩展、删除和滚动更新。
* DaemonSet
DaemonSet 确保所有（或一些）节点运行同一个 Pod，当节点加入 Kubernetes 集群中，Pod 会被调度到该节点上运行，当节点从集群中移除时，DaemonSet 的 Pod 会被删除。删除DaemonSet 会清理它所有创建的 Pod。例如监控服务，可以使用 DaemonSet 对象管理。
* Job
一次性任务，运行完成后 Pod 销毁，不再重新启动新容器。还可以任务定时运行。
### 系统架构及组件功能
#### Master 组件：
* kube-apiserver
Kubernetes API，集群的统一入口，各组件协调者，以 HTTP API 提供接口服务，所有对象资源的增删改查和监听操作都交给 APIServer 处理后再提交给 Etcd 存储。
* kube-controller-manager
处理集群中常规后台任务，一个资源对应一个控制器，而 ControllerManager 就是负责管理这些控制器的。
* kube-scheduler
根据调度算法为新创建的 Pod 选择一个 Node 节点。
#### Node 组件：
* kubelet
kubelet 是 Master 在 Node 节点上的 Agent，管理本机运行容器的生命周期，比如创建容器、Pod 挂载数据卷、下载 secret、获取容器和节点状态等工作。kubelet 将每个 Pod 转换成一组容器。
* kube-proxy
在 Node 节点上实现 Pod 网络代理，维护网络规则和四层负载均衡工作，代理 iptables做防火墙策略。
* docker 或 rocket/rkt
运行容器。
#### 第三方服务：
* etcd
分布式键值存储系统，负责集群的持久化。
用于保持集群状态，比如 Pod、Service 等对象信息。
## 使用 kubeadm 部署 kubenetes 集群
### 安装环境
```javascript
[root@jjh ~]# cat /etc/hosts
192.168.0.36 jjh1
192.168.0.42 jjh2 
192.168.0.43 jjh3
```
#### 安装 docker
```javascript
[root@jjh ~]# yum -y install yum-utils device-mapper-persistent-data lvm2
[root@jjh ~]# yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
[root@jjh ~]# yum -y install docker-ce
[root@jjh ~]# systemctl start docker
[root@jjh ~]# systemctl enable docker
```
将 docker 的镜像仓库改为国内的
```javascript
[root@jjh ~]# vim /etc/docker/daemon.json
{
 "registry-mirrors": ["https://registry.docker-cn.com"]
}
[root@jjh ~]# systemctl restart docker
```
#### 关闭 swap
```javascript
[root@jjh ~]# swapoff -a
[root@jjh ~]# echo "/usr/sbin/swapoff -a" >> /etc/rc.local
[root@jjh ~]# chmod +x /etc/rc.local
```
#### 配置内核参数
```javascript
[root@jjh ~]# vim /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_nonlocal_bind = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
[root@jjh ~]# sysctl --system
```
#### 加载必要的内核模块
```javascript
[root@jjh ~]# vim /etc/sysconfig/modules/ipvs.modules
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
[root@jjh ~]# chmod +x /etc/sysconfig/modules/ipvs.modules 
[root@jjh ~]# /etc/sysconfig/modules/ipvs.modules
```
由于我们无法访问 kubeadm 官网上提供的源，所以选择阿里的镜像站
```javascript
[root@jjh ~]# vim /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
```
### 配置 master 节点
#### 安装所需软件包
```javascript
[root@jjh1 ~]# yum install kubeadm kubectl kubelet ipvsadm -y
```
使用 kubeadm config print init-defaults > kubeadm-init.yaml 打印出默认配置，然后在根据自己的环境修改配置，尤其是镜像，在国外，默认的下不下来
```javascript
[root@jjh1 ~]# kubeadm config print init-defaults > kubeadm-init.yaml
[root@jjh1 ~]# vim kubeadm-init.yaml
```
```
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.0.36 ## master内网IP
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: jjh1
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository:  registry.cn-hangzhou.aliyuncs.com/google_containers ## 更改
kind: ClusterConfiguration
kubernetesVersion: v1.20.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
scheduler: {}
---                                 
apiVersion: kubeproxy.config.k8s.io/v1alpha1   ## 添加
kind: KubeProxyConfiguration
mode: "ipvs"
```
#### 可以预下载镜像
```javascript
[root@jjh1 ~]# kubeadm config images pull --config kubeadm-init.yaml
[root@jjh1 ~]# docker image ls
REPOSITORY                                                                    TAG        IMAGE ID       CREATED         SIZE
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-proxy                v1.20.0    10cc881966cf   9 days ago      118MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-apiserver            v1.20.0    ca9843d3b545   9 days ago      122MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-controller-manager   v1.20.0    b9fa1895dcaa   9 days ago      116MB
registry.cn-hangzhou.aliyuncs.com/google_containers/kube-scheduler            v1.20.0    3138b6e3d471   9 days ago      46.4MB
registry.cn-hangzhou.aliyuncs.com/google_containers/etcd                      3.4.13-0   0369cf4303ff   3 months ago    253MB
registry.cn-hangzhou.aliyuncs.com/google_containers/coredns                   1.7.0      bfe3a36ebd25   6 months ago    45.2MB
registry.cn-hangzhou.aliyuncs.com/google_containers/pause                     3.2        80d28bedfe5d   10 months ago   683kB
```
#### 初始化 master 节点
```javascript
[root@jjh1 ~]# systemctl start kubelet
[root@jjh1 ~]# systemctl enable kubelet
[root@jjh1 ~]# kubeadm init --pod-network-cidr=10.244.0.0/16 --image-repository=registry.cn-hangzhou.aliyuncs.com/google_containers
......
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.0.36:6443 --token xzdklu.nf4lx1djq61i2kdx \
    --discovery-token-ca-cert-hash sha256:9cc9686dda3a41b283c7182fd13df1eb98992e5d74a7c7ac03bd25b0d63f90ce 
```
//10.244.0.0/16 是 flannel 网络的默认网段

kubeadm init 主要执行了以下操作：
[init]：指定版本进行初始化操作
[preflight] ：初始化前的检查和下载所需要的 Docker 镜像文件
[kubelet-start] ：生成 kubelet 的配置文件”/var/lib/kubelet/config.yaml”，没有这个文件 kubelet无法启动，所以初始化之前的 kubelet 实际上启动失败。
[certificates]：生成 Kubernetes 使用的证书，存放在/etc/kubernetes/pki 目录中。
[kubeconfig] ：生成 KubeConfig 文件，存放在/etc/kubernetes 目录中，组件之间通信需要使用对应文件。
[control-plane]：使用/etc/kubernetes/manifest 目录下的 YAML 文件，安装 Master 组件。
[etcd]：使用/etc/kubernetes/manifest/etcd.yaml 安装 Etcd 服务。
[wait-control-plane]：等待 control-plan 部署的 Master 组件启动。
[apiclient]：检查 Master 组件服务状态。
[uploadconfig]：更新配置
[kubelet]：使用 configMap 配置 kubelet。
[patchnode]：更新 CNI 信息到 Node 上，通过注释的方式记录。
[mark-control-plane]：为当前节点打标签，打了角色 Master，和不可调度标签，这样默认就不会使用 Master 节点来运行 Pod。
[bootstrap-token]：生成 token 记录下来，后边使用 kubeadm join 往集群中添加节点时会用到
[addons]：安装附加组件 CoreDNS 和 kube-proxy
#### 为 kubectl 准备 Kubeconfig 文件
kubectl 默认会在执行的用户家目录下面的.kube 目录下寻找 config 文件。这里是将在初始化时[kubeconfig]步骤生成的 admin.conf 拷到.kube/config
```javascript
[root@jjh1 ~]# mkdir -p $HOME/.kube
[root@jjh1 ~]# cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
[root@jjh1 ~]# chown $(id -u):$(id -g) $HOME/.kube/config
```
在该配置文件中，记录了 API Server 的访问地址，所以后面直接执行 kubectl 命令就可以正常连接到 API Server 中
#### 验证 master 各组件运行状态
```javascript
[root@jjh1 ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS      MESSAGE                                                                                       ERROR
scheduler            Unhealthy   Get "http://127.0.0.1:10251/healthz": dial tcp 127.0.0.1:10251: connect: connection refused   
controller-manager   Unhealthy   Get "http://127.0.0.1:10252/healthz": dial tcp 127.0.0.1:10252: connect: connection refused   
etcd-0               Healthy     {"health":"true"}                                       
```
出现报错，解决思路：
注释掉/etc/kubernetes/manifests下的kube-controller-manager.yaml和kube-scheduler.yaml的- – port=0 
确认kube-scheduler和kube-controller-manager组件配置是否禁用了非安全端口
```javascript
[root@jjh1 ~]# cd /etc/kubernetes/manifests
[root@jjh1 manifests]# ls
etcd.yaml  kube-apiserver.yaml  kube-controller-manager.yaml  kube-scheduler.yaml
# 搜索- – port=0更改为：#- – port=0
[root@jjh1 ~]# systemctl restart kubelet
# 重启kubelet
```
重新验证一下
```javascript      
[root@jjh1 ~]# kubectl get cs
Warning: v1 ComponentStatus is deprecated in v1.19+
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok                  
scheduler            Healthy   ok                  
etcd-0               Healthy   {"health":"true"}   
[root@jjh1 ~]#  kubectl get pods -A
NAMESPACE     NAME                           READY   STATUS    RESTARTS   AGE
kube-system   coredns-54d67798b7-lfzwm       0/1     Pending   0          2m32s
kube-system   coredns-54d67798b7-n2k5n       0/1     Pending   0          2m32s
kube-system   etcd-jjh1                      1/1     Running   0          2m47s
kube-system   kube-apiserver-jjh1            1/1     Running   0          2m47s
kube-system   kube-controller-manager-jjh1   1/1     Running   0          56s
kube-system   kube-proxy-k2wwb               1/1     Running   0          2m32s
kube-system   kube-scheduler-jjh1            1/1     Running   0          41s
[root@jjh1 ~]# kubectl get nodes
NAME   STATUS     ROLES                  AGE     VERSION
jjh1   NotReady   control-plane,master   3m41s   v1.20.0
```
### 配置 node 节点
```javascript
[root@jjh2 ~]# yum install kubeadm kubectl kubelet ipvsadm -y
[root@jjh2 ~]# kubeadm join 192.168.0.36:6443 --token xzdklu.nf4lx1djq61i2kdx \
    --discovery-token-ca-cert-hash sha256:9cc9686dda3a41b283c7182fd13df1eb98992e5d74a7c7ac03bd25b0d63f90ce 
```
看见这段话表示成功：
This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.
Run 'kubectl get nodes' on the control-plane to see this node join the cluster.
回到 master 节点看看 node 信息
```javascript
[root@jjh1 ~]# kubectl get nodes
NAME   STATUS     ROLES                  AGE     VERSION
jjh1   NotReady   control-plane,master   5m21s   v1.20.0
jjh2   NotReady   <none>                 23s     v1.20.0
jjh3   NotReady   <none>                 21s     v1.20.0
```
### 部署网络插件 flannel
Master 节点 NotReady 的原因就是因为没有使用任何的网络插件，此时 Node 和 Master的连接还不正常。目前最流行的 Kubernetes 网络插件有 Flannel、Calico、Canal、Weave 这里选择使用 flannel。
在 master 节点上都有执行，执行完成后需要等 flannel 的 pods 运行起来，这需要点时间：
```javascript
[root@jjh1 ~]# kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
The connection to the server raw.githubusercontent.com was refused - did you specify the right host or port?
```
外网无法访问，就直接下到本地安装好了
链接： https://pan.baidu.com/s/13NwlWF24TUpDm23uj66V5Q
提取码： here 
```javascript
[root@jjh1 ~]# ls
flannel.yaml  kubeadm-init.yaml
[root@jjh1 ~]# kubectl apply -f flannel.yaml
podsecuritypolicy.policy/psp.flannel.unprivileged created
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRole is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRole
clusterrole.rbac.authorization.k8s.io/flannel created
Warning: rbac.authorization.k8s.io/v1beta1 ClusterRoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 ClusterRoleBinding
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.apps/kube-flannel-ds-amd64 created
daemonset.apps/kube-flannel-ds-arm64 created
daemonset.apps/kube-flannel-ds-arm created
daemonset.apps/kube-flannel-ds-ppc64le created
daemonset.apps/kube-flannel-ds-s390x created
```
现在重新 get 验证一下，看看是否安装成功
```javascript
[root@jjh1 ~]# kubectl get nodes
NAME   STATUS   ROLES                  AGE     VERSION
jjh1    Ready    control-plane,master   35m     v1.20.0
jjh2   Ready    <none>                 7m43s   v1.20.0
jjh3   Ready    <none>                 7m44s   v1.20.0
[root@jjh1 ~]# kubectl get pods -n kube-system
NAME                           READY   STATUS    RESTARTS   AGE
coredns-54d67798b7-lfzwm       1/1     Running   0          9m6s
coredns-54d67798b7-n2k5n       1/1     Running   0          9m6s
etcd-jjh1                      1/1     Running   0          9m21s
kube-apiserver-jjh1            1/1     Running   0          9m21s
kube-controller-manager-jjh1   1/1     Running   0          7m30s
kube-flannel-ds-amd64-7x9v6    1/1     Running   0          117s
kube-flannel-ds-amd64-8pxgf    1/1     Running   0          117s
kube-flannel-ds-amd64-jfpxb    1/1     Running   0          117s
kube-proxy-679sb               1/1     Running   0          4m24s
kube-proxy-8xm25               1/1     Running   0          4m26s
kube-proxy-k2wwb               1/1     Running   0          9m6s
kube-scheduler-jjh1            1/1     Running   0          7m15s
```
到此K8S基本安装完成

