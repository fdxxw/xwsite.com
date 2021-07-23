---
title: "Gitlab + Gitlab Ci"
description: 
date: 2021-07-19
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

## 安装gitlab

docker安装

```bash
# 拉取gitlab镜像
docker pull gitlab/gitlab-ce
# 创建数据存放目录
mkdir -p /data/gitlab/etc
mkdir -p /data/gitlab/log
mkdir -p /data/gitlab/data

# 运行
docker run \
--detach \
--publish 443:443 \
--publish 80:80 \
--publish 222:22 \
--name gitlab \
--restart unless-stopped \
-v /data/gitlab/etc:/etc/gitlab \
-v /data/gitlab/log:/var/log/gitlab \
-v /data/gitlab/data:/var/opt/gitlab \
gitlab/gitlab-ce
```

### 自签名证书

脚本

```bash
#!/bin/sh

 # create self-signed server certificate:
 
 read -p "Enter your domain [139.199.125.93]: " DOMAIN
 
 echo "Create server key..."
 
 openssl genrsa -des3 -out $DOMAIN.key 1024
 
 echo "Create server certificate signing request..."
 
 SUBJECT="/C=US/ST=Mars/L=iTranswarp/O=iTranswarp/OU=iTranswarp/CN=$DOMAIN"
 
 openssl req -new -subj $SUBJECT -key $DOMAIN.key -out $DOMAIN.csr
 
 echo "Remove password..."
 
 mv $DOMAIN.key $DOMAIN.origin.key
 openssl rsa -in $DOMAIN.origin.key -out $DOMAIN.key
 
 echo "Sign SSL certificate..."
 
 echo subjectAltName = IP:$DOMAIN >> extfile.cnf
 
 openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt -extfile extfile.cnf
 
 echo "TODO:"
 echo "Copy $DOMAIN.crt to /etc/nginx/ssl/$DOMAIN.crt"
 echo "Copy $DOMAIN.key to /etc/nginx/ssl/$DOMAIN.key"
 echo "Add configuration in nginx:"
 echo "server {"
 echo "    ..."
 echo "    listen 443 ssl;"
 echo "    ssl_certificate     /etc/nginx/ssl/$DOMAIN.crt;"
 echo "    ssl_certificate_key /etc/nginx/ssl/$DOMAIN.key;"
 echo "}"
```

把以上脚本存为self-sign.sh，并添加执行权限

```bash
chmod +x ./self-sign.sh
# 假设gitlab所在服务器地址为192.168.13.40
# 运行脚本
./self-sign.sh
# 复制证书到gitlab配置目录下
mkdir -p /data/gitlab/etc/ssl
cp 192.168.13.40.crt /data/gitlab/etc/ssl/server.crt
cp 192.168.13.40.key /data/gitlab/etc/ssl/server.key
sudo chmod -R 700 /data/gitlab/etc/ssl/ 

# 修改 vim /data/gitlab/etc/gitlab.rb
# 配置http协议所使用的访问地址,不加端口号默认为80/443
external_url 'https://192.168.13.40'
 #修改nginx配置 810行
nginx['redirect_http_to_https'] =true
nginx['ssl_certificate'] = "/etc/gitlab/ssl/192.168.13.40.crt"
nginx['ssl_certificate_key'] = "/etc/gitlab/ssl/192.168.13.40.key"

# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = '192.168.13.40'
gitlab_rails['gitlab_shell_ssh_port'] = 222 # 此端口是run时22端口映射的222端口'

# 重启gitlab

docker restart gitlab
```





## 安装gitlab-runner

使用docker安装

```bash
# 拉取镜像
docker pull gitlab/gitlab-runner
# 创建配置文件存放目录
mkdir -p /data/gitlab-runner/etc
mkdir /data/gitlab-runner/etc/certs
mkdir -p /data/gitlab-runner/run
# 复制gitlab的证书到gitlab-runner配置目录
cp /data/gitlab/etc/ssl/server.crt /data/gitlab-runner/etc/certs/192.168.13.40.crt
# 运行gitlab-runner
docker run -d --name gitlab-runner --restart always \
  -v /data/gitlab-runner/etc:/etc/gitlab-runner \
  -v /var/run/docker.sock:/var/run/docker.sock \
  gitlab/gitlab-runner
```

注册一个runner到gitlab

```bash
# https://docs.gitlab.com/runner/configuration/tls-self-signed.html
docker exec -it gitlab-runner  gitlab-runner register
```

依次填写需要的信息

1. Enter the GitLab instance URL (gitlab的地址)

   https://192.168.13.40

2. Enter the registration token (从gitlab的设置 -> CI/CD -> Runner -> 复制注册令牌)

3. Enter a description for the runner （添加描述）

4. Enter tags for the runner (comma-separated) (添加标签)

5. Enter an executor: custom, ssh, docker+machine, shell, virtualbox, docker-ssh+machine, kubernetes, docker, docker-ssh, parallels (选择执行环境)

   docker

6. Enter the default Docker image (for example, ruby:2.6) (设置默认docker镜像)

## 编写.gitlab-ci.yml

官方文档： https://docs.gitlab.com/ce/ci/yaml/index.html

1. 在项目根路径下创建.gitlab-ci.yml文件

2. 设置需要的变量

   ```yaml
   variables:
     jvm: "openj9"
   ```

3. 设置使用的镜像

   ```yaml
   image: fdxxw/build:1.0
   ```

4. 设置构建步骤stages

   同一个stage的可以并发，不同stage的按照顺序执行

   ```yaml
   stages:
     - dependency
     - build
     - docker
     - deploy
     - cleanup
   ```

5. 设置每个任务执行前的脚本

   ```yaml
   before_script:
     - yarn --version
     - mvn --version
     - docker --version
   ```

   

6. 定义任务

   ```yaml
   # 任务名称
   oms_dependency:
     stage: dependency # 任务属于哪个步骤
     # 对任务结果进行缓存，方便下次重复使用
     cache:
       key: "$CI_COMMIT_REF_SLUG-oms-web"
       paths:
         - web/oms-web/node_modules
     # 任务脚本
     script:
       - cd web/oms-web
       - echo "安装oms-web依赖"
       - yarn install
     rules:
       - changes:
           - web/oms-web/package.json
   oms_build:
     stage: build
     cache:
       key: "$CI_COMMIT_REF_SLUG-oms-web"
       policy: pull  # 拉取缓存
       paths:
         - web/oms-web/node_modules
     script:
       - cd web/oms-web
       - yarn build
       - mkdir -p ../../build/web/oms
       - cp -r dist/* ../../build/web/oms/
     # 归档构建的产物
     artifacts:
       expire_in: 1 week
       paths:
         - build/web/oms
   ```

   

