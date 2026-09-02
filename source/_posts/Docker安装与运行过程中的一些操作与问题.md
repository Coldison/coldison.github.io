---
title: Docker build中的常见错误及常用命令(未完)
date: 2022-06-24 14:49:59
tags:
- Linux
- Docker
categories:
- 教程
- Docker使用
---

Docker安装和从docker file build网上应该有教程，这里记录一下我自己在安装pymarl所遇到的一些问题及一些常用命令。目前只写了一点点，后续会继续在这篇博文之中添加。

### 一. 常用命令

   docker删除镜像：

  1. 删除容器

   ```bash
   docker ps #查看正在运行的容器

   docker ps -a #查看所有容器

   docker rm container_id #删除容器
   ```
  2. 删除镜像

   ```bash
   docker images //查看镜像

   docker rmi image_id
   # 删除 null image

   sudo docker rmi $(docker images -f "dangling=true" -q) #删除所有镜像

   # 删掉容器
   docker stop $(docker ps -qa)
   docker rm $(docker ps -qa)

   # 删除镜像
   docker rmi --force $(docker images -q)

   # 删除名称中包含某个字符串的镜像
   # 例如删除包含“some”的镜像
   docker rmi --force $(docker images | grep some | awk '{print $3}')
   ```

### 二. 常见错误

  1. debconf: delaying package configuration, since apt-utils is not installed是说```apt-utils``` 没有安装，对结果并没有什么危害，只是影响交互式安装。这个```apt-utils``` 可以实现在安装过程中交互式配置文件，可以通过：

   ```bash
   RUN apt-get install --assume-yes apt-utils
   ```

   忽略掉这个警告信息。

  2. InRelease:InRelease: The following signatures couldn't be verified because the public key is not available

   ```bash
   sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys　425956BB3E31DF51
   sudo apt update
   ```

   如果不行的话可以尝试在docker file中添加，```RUN rm/etc/apt/sources.list.d/cuda.list```
