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





 







###