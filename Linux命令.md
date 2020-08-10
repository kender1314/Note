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

### 显示当前磁盘挂载（包含剩余空间）

```
df -h
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



## linux权限

```
1.更改目录所有者命令:
chown -R root /data/gitlab
2.更改目录权限命令:
chmod -R 755 /data/gitlab
chmod -R 1000 /data/gitlab
3、查看文件夹的权限
ls -la /data/gitlab
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
输出错误的结局方法
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























### 