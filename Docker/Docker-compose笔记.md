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

后台运行

```
docker-compose up -d
```

停止所有服务

```
docker-compose stop
```



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











 







###