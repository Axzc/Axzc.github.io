---
title: ubuntu 安装 opencv
date: 2019-12-05 15:50:01
categories: 笔记
tags: opencv
---

安装环境 ubuntu18.04
---

1.下载opencv
这里下载的版本是 OpenCV – 3.4.8, 其他版本也基本相似
[下载链接](http://opencv.org/releases.html)
然后选择 源码安装 sources
![](/img/opencv.png)
<!--more-->

2.解压zip
```
unzip opencv-3.4.8.zip
cd opencv-3.4.8.zip
```

3.安装工具和依赖
```
sudo apt-get install cmake  
sudo apt-get install build-essential libgtk2.0-dev libavcodec-dev libavformat-dev libjpeg.dev libtiff4.dev libswscale-dev libjasper-dev  
```

4.cmake&&make
```
mkdir build 
cd build
// 执行 cmake
cmake ..
// 执行make
sudo make   // 这里要多等一会
// 执行 install
sudo make install
```
到这里 opencv 的 编译过程就结束了 还要配置一些opencv 的编译环境

5.编译环境
- 将opencv的库添加到系统路径
```
sudo vim /etc/ld.so.conf.d/opencv.conf
在文件中添加
/usr/local/lib  
sudo ldconfig  //使路径生效
```
- 配置bash
```
sudo vim /etc/bash.bashrc 
// 在末尾加入
PKG_CONFIG_PATH=$PKG_CONFIG_PATH:/usr/local/lib/pkgconfig  
export PKG_CONFIG_PATH
// 使设置生效
source /etc/bash.bashrc
//更新
sudo updatedb
```
6.测试
用一个小程序来测试 opencv 是否安装成功
opencv 官方 已经给了一个例子
```
进入目录
/opencv-3.4.8/samples/cpp/example_cmake
cmake .
make
./opencv_example
```
执行结束后可以看到
![](/img/opencv2.png)
这里是黑的是因为我这里没有摄像头
就代表成功了!!




