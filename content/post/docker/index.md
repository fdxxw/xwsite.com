---
title: "Docker使用教程"
description: 
date: 2021-07-21
image: 
math: 
license: 
---

## 安装

快速安装

```sh
curl -sSL https://get.daocloud.io/docker | sh
```

离线安装

从 https://mirrors.aliyun.com/docker-ce/linux/网站下载对应的包进行安装，下面的例子是在ubuntu 16.04下安装的过程

```bash
# 下载containerd.io
wget -O containerd.io.deb https://mirrors.aliyun.com/docker-ce/linux/ubuntu/dists/xenial/pool/stable/amd64/containerd.io_1.4.6-1_amd64.deb
# 安装containerd.io
sudo dpkg -i containerd.io.deb

# 下载 docker-ce-cli
wget -O docker-ce-cli.deb https://mirrors.aliyun.com/docker-ce/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce-cli_20.10.7~3-0~ubuntu-xenial_amd64.deb

# 安装docker-ce-cli
sudo dpkg -i docker-ce-cli.deb

# 下载docker-ce
wget -O docker-ce.deb https://mirrors.aliyun.com/docker-ce/linux/ubuntu/dists/xenial/pool/stable/amd64/docker-ce_20.10.7~3-0~ubuntu-xenial_amd64.deb

# 安装docker-ce
sudo dpkg -i docker-ce.deb

# 下载docker-compose https://github.com/docker/compose/releases
wget https://github.com/docker/compose/releases/download/1.29.2/docker-compose-Linux-x86_64
# 安装docker-compose
mv docker-compose-Linux-x86_64 /usr/local/bin/docker-compose
# 添加可执行权限
chmod +x /usr/local/bin/docker-compose

# 查看docker 版本
docker version
# 查看docker 服务详细信息
docker info
# 查看docker-compose版本
docker-compose version
# 示例
docker run hello-world 
```

## 配置

1. 镜像加速

   使用网易163镜像源，编辑 /etc/docker/daemon.json

   ```json
   {
       "registry-mirrors": "https//hub-mirror.c.163.com"
   }
   ```

   重启docker `systemctl restart docker`

2. 修改默认存储目录

   docker 默认的存储目录是`/var/lib/docker`

   ```bash
   # 停止docker
   systemctl stop docker
   # 复制数据到新目录/data
   cp -r /var/lib/docker/ /data
   ```

   编辑/etc/docker/daemon.json，使用`data-root`来配置

   ```bash
   {
       "registry-mirrors": "https//hub-mirror.c.163.com",
       "data-root": "/data/docker"
   }
   ```

   修改完后启动docker `systemctl start docker`

3. 

## Docker Hub

官方地址：https://hub.docker.com/

可以进行注册、浏览下载其它社区提供的镜像

登录到docker hub

```bash
# 输入docker id和密码登录
docker login
```

## 镜像

在docker中，一切都是基于镜像，镜像是文件系统和参数的集合

示例

```bash
docker run hello-world
```

docker的run命令表示创建了一个`hello-world`镜像的实例（叫做容器），并运行该容器

镜像的常用操作命令

1. docker images

   列出系统中的所有镜像，镜像有下列属性

   + TAG (镜像的逻辑标签)
   + Image ID (镜像的唯一id)
   + Created (镜像的创建时间)
   + Size (镜像的大小)

2. docker images -q

   只显示镜像的id

3. docker pull image

   从docker hub下载镜像到本地

   ```bash
   docker pull hello-world
   ```

4. docker tag image newImage

   给镜像添加一个新的标签

   ```bash
   docker tag hello-world:latest 192.168.13.91:5000/hello-world:latest
   ```

   

5. docker rmi ImageID

   删除本地镜像

   ```bash
   sudo docker rmi `sudo docker images hello-world:latest -q`
   ```

6. docker info image

   查看镜像的详细信息

   ```bash
   docker inspect ubuntu
   ```

   

7. docker push image

   推送镜像到仓库

   ```bash
   # 推送镜像到官方 docker hub
   docker push $dockerid/$repo:$tag
   
   # 推送镜像到私人仓库, $registry是私人仓库的地址 eq. 192.168.13.91:5000
   docker tag $repo:$tag $registry/$repo:$tag
   docker push $registry/$repo:$tag
   docker rmi $registry/$repo:$tag
   ```

   

8. 



## 容器

容器是镜像的实例，可以通过docker的run命令来运行

容器的常用操作命令

1. docker run -it  ubuntu /bin/bash

   以交互式模式（-it）运行ubuntu容器的bash命令

   按下`Ctrl + q `将返回到系统的命令行，容器在后台运行

   加上`--rm`表示运行完就自动删除容器`docker run -it --rm ubuntu /bin/bash`

   加上`--name`指定容器的名称 `docker run -it --name foo ubunt /bin/bash`

   使用`-d`而不是`-it`表示后台运行容器

2. docker ps

   列出所有运行中的容器，容器信息说明：

   + CONTAINER ID (容器的ID)
   + IMAGE (容器所使用的镜像)
   + COMMAND (容器运行的命令)
   + CREATED (容器的创建时间)
   + STATUS (容器的当前状态)
   + PORTS (容器映射的端口)
   + NAMES (容器的名称)

   加上`-a`参数可以列出所有的容器，包括未运行的 `docker ps -a`

3. docker stop，start，restart  容器名称/id

   停止，启动，重启容器

   ```bash
   docker stop foo
   docker start foo
   docker restart foo
   ```

4. docker rm 容器名称/id

   删除容器，加上`-f`强制删除

   ```bash
   docker stop foo
   docker rm foo
   docker rm -f foo
   ```

5. docker inspect 容器名称/id

   查看容器的详细信息 `docker inspect foo`

6. docker logs 容器名称/id

   查看容器的日志

   ```bash
   docker logs foo
   ```

   

7. docker exec -it 容器名称/id /bin/bash

   执行运行中容器的某个命令，`docker exec -it foo /bin/bash`进入foo容器的命令行

8. 

## 存储

通过`docker info`命令可以看到当前使用的存储驱动

```yaml
Server:
 ...
 Storage Driver: overlay2
  Backing Filesystem: extfs
  Supports d_type: true
  Native Overlay Diff: true
```

### 数据卷

数据卷的作用

+ 在容器被创建之后初始化
+ 可以在多个容器之间共享和使用
+ 对数据卷的任何改变都是直接的
+ 容器被删除后仍然存在

通过docker run 的`-v`参数来映射数据卷

```bash
# 把本地/tmp/vol目录映射到容器的/vol目录
# 需要注意的是docker必须要有目录的权限
docker run --rm -it -v /tmp/vol:/vol / ubuntu /bin/bash
```

数据卷的常用操作命令：

通过docker volume --help来查看卷的子命令

1. 创建卷

   ```bash
   # 创建一个卷，叫做maven
   docker volumn create maven
   # 容器使用
   docker run -v maven:/maven
   ```

   卷创建完成后存放在docker存储目录下的volumes目录下，默认是`/var/lib/docker/volumes`

2. docker volume ls

   列出所有的卷

3. docker volume rm maven

   删除卷

4. docker volume inspect maven

   查看卷的详细信息

5. 

## 网络

网络被用来容器与容器，容器与docker host之间的通信，通过`ifconfig`命令可以看到有一个`docker0`的网络适配器，这个适配器桥接了docker host和linux host

网络的相关操作命令：

1. docker network ls

   列出所有的网络，信息说明：

   + NETWORK ID (网络id)
   + NAME (网络名称)
   + DRIVER (驱动)
   + SCOPE (作用域，一般都是local)

2. docker network inspect host

   查看某个网络的详细信息

3. docker network create --driver drivername name

   创建指定驱动的网络

   drivername: 驱动的名称

   name: 网络的名称

   ```bash
   # 创建new_nw网络
   docker network create --driver bridge new_nw
   # 创建容器的时候使用new_nw
   docker run --rm -it -network=new_nw ubunt /bin/bash
   ```

   

4. docker network rm name

   删除一个网络

   ```bash
   docker network rm new_nw
   ```



### 端口映射

创建容器的时候指定`-p`选项来映射容器和主机之间的端口

```bash
# 把容器的8080端口映射到主机的80端口，默认是tcp
docker run -p 80:8080
# 映射udp协议
docker run -p 3478:3478/udp
# 映射端口范围
docker run -p 49160-49200:49160-49200/udp
```

