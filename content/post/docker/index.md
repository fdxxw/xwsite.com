---
title: "Docker使用教程"
description: 
date: 2021-07-23T14:05:04+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
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

2. 

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

   

6. docker push image

   推送镜像到仓库

   ```bash
   # 推送镜像到官方 docker hub
   docker push $dockerid/$repo:$tag
   
   # 推送镜像到私人仓库, $registry是私人仓库的地址 eq. 192.168.13.91:5000
   docker tag $repo:$tag $registry/$repo:$tag
   docker push $registry/$repo:$tag
   docker rmi $registry/$repo:$tag
   ```

   

7. 