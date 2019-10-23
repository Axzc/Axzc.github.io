---
title: nginx 的docker容器
date: 2019-09-15 09:09:46
categories: 分享
tags: [nginx, docker]
---

之前在学习nginx时总是小心翼翼的修改配置文件,外一哪里改错了一下就乱了,很是头疼.在学习了docker之后我就想到了nginx镜像,如果改错了干脆直接删掉.不用担心环境问题.
下面就是我搭建 nginx 容器的过程
首先在本地创建一个目录并且进入目录
<!--more-->
```
 mkdir nginx_container_demo
 cd nginx_container_demo/
```
然后在创建一个目录叫html,并且创建一个index.html,在里面写点什么 比如<h1>hello,world</h1>
```
mkdir html
vim index.html
```
然后启动nginx 容器 并且把我们本地的 index.html 挂载到 容器当中
```
 docker run \
  -d \
  -p 8080:80 \
  --rm \
  --name mnginx1 \
  --volume "$PWD/html":/usr/share/nginx/html \
  nginx
```
- -d 在后台运行
- -p 把容器的80端口映射到本地的8080端口
- --rm 容器停止运行后自动删除
- --volume  把本地的文件夹映射到容器
上面的 --volume 实际上只是吧 nginx默认的 欢迎页面替换成我们 index.html
替换网页文件还不算完, 还要修改配置文件,接下来把容器中的 nginx 拷贝的本地
`docker cp mnginx1:/etc/nginx .`
注意后面的 . 代表拷贝到当前目录
现在可以吧容器停止了
`docker stop mnginx1`
把刚刚拷贝到本地的nginx配置文件 改名为conf
`mv nginx/ conf/`
*我们重新启动一个容器,这次我们把本地的html和conf文件都映射到容器里面*
```
docker run \
-d \
-p 8082:80 \
 --rm \
 --name mnginx1 \
--volume "$PWD/html":/usr/share/nginx/html \
 --volume "$PWD/conf":/etc/nginx \
 nginx
```
在浏览器中输入127.0.0.1:8082 能看到我们index.html中 helloworld 就说明成功了
*当然这还不算彻底完成,我们还可以在conf文件中继续配置nginx.......*
