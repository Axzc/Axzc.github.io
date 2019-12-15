---
title: python 与 c++ 交互使用ctypes
date: 2019-12-11 11:34:52
categories: 笔记
tags: [python, c++, ctypes]
---
最近在使用python调用c++,在传递numpy的指针类型的参数时遇到了一些问题.所以研究了一下 ctypes.
首先是python调用c++ 编译的so文件,*so文件是c或c++在UNIX环境下的动态链接库,在windows下则为dll*
1.导入动态链接库
```
import ctypes
so = ctypes.cdll.LoadLibrary(os.getcwd() + '/libget.so')  # 使用cdll导入 动态链接库,在windows中使用windll
```
2.访问动态连接库中的函数
```
so.helloworld()
```
- 如何传递参数
ctypes 作为c++ 和 python 联系的桥梁,ctypes 来衔接这两种语言
网上有很多 ctypes类型、python类型和c类型的格式表
主要说一下指针
ctypes里面有三个跟指针相关的函数
|函数    |说明|
|-----------|:---------:|
|byref (x[, offset])     | 返回x的地址,x必须位ctypes类型的一个实例.相当于c中的&xoffset表示偏移量 |
|pointer(x)	    | 创建并返回一个指向 x 的指针实例， x 是一个实例对象。|
|POINTER(type)	   |	返回一个类型，这个类型是指向 type 类型的指针类型， type 是 ctypes 的一个类型。|

byref很好理解,传递参数的时候就用这个 
而pointer 和POINTER的区别就是 pointer返回的是一个实例,POINTER返回的是个类型
```
In [4]: a = c_int(7)
In [5]: b = pointer(a)
In [6]: c = POINTER(c_int)(a)
In [7]: b
Out[7]: <__main__.LP_c_int at 0x7ff54c34e8c8>
In [8]: c
Out[8]: <__main__.LP_c_int at 0x7ff54d402400>
In [9]: b.contents
Out[9]: c_int(7)
In [10]: c.contents
Out[10]: c_int(7)
```
那么如创建递一个numpy类型的指针呢
```
numpy_array = np.asarray([[11, 22], [33, 44]], dtype=np.float64)
numpy_p = numpy_array.data_as(POINTER(c_double)) #  转换为指针类型
```


