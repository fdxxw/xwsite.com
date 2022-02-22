---
title: "Docker Swarm"
description: 
date: 2021-07-28T16:38:49+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

## overlay 网络

`overlay`网络驱动在多个docker宿主机之间创建了一个分布式网络，位于特定主机网络之上，允许容器连接到这个网络上进行安全的通信。

## 前置条件

要使用overlay网络需要防护墙开放以下端口

+ 2377/tcp  (集群管理通信)
+ 7946/tcp和7946/udp   (节点间通信)
+ 4789/udp   (overlay网络传输)

## 创建overlay网络

```bash
docker network create \
--driver overlay \
--subnet=10.11.0.0/16 \
--gateway=10.11.0.2 \
--attachable \
--opt encrypted \
my-overlay
```

+ --driver 使用overlay网络驱动
+ --subnet 设置子网
+ --gateway 设置网关
+ --attachable 可以让孤立容器加入该网络
+ --opt encrypted 加密数据

查看网络详细信息

```bash
docker network inspect my-overlay
```

## 创建容器并加入overlay网络

```bash
# 创建app容器
docker run -it --rm --name app --network my-overlay alpine
# 创建db容器
docker run -it --rm --name db --network my-overlay alpine
```

连通性测试

```bash
# 1. 在db容器执行
ip addr
ping -c 2 app
nc -l -p 5432
# 2. 在app容器执行
ip addr
ping -c 2 db
nc db 5432
```

## swarm集群

一个swarm集群由多个docker主机组成，有一个管理节点和多个工作节点（管理节点也是工作节点），工作节点运行swarm服务，创建一个服务的时候可以定义它的复用个数、网络、存储资源、端口映射等。如果一个工作节点不可用了，docker会将任务调度给其它节点运行。

## 节点

节点是docker engine的实例，可以认为就是一个docker节点。管理节点把任务分配给工作节点，还执行编排任务和必要的集群管理功能来维持集群的理想状态。工作节点执行从管理节点分配下来的任务，工作节点会将其分配的任务的当前状态通知管理节点，以便管理节点进行管理。

## 服务和任务

服务是任务的定义，是swarm集群的中心结构，服务的配置与运行一个容器基本上是一致的，只是多了一些额外的配置内容，服务通常使用docker-compose来编排。



## 创建swarm

在管理节点上运行

```bash
docker swarm init 
```

成功后会提示下面的信息

```bash
Swarm initialized: current node (1jhsmn285m1xp4st6xnmechpt) is now a manager.
To add a worker to this swarm, run the following command:
docker swarm join --token SWMTKN-1-1bpaupkyapd5ialcwo60rfcp9i2x04rxguk3a0z6oil7b4fj5g-3qt48itzm539io1hye4mqvhv2 192.168.13.37:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

按照提示在工作节点上运行对应的加入集群命令即可

例如：

```bash
docker swarm join --token SWMTKN-1-1bpaupkyapd5ialcwo60rfcp9i2x04rxguk3a0z6oil7b4fj5g-3qt48itzm539io1hye4mqvhv2 192.168.13.37:2377
```



## swarm管理

1. 查看节点列表

   docker node ls

2. 查看节点详情

   docker inspect 节点id

3. 离开集群

   docker swarm leave

4. 给节点添加标签

   sudo docker node update --label-add `app_web=true` 节点id

5. 删除节点标签

   sudo docker node update --label-rm `app_web` 节点id

6. 

## 服务编排

docker-compose文件

```yaml
version: '3'
services:
  web:
    image: accu/web:{{.Env.ecsVer}}
    container_name: web
    environment:
      TZ: "Asia/Shanghai"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - "/etc/localtime:/etc/localtime"
    deploy:
      placement:
        constraints:
          - "node.labels.app_web==true"
  postgres:
    image: accu/postgres:{{.Env.postgresVer}}
    container_name: postgres
    command: postgres -c 'max_connections=1000'
    volumes:
      - "/etc/localtime:/etc/localtime"
      - "/data/pgdata:/var/lib/postgresql/data"
    environment:
      - "TZ=Asia/Shanghai"
      - "POSTGRES_PASSWORD=123456"
      - "POSTGRES_USER=postgres"
      - "POSTGRES_DB=ecs"
    ports:
      - "5432:5432"
    deploy:
      placement:
        constraints:
          - "node.labels.db_postgres==true"
  redis:
    image: accu/redis:{{.Env.redisVer}}
    container_name: redis
    environment:
      TZ: "Asia/Shanghai"
    volumes:
      - "/etc/localtime:/etc/localtime"
      - "/data/redis:/data"
    ports:
      - "6379:6379"
    sysctls:
      - net.core.somaxconn=1024
    deploy:
      placement:
        constraints:
          - "node.labels.db_redis==true"
  agent:
    image: portainer/agent
    environment:
      # REQUIRED: Should be equal to the service name prefixed by "tasks." when
      # deployed inside an overlay network
      AGENT_CLUSTER_ADDR: tasks.agent
      # AGENT_PORT: 9001
      # LOG_LEVEL: debug
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    ports:
      - "9002:9000"
      - "8000:8000"
    volumes:
      - /data/portainer_data:/data
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
```

与compose编排容器的区别在于多了`deploy`配置，下面是`deploy`配置的详细说明

1. mode  （模式）

   - global   (每个节点有一个容器)
   - replicated  (指定容器的数量)

   ```yml
   deploy:
     mode: replicated
   ```

2. placement-constraints （可运行容器的约束）

   ```yml
   deploy:
     placement:
       constraints:
         - "node.role==manager"
         - "node.platform.os==linux"
         - "app_web==true"
   ```

3. replicas (可运行容器的数量, 当模式为replicated)

   ```yml
   deploy:
     mode: replicated
     replicas: 6
   ```

   

4. resources (资源限制)

   ```yml
   deploy:
     resources:
       limits:  # 资源限制
         cpus: '0.50'   # 单个core的50%
         memory: 50M    # 不能超过50M内存
       reservations:  # 预留资源（总是可用的）
         cpus: '0.25'
         memory: 20M
   ```

   

5. restart_policy (重启策略)

   ```yml
   deploy:
     restart_policy:
       condition: on-failure  # 条件, {none, on-failure, any}
       delay: 5s  # 重启间隔
       max_attempts: 3 # 重启重试次数
   ```

   

6. 

## 服务管理

通过compose文件可以快速部署服务

```bash
# 部署服务，服务栈叫ecs
docker stack deploy -c docker-compose.yml ecs
```

删除服务栈

```bash
# 删除ecs
docker stack rm ecs
```

查看服务栈任务

```bash
# 查看ecs的所有任务
docker stack ps ecs
```

查看服务栈的服务列表

```bash
docker stack services ecs
```

查看服务日志

```bash
# docker service logs 服务栈_服务名
docker service logs ecs_app
```

