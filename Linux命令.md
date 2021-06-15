# Linux命令

## Centos配置网络

### 配置NAT静态网络

Centos安装后，IP是动态的，下次重启系统后，IP地址也会变化，这时候我们可以把系统的IP设置为静态的，设置步骤如下：
（1）点击VMware虚拟机左上角的“编辑”，选择“虚拟网络编译器”。
（2）选中VMnet8（NAT模式），再点击右侧的“NAT设置”此时会看到如下界面

![这里写图片描述](https://img-blog.csdn.net/20180804201749749?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FraXBhMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（3）在命令行中输入：vim /etc/sysconfig/network-scripts/ifcfg-ens33

（5）将ONBOOT=no改为yes，将BOOTPROTO=dhcp改为BOOTPROTO=static,并在后面增加几行内容：

```
IPADDR=192.168.127.128
NETMASK=255.255.255.0
GATEWAY=192.168.127.2
DNS1=119.29.29.29
```

![这里写图片描述](https://img-blog.csdn.net/20180804202050977?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2FraXBhMTE=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

（6）保存后退出，然后输入命令：systemctl restart network.service来重启网络服务。

### 配置挂起重启后正常访问端口

```
sudo vi /usr/lib/sysctl.d/00-system.conf
```

在最后添加 IPv4转发状态

```
net.ipv4.ip_forward = 1
```

## centos7调整分区大小

查看磁盘的空间大小： df -h

备份/home : cp -r /home/ homebak/

卸载 /home ： umount /home

如果出现 home 存在进程，使用 fuser -m -v -i -k /home 终止 home 下的进程，最后使用 umount /home 卸载 /home

删除/home所在的lv ： lvremove /dev/mapper/centos-home

扩展/root所在的lv，增加4430G ： lvextend -L +4430G /dev/mapper/centos-root

扩展/root文件系统 ： `xfs_growfs /dev/mapper/centos-root`

重新创建`home lv` ： `lvcreate -L 167G -n home centos`

重新创建`home lv` 分区的大小，根据 vgdisplay 中的free PE 的大小确定

创建文件系统： `mkfs.xfs /dev/centos/home`

挂载 `home`： `mount /dev/centos/home /home`





## Linux垃圾清理

### df显示磁盘剩余空间

```
df -h：查看磁盘信息
df -hl：查看磁盘剩余空间
df -h：查看每个根路径的分区大小
du -sh [目录名]：返回该目录的大小
du -sm [文件夹]：返回该文件夹总M数
du -h [目录名]：查看指定文件夹下的所有文件大小（包含子文件夹）
```

### 删除缓存

#### 非常有用的清理命令：

```
sudo apt-get autoclean        清理旧版本的软件缓存
sudo apt-get clean          清理所有软件缓存
sudo apt-get autoremove       删除系统不再使用的孤立软件
```

#### 清理opera firefox的缓存文件：

```
ls ~/.opera/cache4
ls ~/.mozilla/firefox/*.default/Cache
```

#### 清理Linux下孤立的包：

终端命令下我们可以用：

```
sudo apt-get install deborphan -y
```

#### 删除软件

ubuntu软件的删除一般用“ubuntu软件中心”或“新立得”就能搞定，但有时用命令似乎更快更好～～

```
sudo apt-get remove --purge 软件名
sudo apt-get autoremove                            删除系统不再使用的孤立软件
sudo apt-get autoclean                              清理旧版本的软件缓存
dpkg -l |grep ^rc|awk '{print $2}' |sudo xargs dpkg -P       清除残余的配置文件
```



## yum安装程序

### 安装

```
sudo yum install <软件名>
```

### 在yum仓库中查看软件列表

```
yum list docker-ce --showduplicates | sort -r
```

### 启动、设置开启开机启动

```
sudo systemctl start docker
sudo systemctl enable docker
```

### 查看启动状态

```
systemctl status docker
```

### 卸载程序

查询docker安装过的包：

```
yum list installed | grep docker
```

删除安装包：

```
yum remove docker-ce.x86_64 ddocker-ce-cli.x86_64 -y
```

删除镜像/容器等：

```
rm -rf /var/lib/docker
```

## **linux系统信息**

**Linux查看当前操作系统版本信息** 

```
cat /proc/version
```

**Linux查看版本当前操作系统内核信息** 

```
uname -a
```

**linux查看版本当前操作系统发行信息**

```
cat /etc/issue 或 
cat /etc/centos-release
```

**Linux查看cpu相关信息，包括型号、主频、内核信息等**

```
 cat /etc/cpuinfo
```

## **uname命令**

uname命令用于打印当前系统相关信息（内核版本号、硬件架构、主机名称和操作系统类型等）。

uname -a显示全部信息

-m或--machine：显示电脑类型；

-r或--release：显示操作系统的发行编号；

-s或--sysname：显示操作系统名称；

-v：显示操作系统的版本；

-p或--processor：输出处理器类型或"unknown"；

-i或--hardware-platform：**输出硬件平台或"unknown"；** 

-o或--operating-system：输出操作系统名称；

--help：显示帮助；

--version：显示版本信息。





## 关机与重启命令

Linux centos重启命令：
　　1、reboot  普通重启
　　2、shutdown -r now 立刻重启(root用户使用)
　　3、shutdown -r 10 过10分钟自动重启(root用户使用)
　　4、shutdown -r 20:35 在时间为20:35时候重启(root用户使用)
　　如果是通过shutdown命令设置重启的话，可以用shutdown -c命令取消重启
　Linux centos关机命令：
　　1、halt 立刻关机
　　2、poweroff 立刻关机
　　3、shutdown -h now 立刻关机(root用户使用)
　　4、shutdown -h 10 10分钟后自动关机
　　如果是通过shutdown命令设置关机的话，可以用shutdown -c命令取消重启

## chmod命令

```
1.更改目录所有者命令:
chown -R root /data/gitlab
2.更改目录权限命令:
chmod -R 755 /data/gitlab
chmod -R 1000 /data/gitlab
3、查看文件夹的权限
ls -la /data/gitlab
4.脚本执行权限？
chmod 777 ./service-demo.sh
```

```
-rw------- (600)      只有拥有者有读写权限。
-rw-r--r-- (644)      只有拥有者有读写权限；而属组用户和其他用户只有读权限。
-rwx------ (700)     只有拥有者有读、写、执行权限。
-rwxr-xr-x (755)    拥有者有读、写、执行权限；而属组用户和其他用户只有读、执行权限。
-rwx--x--x (711)    拥有者有读、写、执行权限；而属组用户和其他用户只有执行权限。
-rw-rw-rw- (666)   所有用户都有文件读、写权限。
-rwxrwxrwx (777)  所有用户都有读、写、执行权限。
```



## cp命令

```
 cp [选项] 源文件 目标文件
-a：相当于 -d、-p、-r 选项的集合，这几个选项我们一一介绍；
-d：如果源文件为软链接（对硬链接无效），则复制出的目标文件也为软链接；
-i：询问，如果目标文件已经存在，则会询问是否覆盖；
-l：把目标文件建立为源文件的硬链接文件，而不是复制源文件；
-s：把目标文件建立为源文件的软链接文件，而不是复制源文件；
-p：复制后目标文件保留源文件的属性（包括所有者、所属组、权限和时间）；
-r：递归复制，用于复制目录；
-u：若目标文件比源文件有差异，则使用该选项可以更新目标文件，此选项可用于对文件的升级和备用。
```



## jar命令

```
当前ssh窗口被锁定，可按CTRL + C打断程序运行，或直接关闭窗口，程序退出
java -jar XXX.jar
```

```
&代表在后台运行。当前ssh窗口不被锁定，但是当窗口关闭时，程序中止运行。
java -jar XXX.jar &
```

```
nohup 意思是不挂断运行命令,当账户退出或终端关闭时,程序仍然运行
nohup java -jar XXX.jar &
```

```
将command的输出重定向到out.file文件，即输出内容不打印到屏幕上，而是输出到out.file文件中。
nohup java -jar XXX.jar >temp.txt &
输出错误的解决方法
nohup java -jar springboot-1.0.0.jar > master.log 2>&1 &
```

```
通过jobs命令查看后台运行任务
jobs
```

```
如果想将某个作业调回前台控制，只需要 fg + 编号即可
fg 23
```



## tail命令

```
-f 循环读取
-q 不显示处理信息
-v 显示详细的处理信息
-c<数目> 显示的字节数
-n<行数> 显示文件的尾部 n 行内容
--pid=PID 与-f合用,表示在进程ID,PID死掉之后结束
-q, --quiet, --silent 从不输出给出文件名的首部
-s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒
```

```
显示 notes.log 文件的最后 10 行
tail notes.log
```

```
跟踪名为 notes.log 的文件的增长情况
tail -f notes.log
```

```
显示文件 notes.log 的内容，从第 20 行至文件末尾:
tail +20 notes.log
```

```
显示文件 notes.log 的最后 10 个字符:
tail -c 10 notes.log
```

## netstat和lsof命令

```
查询端口占用情况
lsof -i:端口号
或
netstat -tunlp | grep 端口号
```

```
lsof -i:8080：查看8080端口占用
lsof abc.txt：显示开启文件abc.txt的进程
lsof -c abc：显示abc进程现在打开的文件
lsof -c -p 1234：列出进程号为1234的进程所打开的文件
lsof -g gid：显示归属gid的进程情况
lsof +d /usr/local/：显示目录下被进程开启的文件
lsof +D /usr/local/：同上，但是会搜索目录下的目录，时间较长
lsof -d 4：显示使用fd为4的进程
lsof -i -U：显示所有打开的端口和UNIX domain文件
```

```
杀死端口对应的进程
kill -9 PID
```

## history

```
查看历史命令

history
```

## adduser创建用户



```
adduser kender
passwd kender
```

## time 命令

如果你想查看一条命令（比如 ls）到底执行了多长时间，我们可以这样做：

```
[roc@roclinux ~]$ time ls
program  public_html  repo  rocscm
 
real    0m0.002s
user    0m0.002s
sys 0m0.000s
```

real、user 和 sys，它们都代表什么含义呢？哪个才是 ls 命令的执行时间呢？

- real：从进程 ls 开始执行到完成所耗费的 CPU 总时间。该时间包括 ls 进程执行时实际使用的 CPU 时间，ls 进程耗费在阻塞上的时间（如等待完成 I/O 操作）和其他进程所耗费的时间（Linux 是多进程系统，ls 在执行过程中，可能会有别的进程抢占 CPU）。
- user：进程 ls 执行用户态代码所耗费的 CPU 时间。该时间仅指 ls 进程执行时实际使用的 CPU 时间，而不包括其他进程所使用的时间和本进程阻塞的时间。
- sys：进程 ls 在内核态运行所耗费的 CPU 时间，即执行内核系统调用所耗费的 CPU 时间。

## make命令

### make概述

管理员用它通过命令行来编译和安装很多开源的工具，程序员用它来管理他们大型复杂的项目编译问题。

### make如何工作

当 make 命令第一次执行时，它扫描 Makefile 找到目标以及其依赖。如果这些依赖自身也是目标，继续为这些依赖扫描 Makefile 建立其依赖关系，然后编译它们。

### 编译

为了编译整个工程，你可以简单的使用 `make` 或者在 make 命令后带上目标 `all`。

```
make 
或
make all
```

### 通过 -B 选项让所有目标总是重新建立

```
make -B
```

### 使用 -d 选项打印调试信息

```
make -d | more
```

## vim和vi

按下Esc进入控制模式

1. u 表示撤销上一步命令；
2. Ctr+r 表示恢复上一步被撤销的命令；
3. /内容：表示查询指定内容



## rename命令

将main1.c重命名为main.c

```
rename main1.c main.c main1.c
```

**rename支持通配符**

```
?  可替代单个字符
*  可替代多个字符
[charset]  可替代charset集中的任意单个字符
```

**rename支持正则表达式**

字母的替换

```
rename "s/AA/aa/" *  //把文件名中的AA替换成aa
```

修改文件的后缀

```
rename "s//.html//.php/" *     //把.html 后缀的改成 .php后缀
```

批量添加文件后缀

```
rename "s/$//.txt/" *     //把所有的文件名都以txt结尾
```

批量删除文件名

```
rename "s//.txt//" *      //把所有以.txt结尾的文件名的.txt删掉
```



## nohup和&后台运行

### **&**

当在前台运行某个作业时，终端被该作业占据；可以在命令后面加上& 实现后台运行。例如：sh test.sh &
适合在后台运行的命令有find、费时的排序及一些shell脚本。在后台运行作业时要当心：需要用户交互的命令不要放在后台执行，因为这样你的机器就会在那里傻等。不过，作业在后台运行一样会将结果输出到屏幕上，干扰你的工作。如果放在后台运行的作业会产生大量的输出，最好使用下面的方法把它的输出重定向到某个文件中：

```
command > out.file 2>&1 &
```

这样，所有的标准输出和错误输出都将被重定向到一个叫做out.file 的文件中。
PS：当你成功地提交进程以后，就会显示出一个进程号，可以用它来监控该进程，或杀死它。(ps -ef | grep 进程号 或者 kill -9 进程号）

### **nohup**

使用&命令后，作业被提交到后台运行，当前控制台没有被占用，但是一但把当前控制台关掉(退出帐户时)，作业就会停止运行。nohup命令可以在你退出帐户之后继续运行相应的进程。nohup就是不挂起的意思( no hang up)。该命令的一般形式为：

```
nohup command &
```

如果使用nohup命令提交作业，那么在缺省情况下该作业的所有输出都被重定向到一个名为nohup.out的文件中，除非另外指定了输出文件：

```
nohup command > myout.file 2>&1 &
```

使用了nohup之后，很多人就这样不管了，其实这样有可能在当前账户非正常退出或者结束的时候，命令还是自己结束了。所以在使用nohup命令后台运行命令之后，需要使用exit正常退出当前账户，这样才能保证命令一直在后台运行。

ctrl + z 可以将一个正在前台执行的命令放到后台，并且处于暂停状态。

Ctrl+c 终止前台命令。

### **jobs**

查看当前有多少在后台运行的命令。
jobs -l  选项可显示所有任务的PID，jobs的状态可以是running, stopped, Terminated。但是如果任务被终止了（kill），shell 从当前的shell环境已知的列表中删除任务的进程标识。

### 案例

1. nohup command > myout.file 2>&1 &  

在上面的例子中，0 – stdin (standard input)，1 – stdout (standard output)，2 – stderr (standard error) ；

2>&1是将标准错误（2）重定向到标准输出（&1），标准输出（&1）再被重定向输入到myout.file文件中。

2. 0 22 * * * /usr/bin/python /home/pu/download_pdf/download_dfcf_pdf_to_oss.py > /home/pu/download_pdf/download_dfcf_pdf_to_oss.log 2>&1

这是放在crontab中的定时任务，晚上22点时候怕这个任务，启动这个python的脚本，并把日志写在download_dfcf_pdf_to_oss.log文件中

## tar命令

**语法** 

```
tar(选项)(参数)
```

**选项** 

```
-A或--catenate：新增文件到以存在的备份文件；
-B：设置区块大小；
-c或--create：建立新的备份文件；
-C <目录>：这个选项用在解压缩，若要在特定目录解压缩，可以使用这个选项。
-d：记录文件的差别；
-x或--extract或--get：从备份文件中还原文件；
-t或--list：列出备份文件的内容；
-z或--gzip或--ungzip：通过gzip指令处理备份文件；
-Z或--compress或--uncompress：通过compress指令处理备份文件；
-f<备份文件>或--file=<备份文件>：指定备份文件；
-v或--verbose：显示指令执行过程；
-r：添加文件到已经压缩的文件；
-u：添加改变了和现有的文件到已经存在的压缩文件；
-j：支持bzip2解压文件；
-v：显示操作过程；
-l：文件系统边界设置；
-k：保留原有文件不覆盖；
-m：保留文件不被覆盖；
-w：确认压缩文件的正确性；
-p或--same-permissions：用原来的文件权限还原文件；
-P或--absolute-names：文件名使用绝对名称，不移除文件名称前的“/”号；
-N <日期格式> 或 --newer=<日期时间>：只将较指定日期更新的文件保存到备份文件里；
--exclude=<范本样式>：排除符合范本样式的文件。
```

**实例** 

将文件全部打包成tar包：

```
tar -cvf log.tar log2012.log    仅打包，不压缩！ 
tar -zcvf log.tar.gz log2012.log   打包后，以 gzip 压缩 
tar -jcvf log.tar.bz2 log2012.log  打包后，以 bzip2 压缩 
```

在选项`f`之后的文件档名是自己取的，我们习惯上都用 .tar 来作为辨识。 如果加`z`选项，则以.tar.gz或.tgz来代表gzip压缩过的tar包；如果加`j`选项，则以.tar.bz2来作为tar包名。

**查阅上述tar包内有哪些文件**：

```
tar -ztvf log.tar.gz
```

由于我们使用 gzip 压缩的log.tar.gz，所以要查阅log.tar.gz包内的文件时，就得要加上`z`这个选项了。

**将tar包解压缩**：

```
tar -zxvf /opt/soft/test/log.tar.gz
```

在预设的情况下，我们可以将压缩档在任何地方解开的

**只将tar内的部分文件解压出来**：

```
tar -zxvf /opt/soft/test/log30.tar.gz log2013.log
```

我可以透过`tar -ztvf`来查阅 tar 包内的文件名称，如果单只要一个文件，就可以透过这个方式来解压部分文件！

**文件备份下来，并且保存其权限**：

```
tar -zcvpf log31.tar.gz log2014.log log2015.log log2016.log
```

这个`-p`的属性是很重要的，尤其是当您要保留原本文件的属性时。

**在文件夹当中，比某个日期新的文件才备份**：

```
tar -N "2012/11/13" -zcvf log17.tar.gz test
```

**备份文件夹内容是排除部分文件：**

```
tar --exclude scf/service -zcvf scf.tar.gz scf/*
```

**其实最简单的使用 tar 就只要记忆底下的方式即可：**

```
压　缩：tar -jcv -f filename.tar.bz2 要被压缩的文件或目录名称
查　询：tar -jtv -f filename.tar.bz2
解压缩：tar -jxv -f filename.tar.bz2 -C 欲解压缩的目录
```

## 查看进程和端口（ps和ss）

netstat -nap|





## Traceroute（路由追踪）命令

现实世界中的网络是由无数的计算机和路由器组成的一张的大网，应用的数据包在发送到服务器之前都要经过层层的路由转发。而Traceroute是一种常规的网络分析工具，<font style="color: red">用来定位到目标主机之间的所有路由器</font>。

### 原理

在介绍Traceroute的原理之前，需要了解几个技术名词：

- **IP协议**

  IP协议是TCP/IP协议族中最核心的部分，它的作用是在两台主机之间传输数据，所有上层协议的数据（HTTP、TCP、UDP等）都会被封装在一个个的IP数据包中被发送到网络上。

- **ICMP**
  ICMP全称为**互联网控制报文协议**，它常用于传递错误信息，ICMP协议是IP层的一部分，它的报文也是通过IP数据包来传输的。
- **TTL**
  TTL（time-to-live）是IP数据包中的一个字段，它指定了数据包最多能经过几次路由器。从我们源主机发出去的数据包在到达目的主机的路上要经过许多个路由器的转发，在发送数据包的时候源主机会设置一个TTL的值，每经过一个路由器TTL就会被减去一，当TTL为0的时候该数据包会被直接丢弃（不再继续转发），并发送一个超时ICMP报文给源主机。

具体到traceroute的实现细节上，有两种不同的方案：

### 基于UDP实现

在基于UDP的实现中，客户端发送的数据包是通过UDP协议来传输的，使用了一个大于30000的端口号，服务器在收到这个数据包的时候会返回一个**端口不可达**的ICMP错误信息，客户端通过判断收到的错误信息是TTL超时还是端口不可达来判断数据包是否到达目标主机，具体的流程如图：

![img](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20201210154229.png)

基于UDP实现的traceroute

1. 客户端发送一个TTL为1，端口号大于30000的UDP数据包，到达第一站路由器之后TTL被减去1，返回了一个超时的ICMP数据包，客户端得到第一跳路由器的地址。
2. 客户端发送一个TTL为2的数据包，在第二跳的路由器节点处超时，得到第二跳路由器的地址。
3. 客户端发送一个TTL为3的数据包，数据包成功到达目标主机，返回一个**端口不可达**错误，traceroute结束。

## ping命令

Linux ping命令用于检测主机。

执行ping指令会使用ICMP传输协议，发出要求回应的信息，若远端主机的网络功能没有问题，就会回应该信息，因而得知该主机运作正常。

```
ping [-dfnqrRv][-c<完成次数>][-i<间隔秒数>][-I<网络界面>][-l<前置载入>][-p<范本样式>][-s<数据包大小>][-t<存活数值>][主机名称或IP地址]
```

**参数说明**：

- -d 使用Socket的SO_DEBUG功能。
- -c<完成次数> 设置完成要求回应的次数。
- -f 极限检测。
- -i<间隔秒数> 指定收发信息的间隔时间。
- -I<网络界面> 使用指定的网络接口送出数据包。
- -l<前置载入> 设置在送出要求信息之前，先行发出的数据包。
- -n 只输出数值。
- -p<范本样式> 设置填满数据包的范本样式。
- -q 不显示指令执行过程，开头和结尾的相关信息除外。
- -r 忽略普通的Routing Table，直接将数据包送到远端主机上。
- -R 记录路由过程。
- -s<数据包大小> 设置数据包的大小。
- -t<存活数值> 设置存活数值TTL的大小。
- -v 详细显示指令的执行过程。

## awk 命令

AWK 是一种处理文本文件的语言，是一个强大的文本分析工具。

**语法**

```
awk [选项参数] 'script' var=value file(s)
或
awk [选项参数] -f scriptfile var=value file(s)
```

**选项参数说明：**

- -F fs or --field-separator fs

  指定输入文件折分隔符，fs是一个字符串或者是一个正则表达式，如-F:。

- -v var=value or --asign var=value

  赋值一个用户定义变量。

- -f scripfile or --file scriptfile

  从脚本文件中读取awk命令。

- -mf nnn and -mr nnn

  对nnn值设置内在限制，-mf选项限制分配给nnn的最大块数目；-mr选项限制记录的最大数目。这两个功能是Bell实验室版awk的扩展功能，在标准awk中不适用。

- -W compact or --compat, -W traditional or --traditional

  在兼容模式下运行awk。所以gawk的行为和标准的awk完全一样，所有的awk扩展都被忽略。

- -W copyleft or --copyleft, -W copyright or --copyright

  打印简短的版权信息。

- -W help or --help, -W usage or --usage

  打印全部awk选项和每个选项的简短说明。

- -W lint or --lint

  打印不能向传统unix平台移植的结构的警告。

- -W lint-old or --lint-old

  打印关于不能向传统unix平台移植的结构的警告。

- -W posix

  打开兼容模式。但有以下限制，不识别：/x、函数关键字、func、换码序列以及当fs是一个空格时，将新行作为一个域分隔符；操作符**和**=不能代替^和^=；fflush无效。

- -W re-interval or --re-inerval

  允许间隔正则表达式的使用，参考(grep中的Posix字符类)，如括号表达式[[:alpha:]]。

- -W source program-text or --source program-text

  使用program-text作为源代码，可与-f命令混用。

- -W version or --version

  打印bug报告信息的版本。

用法一：

```
awk '{[pattern] action}' {filenames}   # 行匹配语句 awk '' 只能用单引号
实例：打印log.txt中每行的第二个和第五个值
awk '{print $1,$4}' log.txt
```

用法二：

```
awk -F  #-F相当于内置变量FS, 指定分割字符
实例：# 使用","分割
 $  awk -F, '{print $1,$2}'   log.txt
```

用法三：

```
awk -v  # 设置变量
实例：
awk -va=1 '{print $1,$1+a}' log.txt
```

用法四：

```
awk -f {awk脚本} {文件名}
实例：
awk -f cal.awk log.txt
```

详细用法：https://www.runoob.com/linux/linux-comm-awk.html

## echo命令

1. 在窗口输出指定内容
   echo “content” 或 echo 'content’

![image-20210604101653397](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20210604101701.png)



2. 向文件中写入内容
   echo “content” > 文件名
   如：echo “cover” > a.txt
   （文件原先的内容会被覆盖掉）

3. 向文件追加内容
   echo “content” >> 文件名
   如：echo “add add” >> a.txt





















### 