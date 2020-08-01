# <center>Linux命令</center>

## Centos配置网络

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



















### 