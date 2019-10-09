---
title: docker 的安装和常用命令
date: 2019-08-25 14:12:10
categories: 技术分享
tags: docker
---

### 安装docker
系统环境: ubuntu 18.04
要确保能够访问docker 仓库地址  :https://download.docker.com/linux/ubuntu
```
sudo apt-get update
 sudo apt install apt-transport-https ca-certificates curl software-properties-common
```
在/etc/apt/sources.list.d/docker.list文件中添加下面内容
`deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable`
<!--more-->
添加密匙
`curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -`
安装 docker-ce
`sudo apt install docker-ce`
查看 docker 是否安装成功
`docker--version`

### 一些docker 的常用命令
下载镜像
`docker pull centos`
查看本地镜像
`docker images`
运行一个镜像病生成容器
`docker run centos /bin/bash`
查看正在运行的容器
```
docker ps
docker container ls
```
查看所有容器
`docker ps -a`
`docker container ls -a`
运行一个容器并且运行在后台
`docker run -d centos`
进入一个已经运行的容器
`docker  exec -it  containerid /bin/bash`
或者
`docker attach containerid`
两者的区别就在于attach 在退出 容器后容器会停止运行 而 exec 不会
端口映射
`docker run -d -p 5001:5000  imagename`
5000为容器暴露的端口 5001 为宿主机端口
从 DockerFile 创建对象
`docker build -t myimage:v1 .`
-t: 指定镜像名称和标签 格式为 name:tag 最后一个. 代表当前目录
### 将一个镜像提交到镜像仓库 dockerhub
首先到https://hub.docker.com/注册一个账号
然后到命令行登录
`docker login`
选择要上传的镜像 重命名为指定格式
`docker tag imagename username/imagename:v1`
username/imagename 用户名和镜像名
v1 版本号
