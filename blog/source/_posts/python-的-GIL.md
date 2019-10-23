---
title: 关于 python 的 GIL
date: 2019-09-03 14:59:31
categories: 学习笔记
tags: python
---

### 什么是 GIL
经常听到人说"python的多线程是个鸡肋, 推荐使用多进程"为什么会这么说,就是因为GIL的存在.
GIL的全称是Global Interpreter Lock(全局解释器锁)，只有在Cpython解释器中存在GIL.据说在python设计的时候只考虑到了单核性cpu,而GIL就是为了保证线程之间数据的完整性和状态的同步.因为CPython解释器只允许拥有GIL的线程才能运行, 也就是说同一时间内一个进程中只有一个线程可以访问CPU.

<!--more-->

*在python多线程下 每个线程执行的方式:*
1. 获取GIL
2. 执行代码直到sleep或者python虚拟机将其挂起
3. 释放GIL
也就是说 如果一个线程想要执行就必须先获取到GIL,在python中 一个进程中只有一个 GIL这也就是一个进程在同一时间只能运行一个线程的原因
在python2.x中GIL 的释放逻辑是通过当前线程遇见IO操作或者ticks技术达到100(这个ticks可以看成是python 自身的一个计数器,专门用于GIL每次释放后都会归零, 这个计数可以通过 sys.setcheckinterval())进行释放
```
>>> import sys
>>> sys.getcheckinterval()
100
```
在python3.x中
```
>>> import sys
>>> sys.getswitchinterval()
0.005
```
在python3中 废弃了sys.getcheckinterval()方法改为sys.getswitchinterval() 是因为在python3.x中GIL释放条件改变了,在python2中ticks达到100就释放GIL的方法会导致CPU密集型的线程一直占用GIL,让IO密集型的线程一直都获取不到GIL不能解析.
### 有了GIL 为什么还要加锁
```
import threading
import time

num = 0

def add_a(count):
    global num
    for i in range(count):
        num+=1
    print('func a:{}'.format(num))


def add_b(count):
    global num
    for i in range(count):
        num += 1
    print('func b:{}'.format(num))


def main():
    t1 = threading.Thread(target=add_a, args=(1000000,))
    t2 = threading.Thread(target=add_b, args=(1000000,))
    t1.start()
    t2.start()
    time.sleep(1)
    print('最终结果:{}'.format(num))


if __name__ == "__main__":
    main()
```
输出结果:
```
func a:1305342
func b:1382821
最终结果:1382821
```
我们传递参数100w,也就是 add_a和 add_b 应该的都是 num+1 执行100w次的结果.按照常识来说结果应该是200w才对,但是结果是1382821(结果并不固定)
解决这种问题也很简单就是给add_a和add_b中的 num+=1 加锁
```
lock.acquire()
num += 1
lock.release()
```
*那么问题来了GIL 不是只允许同一进程中只有一个线程可以访问cpu,那么为什么还要加锁,去保证线程安全*
首先 t1和t2 两个线程 因为GIL不会去同时运行,而t1 和 t2 都是对同一个变量进行操作
 python解释器会把 i+=1 分为多步
1. 获取num的值
2. 将num +1
3. 讲运算结果赋值给num
很有可能t1执行到 num+1 还没有赋值 因为tasks达到100或者碰到io操作GIL被释放,而刚好t2获得了GIL 等t2执行玩num+=1的全部操作,再回到T1这时会把刚刚 num +1 的值赋给num,最后num+1值还是t2没执行num+=1之前的值.
这就导致了命名执行了 两次 num+=1 实际上只相加了一次的错误,所以需要加锁.
python文件的执行流程
.py文件  >> 字节码文件  >> python解释器执行字节码文件
python只会在 字节码文件中的空隙去切换线程,并且多久切换一次是可以切换.可以使用我们上面使用过的sys.getswitchinterval()进行设置默认是5ms
而函数的字节码可以通过dis.dis查看
```
>>> import dis
>>> dis.dis(test)
  2           0 LOAD_GLOBAL              0 (dict)
              2 CALL_FUNCTION            0
              4 STORE_FAST               0 (d)

  3           6 LOAD_CONST               1 (2)
              8 LOAD_FAST                0 (d)
             10 LOAD_CONST               2 (1)
             12 STORE_SUBSCR
             14 LOAD_CONST               0 (None)
             16 RETURN_VALUE

```
上面每一行就是一个字节码指令,在python3.6 以后所有字节码指令只占两个字节
