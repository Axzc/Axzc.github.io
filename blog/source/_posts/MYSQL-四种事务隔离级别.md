---
title: MYSQL 四种事务隔离级别
date: 2019-11-04 12:35:02
categories: 笔记
tags: mysql
---

### 隔离级别（ISOLATION LEVEL）
在mysql 中 定义了四种隔离级别,分别是:
- READ UNCOMMITTED（未提交读）
- READ COMMITTED（读已提交）
- REPEATABLE READ（可重复读）(mysql 默认的)
- SERIALIZABLE（可串行化）

通过下面命令 设置隔离级别
```
SET [SESSION|GLOBAL] TRANSACTION ISOLATION LEVEL [READ UNCOMMITTED|READ COMMITTED|REPEATABLE READ|SERIALIZABLE]
```
<!--more-->
#### READ UNCOMMITTED（未提交读）
在未读提交中 ,事务的修改即使没有提交,其他事务也都是可见的.这也叫脏读.
*脏读:  如果事务a 读取到了 事务b 更新但未提交的数据, 然后 事务b 进行了回滚. 事务a读到的就是脏数据*
这个隔离级别会导致出现很多问题,虽然在性能方面是最优的.
READ UNCOMMITTED实践 开启两个MySQL SESSION，并将MySQL的默认隔离级别设置为READ UNCOMMITTED
```
SET SESSION TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;
```
![](/img/UNCOMMITTED.png)
(*这张图显示了具体过程 左边是 session a 右边是 session b*)
- **SESSION A 第一次查询point的值为90**
- **SESSION B 更新 point 的值为 10**
- **此时 SESSION B 的更改未提交**
- **SESSION A 再次查询point 的值为10**


####  READ COMMITTED（读已提交）
大多数DBMS 使用的都是 READ COMMITTED (MYSQL 不是)
大概含义就是 一个事务只能看到已经提交的事务的修改结果,换句话说 同一事务中两次查询结果可能是不一样的.
切换隔离级别:
```
SET SESSION TRANSACTION ISOLATION LEVEL READ COMMITTED;
```
![](/img/COMMIT.png)
(*这张图显示了具体过程 左边是 session a 右边是 session b*)
- **SESSION A 第一次查询point的值为10**
- **SESSION B 将point的值更新为90**
- **SESSION A 第二次查询point的值为10**
- **SESSION B 提交事务**
- **SESSION A 第三次查询point的值为90**
**可以看到 当SESSION B 事务提交后, SESSION A 就能读取到 SESSION B的数据**


#### REPEATABLE READ（可重复读）
可重复读是MYSQL 默认的事务隔离级别, 他解决了脏读的问题,这个级别保证了 在同一事务中多次查询相同的语句结果是一致的.
切换隔离级别:
```
SET SESSION TRANSACTION ISOLATION LEVEL REPEATABLE READ;
```
![](/img/REPEATABLE.png)
(*这张图显示了具体过程 左边是 session a 右边是 session b*)
- **SESSION A 第一次查询point的值为10**
- **SESSION B 将point的值更新为90**
- **SESSION A 第二次查询point的值为10**
- **SESSION B 提交事务**
- **SESSION A 第三次查询point的值为10**
可以看到无论SESSION B 怎么改变, SESSION A在事务开启后, **同一查询语句的查询结果都是一致的**


#### SERIALIZABLE（可串行化）
SERIALIZABLE 是最高的隔离级别,可串行化 会在读取的每一行数据上都加锁,所以会导致大量锁等待和超时时间,所以很少用到这个级别.
切换隔离级别:
```
SET SESSION TRANSACTION ISOLATION LEVEL SERIALIZABLE;
```
![](/img/SERIALIZABLE.png)
(*这张图显示了具体过程 左边是 session a 右边是 session b*)
- **SESSION A 第一次查询point的值为90 只有这一条结果**
- **SESSION B 插入id=2的数据，因为SESSION A的事务还未提交，此时锁等待。**
- **SESSION A 第二次查询point的值为90 仍只有这一条结果**
- **SESSION A提交事务，在提交事务的瞬间SESSION A释放锁，SESSION B锁等待结束**
- **SESSION B 提交事务 **
- **第三次查询point 变为90,和60 两条**
从上面的过程我们可以看到，"可串行化"是通过对每一行数据都加锁,这种方式效率非常的低，很容易造成较长时间的锁等待。
