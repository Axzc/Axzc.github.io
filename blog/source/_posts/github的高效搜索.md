---
title: github的高效搜索
date: 2020-01-06 09:41:04
tags: github
categories: 笔记
---

经常在github上搜索代码、项目，最简单的方法就是直接在主页的搜索框中查找，但这样搜到的很多结果都不是自己想要看到的。在这里总结一下搜索技巧相关操作。
### 在搜索结果中过滤
<!--more-->
![](/img/github.png)

### 高级搜索
高级搜索主要是在搜索栏中,写入想要的过滤条件.
![](/img/github_search_bar.png)
1.  搜索标题、描述、README 
查找标题中包含的关键字的项目
`in:name 关键字`
查找标书中包含关键字的项目
`in:descripton 关键字`
查找项目的README文件中包含关键字的项目
`in:readme 关键字`
2. star、fork 数量作为过滤条件
star 数量是这个项目的受欢迎条件
`stars:> star数量 关键字`
`stars:>3000 spring boot`
 还可以指定范围
`stars:1900..2000 spring cloud`
 fork 的语法与star 相同只需要吧star换成fork即可。
`fork:>3000 spring boot`
`fork:1900..2000 spring cloud`
3. 按占用磁盘空间大小搜索
`size:>=5000 关键词`
`size:>30000 spring cloud`
*** 需要注意的是这里为KB 30000 也就是30MB ***
4. 安装项目的push 时间搜索
这里很关键，通过push 时间我们可以知道这个项目是否还在维护。
`pushed:>yyyy-mm-dd 关键字`
比如查看元旦之后还在更新的项目
`pushed:>2020-01-01 spring boot`
还可以把 pushed 换成 created 改为 创建时间
`created:>2020-01-01 spring boot`
5. 按照语言搜索
language: 语言 关键词
`language:python wechat`
6. 搜索用户的仓库
`user:Axzc`
还可以使用多个语句 中间只要用空格分割就好了
`user:Axzc language:python`



