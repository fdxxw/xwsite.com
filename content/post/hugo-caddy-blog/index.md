---
title: "如何使用hugo+caddy2搭建个人blog"
description: 
date: 2021-07-12T18:38:14+08:00
image: 
math: 
license: 
hidden: false
comments: true
draft: false
---

# 前言

记录一下个人博客的搭建过程和踩的一些坑

# 安装hugo 

1. 可执行程序安装

   从 https://github.com/gohugoio/hugo/releases 下载对应系统的压缩包，解压后放到系统的path环境变量对应的目录

   linux下安装脚本：

   ```bash
   cd /tmp
   export hugo_version=0.85.0
   wget -O hugo.tar.gz  https://github.com/gohugoio/hugo/releases/download/v0.85.0/hugo_extended_${hugo_version}_Linux-64bit.tar.gz
   tar -xvf hugo.tar.gz
   mv hugo /usr/local/bin/hugo
   # 添加执行权限
   chmod +x /usr/local/bin/hugo
   # 查看版本
   hugo version
   ```

   

2. docker安装

   ```bash
   docker pull klakegg/hugo:0.84.4-ext
   # 查看hugo版本
   docker run --rm -it klakegg/hugo:0.84.4-ext version
   ```

# 创建site

```
$ hugo new site $站点名称

# 目录结构说明
├── archetypes
│   └── default.md
├── config.toml        # 配置文件
├── content            # 文章目录
├── data
├── layouts
├── static
└── themes             # 主题目录

```

# 使用stack主题

