# Docker-compose笔记

## 概述

Docker Compose 是 Docker 官方编排（Orchestration）项目之一，负责快速的部署分布式应用，它是由 `python` 编写。

`Compose` 定位是定义和运行多个 Docker 容器的应用。

`Compose` 有两个重点

- `docker-compose.yml` `compose` 配置文件
- `docker-compose` 命令行工具

## 安装Docker-compose

```
yum install docker-compose
```



## Docker-compose举例

### 步骤1：设定

```
$ mkdir composetest
$ cd composetest
```

在项目目录中创建一个名为`app.py`的文件，这个文件主要是运行的代码

```
import time

import redis
from flask import Flask

app = Flask(__name__)
cache = redis.Redis(host='redis', port=6379)

def get_hit_count():
    retries = 5
    while True:
        try:
            return cache.incr('hits')
        except redis.exceptions.ConnectionError as exc:
            if retries == 0:
                raise exc
            retries -= 1
            time.sleep(0.5)

@app.route('/')
def hello():
    count = get_hit_count()
    return 'Hello World! I have been seen {} times.\n'.format(count)
```

在项目目录中创建另一个名为`requirements.txt`的文件

```
flask
redis
```

### 步骤2：建立

创建一个名为`Dockerfile`的文件

```
FROM python:3.7-alpine
WORKDIR /code
ENV FLASK_APP=app.py
ENV FLASK_RUN_HOST=0.0.0.0
RUN apk add --no-cache gcc musl-dev linux-headers
COPY requirements.txt requirements.txt
RUN pip install -r requirements.txt
EXPOSE 5000
COPY . .
CMD ["flask", "run"]
```

这告诉Docker：

- 从Python 3.7映像开始构建映像。
- 将工作目录设置为`/code`。
- 设置`flask`命令使用的环境变量。
- 安装gcc和其他依赖项
- 复制`requirements.txt`并安装Python依赖项。
- 向图像添加元数据以描述容器正在侦听端口5000
- 将`.`项目中的当前目录复制到`.`映像中的工作目录。
- 将容器的默认命令设置为`flask run`。

### 步骤3：在撰写文件中定义服务

在项目目录中创建一个名为`docker-compose.yml`的文件

```
version: "3.8"
services:
  web:
    build: .
    ports:
      - "5000:5000"
    volumes:
      - .:/code
    environment:
      FLASK_ENV: development
  redis:
    image: "redis:alpine"
```

该Compose文件定义了两个服务：`web`和`redis`。

### 第4步：生成和运行与撰写你的应用程序

在项目目录中，运行`docker-compose up`来启动应用程序。

```
docker-compose up
```



## 操作命令

### 后台运行

```
docker-compose up -d
```

### 停止所有服务

```
docker-compose stop
```

### 运行指定compose文件

```
docker-compose -f docker-compose.yml -f production.yml up -d
```

### 重建镜像并重建容器

```
docker-compose build web
docker-compose up --no-deps -d web
```

这首先重建图像`web`，然后停止，销毁，并重新创建 `web`服务。--no-deps标志阻止Compose重新创建任何web依赖的服务。











## 多个docker-compose文件

在本节中，有两个常见的用例，用于多个Compose文件：针对不同的环境更改Compose应用程序，以及针对Compose应用程序运行管理任务。

定义服务规范配置的基本文件**docker-compose.yml**

```
web:
  image: example/my_web_app:latest
  depends_on:
    - db
    - cache

db:
  image: postgres:latest

cache:
  image: redis:latest
```

**示例一**

开发配置将一些端口暴露给主机，将我们的代码作为卷安装，并构建Web映像。

**docker-compose.override.yml**

```
web:
  build: .
  volumes:
    - '.:/code'
  ports:
    - 8883:80
  environment:
    DEBUG: 'true'

db:
  command: '-d'
  ports:
    - 5432:5432

cache:
  ports:
    - 6379:6379
```

运行`docker-compose up`它将自动读取docker-compose.override.yml。

**示例二**

现在另外建一个**docker-compose.prod.yml**，用于别的用途

```
web:
  ports:
    - 80:80
  environment:
    PRODUCTION: 'true'

cache:
  environment:
    TTL: '500'
```

运行则执行

```
docker-compose -f docker-compose.yml -f docker-compose.prod.yml up -d
```

## **配置选项**

### **bulid**

build可以指定 Dockerfile 所在文件夹的路径，可以是相对路径，也可以是绝对路径。

```
build: /path/to/build/dir
```

### **context**

context 选项可以是 Dockerfile 的文件路径，也可以是到链接到 git 仓库的url，当提供的值是相对路径时，它被解析为相对于compose文件的路径，此目录也是发送到 Docker 守护进程的 context

```
build:
  context: ./dir
```

### **dockerfile**

使用此 dockerfile 文件来构建，必须指定构建路径

```
build:
  context: .
  dockerfile: Dockerfile-alternate
```

### **image**

指定镜像

```
services:
  web:
    image: nginx
```

###  **args**

添加构建参数，这些参数是仅在构建过程中可访问的环境变量
首先， 在Dockerfile中指定参数：

```
ARG fendo
ARG password
 
RUN echo "Build number: $fendo"
RUN script-requiring-password.sh "$password"
```

然后指定 build 下的参数,可以传递映射或列表

```
build:
  context: .
  args:
    fendo: 1
    password: fendo
```

或

```
build:
  context: .
  args:
    - fendo=1
    - password=fendo
```

指定构建参数时可以省略该值，在这种情况下，构建时的值默认构成运行环境中的值

```
args:
  - fendo
  - password
```

### **command**

使用 command 可以覆盖容器启动后默认执行的命令。

```html
command: bundle exec thin -p 3000
```

该命令也可以是一个列表，方法类似于 dockerfile:

```html
command: ["bundle", "exec", "thin", "-p", "3000"]
```

### **container_name**

Compose 的容器名称格式是：<项目名称><服务名称><序号>

虽然可以自定义项目名称、服务名称，但是如果你想完全控制容器的命名，可以使用这个标签指定：

```html
container_name: app
```

这样容器的名字就指定为 app 了。

### **depends_on**

在使用 Compose 时，最大的好处就是少打启动命令，但是一般项目容器启动的顺序是有要求的，如果直接从上到下启动容器，必然会因为容器依赖问题而启动失败。

例如在没启动数据库容器的时候启动了应用容器，这时候应用容器会因为找不到数据库而退出，为了避免这种情况我们需要加入一个标签，就是 depends_on，这个标签解决了容器的依赖、启动先后的问题。

例如下面容器会先启动 redis 和 db 两个服务，最后才启动 web 服务：

```html
version: '3'
services:
  web:
    build: .
    depends_on:
      - db
      - redis
  redis:
    image: redis
  db:
    image: postgres
```

注意的是，默认情况下使用 docker-compose up web 这样的方式启动 web 服务时，也会启动 redis 和 db 两个服务，因为在配置文件中定义了依赖关系。

### **pid**

```html
pid: "host"
```

将PID模式设置为主机PID模式，跟主机系统共享进程命名空间。容器使用这个标签将能够访问和操纵其他容器和宿主机的名称空间。

### **ports**

映射端口的标签。

使用HOST:CONTAINER格式或者只是指定容器的端口，宿主机会随机映射端口。

```html
ports:
 - "3000"
 - "8000:8000"
 - "49100:22"

 - "127.0.0.1:8001:8001"
```

注意：当使用HOST:CONTAINER格式来映射端口时，如果你使用的容器端口小于60你可能会得到错误得结果，因为YAML将会解析xx:yy这种数字格式为60进制。所以建议采用字符串格式。

### **extra_hosts**

添加主机名的标签，就是往/etc/hosts文件中添加一些记录，与Docker client的--add-host类似：

```html
extra_hosts:
 - "somehost:162.242.195.82"
 - "otherhost:50.31.209.229"
```

启动之后查看容器内部hosts：

```html
162.242.195.82  somehost
50.31.209.229   otherhost
```

### **volumes**

挂载一个目录或者一个已存在的数据卷容器，可以直接使用 [HOST:CONTAINER] 这样的格式，或者使用 [HOST:CONTAINER:ro] 这样的格式，后者对于容器来说，数据卷是只读的，这样可以有效保护宿主机的文件系统。
Compose的数据卷指定路径可以是相对路径，使用 . 或者 .. 来指定相对目录。

数据卷的格式可以是下面多种形式：

```html
volumes:
  // 只是指定一个路径，Docker 会自动在创建一个数据卷（这个路径是容器内部的）。
  - /var/lib/mysql
  // 使用绝对路径挂载数据卷
  - /opt/data:/var/lib/mysql
  // 以 Compose 配置文件为中心的相对路径作为数据卷挂载到容器。
  - ./cache:/tmp/cache
  // 使用用户的相对路径（~/ 表示的目录是 /home/<用户目录>/ 或者 /root/）。
  - ~/configs:/etc/configs/:ro
  // 已经存在的命名的数据卷。
  - datavolume:/var/lib/mysql
```

如果你不使用宿主机的路径，你可以指定一个volume_driver。

```html
volume_driver: mydriver
```

## Compose和Docker兼容性矩阵

| **撰写档案格式** | **Docker Engine版本** |
| :--------------- | :-------------------- |
| 3.8              | 19.03.0+              |
| 3.7              | 18.06.0+              |
| 3.6              | 18.02.0+              |
| 3.5              | 17.12.0+              |
| 3.4              | 17.09.0+              |
| 3.3              | 17.06.0+              |
| 3.2              | 17.04.0+              |
| 3.1              | 1.13.1+               |
| 3.0              | 1.13.0+               |
| 2.4              | 17.12.0+              |
| 2.3              | 17.06.0+              |
| 2.2              | 1.13.0+               |
| 2.1              | 1.12.0+               |
| 2.0              | 1.10.0+               |
| 1.0              | 1.9.1。+              |

















###