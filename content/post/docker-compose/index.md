---
title: "Docker Compose"
description: 
date: 2021-07-28
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

## 简介

在实际的应用中，单个容器完整的提供服务功能的情况是非常少的，大部分都需要运行多个容器来完成服务提供。Docker Compose就是用来同时管理多个容器，这些容器作为一个服务来管理，而不需要一个一个的管理单个容器。

## 安装

Docker Compose可以直接从github的官方仓库下载 https://github.com/docker/compose/releases/

可以直接通过下面的脚本直接下载安装

```bash
# 下载
curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose
   -$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
# 添加执行权限
chmod +x /usr/local/bin/docker-compose
# 查看版本
docker-compose version
```

## Compose 文件

Docker Compose通过compose文件来对容器进行描述和管理，compose文件的格式为`YAML`，缺省名称为docker-compose.yml，缺省名称在运行docker-compose命令时可以不用指定文件

### 示例

下面是一个数据库和web组成服务的例子

```yaml
# compose文件格式版本
version: 3
# 定义服务
services:
  # 数据库服务
  postgres:
    image: postgres:10.6
    container_name: postgres
    command: postgres -c 'max_connections=1000'
    volumes:
      - "/etc/localtime:/etc/localtime"
      - "/data/pgdata:/var/lib/postgresql/data"
    environment:
      - "TZ=Asia/Shanghai"
      - "POSTGRES_PASSWORD=123456"
      - "POSTGRES_USER=postgres"
      - "POSTGRES_DB=demo"
    ports:
      - "5432:5432"
  # web服务
  web:
  	image: nginx
  	container_name: nginx
  	ports:
  	  - "80:80"
  	  - "443:443"
```

+ `postgres`和`web`这两个关键字定义了两个服务，一个用来运行postgres数据库，一个用来运行nginx web服务
+ `image`关键字用来指定使用哪个镜像来运行容器
+ `container_name`用来指定容器的名称
+ `command`用来覆盖镜像本身的命令，运行自己的命令
+ `volumes`用来映射挂载卷
+ `environment`用来设置环境变量
+ `ports`用来映射端口

## 管理

通过docker-compose可以很方便的管理compose文件中定义的容器，下面是一些常用的管理命令

`-f`选项可以指定compose文件

### 启动

创建并启动所有的容器

```bash
# 前台启动所有的服务
docker-compose up
# 后台启动
docker-compose up -d
```

启动单个服务

```bash
# 只启动web服务
docker-compose start web
```

### 停止

```bash
# 停止并删除所有的容器
docker-compose down
# 停止所有容器
docker-compose stop
```

停止单个容器

```bash
docker-compose stop web
```

### 重启

```bash
# 重启所有容器
docker-compose restart
# 重启单个容器
docker-compose restart web
```

### 查看容器列表

```bash
# 查看运行中的容器
docker-compose ps
# 查看所有容器
docker-compose ps -a
```

### 查看日志

```bash
# 查看web服务容器的日志
docker-compose logs web
```

### 运行单个服务

```bash
# 单独运行一次web服务
docker-compose run --rm web
docker-compose run --rm web bash
```

### 执行其他命令

```bash
# 登录web服务的容器
docker-compose exec web bash
```

