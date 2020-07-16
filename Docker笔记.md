# Docker笔记

## Docker命令

### 登录和退出

```
docker login
docker logout
```

### 验证是否安装成功

```
docker version
或者
docker info
```

### docker search 

查找官方仓库中的镜像

```
docker search ubuntu
//搜索STARS超过30的
docker search -s 30 ubuntu
//显示完整镜像信息
docker search -s 30 --no-trunc ubuntu
```

### 配置国内镜像源

```
sudo mkdir -p /etc/docker
sudo vi /etc/docker/daemon.json

{
  "registry-mirrors": ["https://hxzqqggz.mirror.aliyuncs.com"]
}

sudo systemctl daemon-reload
sudo systemctl restart docker
```

### 导出和导入容器

```
导出
docker export 1e560fca3906 > ubuntu.tar
导入
cat docker/ubuntu.tar | docker import - test/ubuntu:v1
```

### 查看网络端口

```
docker port bf08b7f2cd89(容器id)
```

![image-20200711230428875](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200711230430.png)

### 查看WEB应用程序日志

```
docker logs -f bf08b7f2cd89(容器id)
或
docker logs bf08b7f2cd89(容器id)
```

### 查看WEB应用程序容器的进程

```
docker top bf08b7f2cd89(容器id)
```

### 查看 Docker 的底层信息

```
docker inspect bf08b7f2cd89(容器id)
```

### 查询最后一次创建的容器

```
docker ps -l
```

### Docker服务与防火墙设置

#### 设置开机自动启动和关闭

```
systemctl enable docker.service
systemctl disable docker.service
```

#### 启动docker服务

service 命令的用法

```
sudo service docker start
```

systemctl 命令的用法

```
sudo systemctl start docker
```

#### 关闭防火墙

```
systemctl stop firewalld
systemctl start firewalld
firewall-cmd --zone=public --add-port=5005/tcp --permanent   （--permanent永久生效，没有此参数重启后失效）
//删除端口
firewall-cmd --zone=public --remove-port=80/tcp --permanent
//查看端口
firewall-cmd --zone=public --query-port=80/tcp
//查看开启了哪些端口
firewall-cmd --list-ports
```

#### 查看开启了哪些服务

```
firewall-cmd --list-services
firewall-cmd --add-service=http
```

### Docker镜像

#### 列出本机的所有 image 文件

```
docker image ls
```

#### 删除 image 文件

```
//两种删除方式
docker image rm [imageName]
docker rmi [imageName]
//强制删除 -f
docker rmi -f [imageName]
//删除多个
docker rmi -f [imageName1] [imageName2]
//删除全部
docker rmi -f $(docker images -qa)
```

#### 显示镜像的所有id

可以通过批量获取id，实现删除镜像等操作。

```
docker images -q
```

#### 显示完整的镜像信息

```
docker images --digests --no-trunc
```

效果（显示完整的镜像id等信息）：

![image-20200715230000783](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200715230003.png)





### docker volume

#### 创建volume

```
docker volume create rosemary
```

#### 显示所有volume

```
docker volume ls
```

#### 删除volume

```
docker volume rm [volumes名字]
```

更新镜像

```
docker commit -m="has update" -a="runoob" e218edb10161 runoob/ubuntu:v2
```

### Docker容器

#### 进入容器

```
sudo docker exec -it containerID /bin/bash 
```

#### 运行image 文件

```
docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456 -d -p 3306:3306 mysql:5.7
```

前面的3306是对应的服务器端口

#### 停止和开始容器

```
docker stop mysql 
docker start mysql 
```

#### 删除容器

```
docker rm 7ac94d6f967f
```

#### 容器中登录mysql

```
mysql -u root -p "123456" 
```

#### 退出容器

```
exit
```

#### 显示后台运行的docker

```
docker attach  mysql
```

### 设置centos可以访问的端口

#### 查询端口

```
firewall-cmd --query-port=3306/tcp
firewall-cmd --zone=public --list-ports
```

#### 设置端口为可被访问

```
firewall-cmd --zone=public --add-port=3306/tcp --permanent 
firewall-cmd --add-port=3306/tcp
firewall-cmd --reload
```

#### 删除端口

```
firewall-cmd --zone=public --remove-port=80/tcp --permanent
```

## Docker 的优点

Docker 是一个用于开发，交付和运行应用程序的开放平台。

- Docker 能够将应用程序与基础架构分开，从而可以快速交付软件。
- 借助 Docker，开发者可以与管理应用程序相同的方式来管理基础架构。
- 通过利用 Docker 的方法来快速交付，测试和部署代码，开发人员可以大大减少编写代码和在生产环境中运行代码之间的延迟。

 ![image-20200713074420016](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200713074432.png)



注：

1. Docker本身并不是容器，它是创建容器的工具，是应用容器引擎。Docker就是下图中圈住的

   ![image-20200713202957412](../../../Typora/Picture/image-20200713202957412.png)

2. K8S是基于容器的集群管理平台，它的全称，是kubernetes。

3. Helm 是 Kubernetes 的包管理器。包管理器类似于我们在 Ubuntu 中使用的apt、Centos中使用的yum 或者Python中的 pip 一样，能快速查找、下载和安装软件包。

## Docker底层原理

Docker是脱胎于虚拟机

### Docker底层原理

![image-20200715221813485](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200715222031.png)



1. Docker有着比虚拟机更少的抽象层。由于Docker不需要Hypervisor实现硬件资源虚拟化，运行在Docker容器上的程序直接被使用的都是实际物理机的硬件资源。因此在CPU，内存利用率上，Docker有明显优势。
2. Docker利用的是宿主机的内核，而不需要Guest OS（客机操作系统（guest OS）是指一个安装在虚拟机上的操作系统）。当新建容器时，Docker不需要和虚拟机一样重新加载操作系统内核



### 虚拟机的缺点

1. 启动慢
2. 容易步骤多
3. 占用资源多

### Docker与虚拟机之间的不同

1. 传统虚拟机是虚拟出一套硬件后，在其上运行一个完成操作系统，在该系统上再运行所需应用程序。
2. 而容器内的应用进程直接运行于宿主的内核，容器内没有自己内核，<span style="color:red">而且也没有进行硬件虚拟</span>，因此容器要比传统虚拟机更为轻便。
3. 每个容器之间互相隔离，每个容器都有自己的文件系统，容器之间进程不会相互影响，能区分计算机资源。



### 虚拟机（VM）和Docker容器的区别

|            | Docker容器              | 虚拟机（VM）                |
| ---------- | ----------------------- | --------------------------- |
| 操作系统   | 与宿主机共享OS          | 宿主机OS上运行虚拟机OS      |
| 存储大小   | 镜像小，便于存储和传输  | 镜像庞大（vmdk，vdi等）     |
| 运行性能   | 几乎无额外性能损失      | 操作系统额外的CPU，内存消耗 |
| 移植性     | 轻便，灵活，适应于Linux | 笨重，与虚拟化技术耦合度高  |
| 硬件亲和力 | 面向软件开发者          | 面向硬件运维者              |
| 部署速度   | 快速，秒级              | 较慢，10s以上               |



## Docker 架构

Docker 包括三个基本概念:

- 镜像（Image）：Docker 镜像（Image），就相当于是一个 root 文件系统。比如官方镜像 ubuntu:16.04 就包含了完整的一套 Ubuntu16.04 最小系统的 root 文件系统。
- 容器（Container）：镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的类和实例一样，镜像是静态的定义，容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等。
- 仓库（Repository）：仓库可看成一个代码控制中心，用来保存镜像。

 

## Docker参数

- -t: 在新容器内指定一个伪终端或终端。
- -i: 允许你对容器内的标准输入 (STDIN) 进行交互。
- -d：后台运行。
- -P:将容器内部使用的网络端口随机映射到我们使用的主机上。
- -p : 是容器内部端口绑定到指定的主机端口。
- -f: 让 docker logs 像使用 tail -f 一样来输出容器内部的标准输出。
- -m: 提交的描述信息。
- -a: 指定镜像作者。
- --name：--name 用于命名容器。

 

##  容器

### 容器定义

1. Docker利用容器独立运行一个或一组应用。容器是用镜像创建的运行实例。
2. 每个容器都是相互隔离的、保证安全的平台。
3. 可以把容器看做一个简易版的Linux环境（包括用户权限、进程空间等）和运行在其中的应用程序。
4. 容器的定义和镜像就是一模一样的，唯一的区别在于容器的最上面那一层是可读可写的。

### **普通运行**

docker run ubuntu:15.10 /bin/echo "Hello world"

### **后台运行**

```
docker run -d ubuntu:15.10 /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

### **容器详情**

[root@iz2zeczghmsqk5hx2uvkuwz bin]# docker ps

CONTAINER ID   IMAGE   COMMAND   CREATED   STATUS  PORTS   NAMES

- CONTAINER ID: 容器 ID。
- IMAGE: 使用的镜像。
- COMMAND: 启动容器时运行的命令。
- CREATED: 容器的创建时间。
- STATUS: 容器状态。

状态有7种：

created（已创建）

restarting（重启中）

running（运行中）

removing（迁移中）

paused（暂停）

exited（停止）

dead（死亡）

- PORTS: 容器的端口信息和使用的连接类型（tcp\udp）。
- NAMES: 自动分配的容器名称。

## 仓库

![image-20200713201843455](../../../Typora/Picture/image-20200713201843455.png)







## Docker帮助

**查看****Docker****总体帮助**

```
runoob@runoob:~# docker
```

**查看具体的命令帮助（以查看**stats**帮助）**

```
runoob@runoob:~# docker stats --help
```

 

## 镜像（images）

```
runoob@runoob:~$ docker images           
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              14.04               90d5884b1ee0        5 days ago          188 MB
php                 5.6                 f40e9e0f10c8        9 days ago          444.8 MB
nginx               latest              6f8d099c3adc        12 days ago         182.7 MB
mysql               5.6                 f2e8d6c772c0        3 weeks ago         324.6 MB
httpd               latest              02ef73cf1bc0        3 weeks ago         194.4 MB
ubuntu              15.10               4e3b13c8a266        4 weeks ago         136.3 MB
hello-world         latest              690ed74de00f        6 months ago        960 B
training/webapp     latest              6fae60ef3446        11 months ago       348.8 MB
```

镜像详情：

- REPOSITORY：表示镜像的仓库源
- TAG：镜像的标签
- IMAGE ID：镜像ID
- CREATED：镜像创建时间
- SIZE：镜像大小

## 构建镜像

我们使用命令 docker build ， 从零开始来创建一个新的镜像。为此，我们需要创建一个 Dockerfile 文件，其中包含一组指令来告诉 Docker 如何构建我们的镜像。

```
runoob@runoob:~$ cat Dockerfile 
FROM    centos:6.7
MAINTAINER      Fisher "fisher@sudops.com"
 
RUN     /bin/echo 'root:123456' |chpasswd
RUN     useradd runoob
RUN     /bin/echo 'runoob:123456' |chpasswd
RUN     /bin/echo -e "LANG=\"en_US.UTF-8\"" >/etc/default/local
EXPOSE  22
EXPOSE  80
CMD     /usr/sbin/sshd -D
```

每一个指令都会在镜像上创建一个新的层，每一个指令的前缀都必须是大写的。

第一条FROM，指定使用哪个镜像源

RUN 指令告诉docker 在镜像内执行命令，安装了什么。。。

然后，我们使用 Dockerfile 文件，通过 docker build 命令来构建一个镜像。

```
runoob@runoob:~$ docker build -t runoob/centos:6.7 .
Sending build context to Docker daemon 17.92 kB
Step 1 : FROM centos:6.7
 ---&gt; d95b5ca17cc3
Step 2 : MAINTAINER Fisher "fisher@sudops.com"
 ---&gt; Using cache
 ---&gt; 0c92299c6f03
Step 3 : RUN /bin/echo 'root:123456' |chpasswd
 ---&gt; Using cache
 ---&gt; 0397ce2fbd0a
Step 4 : RUN useradd runoob
......
```

参数说明：

- -t ：指定要创建的目标镜像名
- . ：Dockerfile 文件所在目录，可以指定Dockerfile 的绝对路径

为镜像设置标签（tag）

```
runoob@runoob:~$ docker tag 860c279d2fec runoob/centos:dev
```

 

## Docker 容器互联

端口映射并不是唯一把 docker 连接到另一个容器的方法。

docker 有一个连接系统允许将多个容器连接在一起，共享连接信息。

docker 连接会创建一个父子关系，其中父容器可以看到子容器的信息。

 

## Docker Machine

Docker Machine 是一种可以让您在虚拟主机上安装 Docker 的工具，并可以使用 docker-machine 命令来管理主机。

Docker Machine 也可以集中管理所有的 docker 主机，比如快速的给 100 台服务器安装上 docker。

 

## Swarm 集群管理

Docker Swarm 是 Docker 的集群管理工具。它将 Docker 主机池转变为单个虚拟 Docker 主机。 Docker Swarm 提供了标准的 Docker API，所有任何已经与 Docker 守护程序通信的工具都可以使用 Swarm 轻松地扩展到多个主机。













###



