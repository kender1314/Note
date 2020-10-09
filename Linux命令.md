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

（7）配置完成之后，还需要在配置端口

![image-20200730160124962](https://cdn.jsdelivr.net/gh/kender1314/NotePicture/20200730160125.png)

### 配置挂起重启后正常访问端口

```
sudo vi /usr/lib/sysctl.d/00-system.conf
```

在最后添加 IPv4转发状态

```
net.ipv4.ip_forward = 1
```

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











### 