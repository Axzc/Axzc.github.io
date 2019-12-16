---
title: ab压力测试工具
date: 2019-12-16 16:13:24
categories: 笔记
tags: [ab, 压力测试]
---
系统环境 ubuntu18.04LTS
- 什么是ab
ab是 apache bench命令的缩写是 apache 自带的压力测试工具

- 安装ab
```
sudo apt-get install apache2-utils
ab -V
This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/
```
<!--more-->
- ab 的参数
```
最简单使用: -c concurrency 并发数; -n requests 请求数量
发送post请求:  -p params.file  post需要的参数,  -T 指定 content-type
timeout:   -s 20
 -k: 在 http 请求的 headers 中加上 Connection: keep-alive, 以保持连接 
 -t: timelimit, 指定本次压测的最长时间限制
```
- 使用ab
```
ab -n 500 -c 1 -s 5000 -p post.txt -T 'application/json' http://127.0.0.1:18000/getpose2

This is ApacheBench, Version 2.3 <$Revision: 1807734 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking 127.0.0.1 (be patient).....done


Server Software:         nginx   // 测试服务器的名字
Server Hostname:        127.0.0.1  // 求的URL主机名
Server Port:            18000  // 端口

Document Path:          /get5GDispose2  // 请求路径
Document Length:        344 bytes  // HTTP响应数据的正文长度

Concurrency Level:      500  // 并发用户数，这是我们设置的 -n参数
Time taken for tests:   4.334 seconds  // 所有这些请求被处理完成所花费的总时间 单位秒
Complete requests:      5000  // 总请求数量 我们设置的 -c 参数
Failed requests:        0  // 请求失败的请求数量
Total transferred:      416 bytes  // 所有请求的响应数据长度总和
Total body sent:        10486303
HTML transferred:       344 bytes  // 所有请求的响应数据中正文数据的总和
Requests per second:   2284.44 [#/sec] (mean)  // 吞吐量，计算公式：Complete requests/Time taken for tests  总请求数/处理完成这些请求数所花费的时间
Time per request:       4333.569 [ms] (mean)  // 用户平均等待时间
Time per request:       4333.569 [ms] (mean, across all concurrent requests)
Transfer rate:          0.03 [Kbytes/sec] received  // 表示这些请求在单位时间内从服务器获取的数据长度，计算公式：Total
                        714.44 kb/s sent
                        714.47 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       0
Processing: 4333 4333   0.0  4333   4333
Waiting:    14329 14329   0.0  14329   14329
Total:      14334 14334   0.0  14334   14334
```
