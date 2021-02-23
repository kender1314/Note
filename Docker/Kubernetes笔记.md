#  Kubernetes笔记

## Kubernetes简介

Kubernetes是用于自动部署，扩展和管理容器化应用程序的开源系统。

### 同类产品

MESOS，Docker Swarm，borg（Kubernetes的前身）



### 特点

1. 轻量级：消耗资源小
2. 开源
3. 弹性伸缩：方便添加或减少服务器
4. 负载均衡：IPVS



### Helm介绍

相当于Linux中的yum，可以一键部署mongoDB的集群等等。。。。



## Kubernetes相关架构

### borg架构

![image-20210109110015831](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/image-20210109110015831.png)

注：

1. borg主要分为Borgmaster和Borglet，Borgmaster包含scheduler等
2. scheduler主要是分发请求，它不会直接与Borglet进行交互，而是将请求写入Paxos键值对数据库中，Borglet会实时监听Paxos，如果有自己的请求，就会取出来消费
3. Borgmaster数量一般是单数（防止脑裂），并且大于1（防止单节点故障）
4. Borglet是一个个计算节点



### Kubernetes架构

![image-20210109110523429](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/image-20210109110523429.png)

注：

1. 与borg稍有不同，Kubernetes中的scheduler不会直接将任务写入数据中，而是交给api server再写入数据库中
2. api server所有服务的统一入口
3. 副本控制器的作用是控制容器副本的数量，如果容器的副本数量不符合副本控制器的期望，则会控制副本数量以符合期望
4. etcd 是 CoreOS 团队于 2013 年 6 月发起的开源项目，它的目标是构建一个高可用的分布式键值(key-value)数据库。
5. kubelet会与docker进行交互，操作docker操作对应的容器
6. kube proxy：负责写入规则至IPTABLES、IPVS实现服务映射访问

其他的插件

1. CoreDNS：可以为集群中的SVC创建一个域名IP的对应解析关系（说白了就是为ip端口映射域名）
2. DashBoard：为k8s提供B/S模式展示
3. Ingress Controller：官方k8s只能实现4层代理，Ingress 可以实现7层代理
4. Fedetation：可以跨k8s集群进行统一管理
5. Prometheus：提供k8s集群的监控



### ETCD架构

![image-20210109111546450](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/image-20210109111546450.png)

注：

1. Raft存储数据，并且实时将数据存储到内存中
2. 读写日志会记录用户的操作，并且定时生成数据集，存储起来

## Pod

### Pod简介









































###