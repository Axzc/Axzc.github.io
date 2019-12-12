---
title: 解决postman 打开后崩溃黑屏问题
date: 2019-12-12 16:12:25
tags: postman
categories: 问题
---

系统环境: ubuntu 18.04
#### postman 叒出问题了!!!
这次是打开后崩溃黑屏
<!--more-->
![](/img/postman_black.png)
起初我并没有把这当一回事,毕竟ubuntu18.04 里面 大大小小的毛病总会有一点哈哈哈
直到我 重启postman, 重新系统,最后升级到最新版本.这个问题还是存在.....翻了翻github中的lssues 也没有找到类似的问题(最后找到了[lssues](https://github.com/postmanlabs/postman-app-support/issues/4764))
因为打开postman后并不是立即黑屏,而是在大概10几秒后不管有没有操作都会黑屏,我就想到有可能是内存占用的问题,用htop 看了一下果然如此.....没有截图

#### 最后解决办法
- 打开postman->view->developer->showdevtools(current view), 在左边的侧边栏找到clear storage

![](/img/postman_black2.png)
clear site data  重启postman, 解决!