---
title: python 的 GIL
date: 2019-09-03 14:59:31
categories: 学习笔记
tags: python
---

### 什么是 GIL
经常听到人说"python的多线程是个鸡肋, 推荐使用多进程"为什么会这么说,就是因为GIL的存在.
GIL的全称是Global Interpreter Lock(全局解释器锁)，据说在python设计的时候只考虑到了单核性cpu,而GIL就是为了保证线程之间数据的完整性和状态的同步.因为CPython只允许拥有GIL的线程才能运行, 也就是说同一时间内一个进程中只有一个线程可以访问CPU.
*在python多线程下 每个线程执行的方式:*
1. 获取GIL
2. 执行代码直到sleep或者python虚拟机将其挂起
3. 释放GIL
也就是说 如果一个线程想要执行就必须先获取到GIL,在python中 一个进程中只有一个 GIL这也就是一个进程在同一时间只能运行一个线程的原因
在python2.x中GIL 的释放逻辑是通过当前线程遇见IO操作或者ticks技术达到100(这个ticks可以看做是python 自身的一个计数器,专门用于GIL每次释放后都会归零, 这个计数可以通过 sys.setcheckinterval 来调整)进行释放
https://blog.csdn.net/weixin_34167819/article/details/91447783
https://zhuanlan.zhihu.com/p/20953544
