---
title: 将ubuntu上的Shadowsocks代理分享给其他设备
date: 2019-08-20 16:50:07
categories: 技术分享
tags: privoxy
---

系统环境:ubuntu 18.04
这个方法可以将同一局域网中的多个设备共享其中一台已经安装 shadowsocks 的电脑,不用在每个设备中 都安装 shadowsocks,缺点是需要手动填写被共享设备的局部代理的具体要求...
<!--more-->
### 安装 shadowsocks
首先更新软件源
`sudo apt-get update`
安装 pip (如果已经安装可以略过)
`sudo apt-get install python-pip`
安装 shadowsocks
`sudo pip install shadowsocks`
### 修改配置文件
`sudo vim /etc/shadowsocks.json`
添加以下几行

```
{
    "server": "你的服务器ip",
    "server_port": 你的服务器端口,
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "password": "密码",
    "method": "aes-256-cfb",  //加密方式
    "fast_open": true,
    "timeout":300
}
```

### 启动 shadowsocks
` sudo sslocal -c /etc/shadowsocks.json -d -start`
这里可能会报错
`EVP_CIPHER_CTX was made opaque in OpenSSL 1.1.0. As a result, EVP_CIPHER_CTX_reset() appeared and EVP_CIPHER_CTX_cleanup() disappeared.
`
####解决办法:
使用vim 打开文件
` vim /usr/local/lib/python2.7/dist-packages/shadowsocks/crypto/openssl.py`
(这里的根据自己情况修改)
然后替换、保存
`:%s/cleanup/reset/`
`:x`
再次执行
` sudo sslocal -c /etc/shadowsocks.json -d -start`
这样shadowsocks就安装配置完成 接下来
### 安装配置 Privoxy
`sudo apt-get install privoxy`
### 配置 Privoxy
`sudo vim /etc/privoxy/config`
- 打开文件搜索 listen-address
- 找到 # listen-address 127.0.0.1:8118 取消注释
- 把 127.0.0.1改为 0.0.0.0 端口号改成一个 未占用的端口
- 搜索 forward-socks5t
- 将端口改为 shadowsocks 的端口
- 找到 # forward-socks5t / 127.0.0.1:9050
- 去掉注释 将端口 改为 shadowsocks 的端口 (就是 shadowsocks.json文件中的 local_port)
### 设置局部代理
打开 Privoxy 配置文件
`sudo vim /etc/privoxy/config`
直接搜索 confdir 将它的值设置为 /etc/privoxy
**这个位置很关键, 如果你想把配置文件都放其它的地方，比如放到/root/privoxy下，就把它设置成confdir /root/privoxy**
**创建一个 pac.action  文件 添加到 confdir目录中, 并且在 config文件中 添加一行 actionsfile pac.action**
```
{{alias}}

direct = +forward-override{forward .}

shadowsocks = +forward-override{forward-socks5 127.0.0.1:1080 .}
# 有了这一行 就可以把 config 文件中 forward 注释掉了

# default {direct} /

# shadowsocks

{shadowsocks}
# 以下为你要使用代理的网址 可以自行添加
.google.com
.gmail.com
```
最后重启 Privoxy
`sudo /etc/init.d/privoxy restart`
**最后拿出手机连接 wifi 选择手动代理 把ubuntu 电脑的ip地址和 刚刚 config文件中的listen-address的端口号填进去
**
大功告成!
