---
title: "Dockerfile教程"
description: 
date: 2021-07-27T15:34:25+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

## 简介

阅读本文章前需要先了解docker的基本知识。

我们可以通过`docker pull imageName`来拉取特定的镜像，也可以通过Dockerfile来构建自己的镜像。Dockerfile是一个简单的文本文件，包含了一系列的指令来告诉docker如何构建镜像。

## 快速开始

1. 在当前目录创建一个文件，叫做Dockerfile，可以使用vim来编辑

   ```bash
   sudo vim Dockerfile
   ```

   

2. 在Dockerfile中添加一下指令

   ```dockerfile
   #简单的nginx镜像
   FROM ubuntu 
   MAINTAINER ucmxxw@163.com 
   RUN apt-get update 
   RUN apt-get install –y nginx 
   CMD [“echo”,”Image created”] 
   ```

   第一行是注释，Dockerfile的注释以#开头

   第二行 `FROM ubuntu` 告诉docker用ubuntu作为新镜像的基础镜像，也可以使用其他镜像

   第三行 `MAINTAINER ucmxxw@163.com ` 说明是谁维护的这个镜像，一般用邮箱来作为标志

   第四行`RUN apt-get update` 表示在ubuntu基础镜像上运行`apt-get update`命令

   第五行`RUN apt-get install –y nginx `表示安装nginx

   第六行`CMD [“echo”,”Image created”] `是容器运行时执行的命令

3. 构建镜像并运行

   ```bash
   docker build -t fdxxw/test:0.1 .
   ```

   docker build命令构建镜像

   -t 参数指示目标镜像的名称和版本，`imageName:version`

   . 告诉docker构建的上下文是当前目录

## 镜像的结构

Docker镜像结构是分层的，由一系列的层组成（Layer），每一层对应于Dockerfile中的一条指令，指令越多，层越多，Docker构建镜像时按照指令生成每一层，层的大小按照指令开始到结束文件系统新增的大小来计算，最好把无用的文件删除，减少层的体积，构建时有缓存，指令没有变化的会使用上一次该指令的结果，所以不会变化的指令尽量放在开始。

## 指令

Dockerfile是由一行行的指令命令组成的，下面是一些常用的指令

### FROM

`FROM`指令告诉docker使用哪个镜像作为基础镜像

```dockerfile
FROM node:14.17.1-buster
```

### RUN

`RUN`指令表示在基础镜像中运行命令，类似于在对应的系统中运行命令

```dockerfile
# 修改时区和设置yarn的taobao镜像
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime; \
    echo "Asia/Shanghai" > /etc/timezone; \
    yarn config set registry https://registry.npm.taobao.org;
```

### ENV

`ENV`表示设置环境变量，可以在后续指令以及容器中使用

```dockerfile
# 设置代理
ENV HTTPS_PROXY=http://192.168.13.37:7890
```

### CMD

`CMD`指令表示当容器运行时执行的命令

语法`CMD command param1 `

```dockerfile
# 容器运行时输出hello world
CMD ["echo", "hello world"]
```

### ENTRYPOINT

语法`ENTRYPOINT command param1 `

`ENTRYPOINT`指令的作用同`CMD`，也表示容器运行时执行的命令，不过它是可扩展的，容器运行时可以紧跟命令参数

```dockerfile
FROM ubuntu
MAINTAINER ucmxxw@163.com
ENTRYPOINT ["echo"]
```

构建镜像

```bash
docker build -t fdxxw/test:0.1 .
```

运行容器

```bash
docker run --rm fdxxw/test:0.1 hello world
```

结果输出hello world

### WORKDIR

`WORKDIR`指令用来设置容器的当前工作目录

```dockerfile
FROM ubuntu
MAINTAINER ucmxxw@163.com
WORKDIR /app
CMD pwd
```

输出 /app

### COPY

`COPY`指令用来把文件/目录添加的镜像中

```dockerfile
FROM nginx:1.19.6
MAINTAINER ucmxxw@163.com
# 把当前目录的nginx.conf配置文件复制到镜像的/etc/nginx下
COPY nginx.conf /etc/nginx/nginx.conf
```

额外的参数

```dockerfile
# 复制的文件添加所有者权限
COPY --chown=emqx:emqx  source target
```

### ADD

`ADD`指令的作用同`COPY`指令都是用来添加文件到镜像中，区别是`ADD`支持添加压缩文件（tar,zip）、网络文件

添加压缩文件时会自动解压到镜像中，网络文件会自动下载并添加到镜像中

```dockerfile
FROM ubuntu
# 添加jre并解压到镜像的/opt/soft目录下，/opt/soft/jre1.8.0_271
ADD jre-8u271-linux-x64.tar.gz /opt/soft/
```



### ARG

`ARG`指令用来定义构建镜像时可以传递的参数，这样可以更方便的来定制所需的镜像

语法 `ARG 参数名=默认值`

```dockerfile
# 定义了一个名为jvm的构建参数，默认值为openj9
ARG jvm=openj9
# 根据jvm参数的值不同来拉取不同版本的jre镜像
FROM adoptopenjdk:11-jre-${jvm}
```

构建时传参，通过`--build-arg 参数名=参数值`来传参

```bash
# 使用hotspot jvm
docker -t fdxxw/app:0.1 -f Dockerfile --build-arg jvm=hotspot .
# 使用openj9 jvm
docker -t fdxxw/app:0.1 -f Dockerfile --build-arg jvm=openj9 .
```

## 分阶段构建

我们在构建镜像的过程中，可能需要依赖很多工具来构建我们的应用，这些工具在不同的镜像中，为了方便的构建镜像，Dockerfile允许我们分阶段来构建镜像，后面的阶段可以获取前面阶段的产物，最后一个阶段作为最终的镜像。

```dockerfile
# 第一阶段，使用node镜像，设置别名为builder
FROM node:14.17.1-buster AS builder
# 复制项目到镜像
COPY project /tmp/project
# 构建web项目，产物在/tmp/project/dist目录下
RUN cd /tmp/project; \
	yarn config set registry https://registry.npm.taobao.org; \
	yarn install; \
	yarn build;

# 第二阶段，构建最终使用的镜像
FROM nginx:1.19.6
# 从第一阶段 builder复制/tmp/project/dist产物到本阶段/var/www
COPY --from=builder /tmp/project/dist /var/www
```

