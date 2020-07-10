# <center>Linux命令</center>

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