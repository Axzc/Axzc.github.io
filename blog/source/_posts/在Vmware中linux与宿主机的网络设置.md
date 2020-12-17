---
title: 在Vmware中linux与宿主机的网络设置
date: 2020-05-20 13:52:49
tags: Linux 
categories: 技术分享
---  
## 在Vmware中linux与宿主机的网络设置
首先在在虚拟机工具栏点击选择虚拟机--->设置---->硬件---->网络适配器--->网络连接，选择桥接模式。  
<!--more-->  
![](/img/vmware.png)    
如果宿主机是自动获取IP的，那么这样设置连接之后，也会自动给虚拟机分配一个同局域网的IP地址，如果在主机能相互ping通，则说明配置成功！  
### 手动设置静态IP    
<hr>
首先以centos7为例  
要设置的静态IP的原因是因为centos7默认是以dhcp动态的分配ip，导致每次开机可能我们的centos的ip会发生变化，在以后连接的时候可能会不方便。  
  
首先使用`dhclient`命令为分配一个当前可用的ip地址  

查看分配的ip地址  

``` bash  
ifconfig  
ens33: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.7  netmask 255.255.255.0  broadcast 192.168.1.255
        inet6 fe80::9cfd:b2c8:f9e0:ae8c  prefixlen 64  scopeid 0x20<link>
        inet6 2408:8207:b426:73c0:c188:a2e9:92b6:d5f3  prefixlen 64  scopeid 0x0<global>
        ether 00:0c:29:54:e4:fb  txqueuelen 1000  (Ethernet)
        RX packets 40780  bytes 54077486 (51.5 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 19704  bytes 1765892 (1.6 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```  

可以看到 分配的ip为 192.168.1.7，接着我们去配置这块网卡，将这个ip设置为静态ip。  
打开 `vim /etc/sysconfig/network-scripts/ifcfg-ens33`  ens33为网卡的名字。然后修改添加一下内容  

``` bash  
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static  # 设置为静态
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=f0bc03b7-84fd-499a-b0a3-e7c1d02e9ab7
DEVICE=ens33
ONBOOT=yes  # 在启动时加载
# 以下为新添加内容
IPADDR=192.168.1.7  # ip地址
NETMASK=255.255.255.0  # 子网掩码
GATEWAY=192.168.1.1  # 网关， 和宿主机一致
DNS1=119.29.29.29              
```  

最后 `service network restart`   重启网卡 使其生效  
 需要注意的是 在使用linux 连接 宿主机的时候要注意宿主接的防火墙  
<hr>  
接下来是 在ubuntu 18.04 中的配置  
ubuntu 和 centos 的配置大致相同 区别在于 配置文件    

打开 `vim /etc/netplan/01-network-manager-all.yaml`  
```bash    
network:
  ethernets:
    ens33:     #配置的网卡的名称
      addresses: [192.168.1.7/24]    #配置的静态ip地址和掩码
      dhcp4: no    #关闭DHCP，如果需要打开DHCP则写yes
      optional: true
      gateway4: 192.168.1.1    #网关地址
      nameservers:
         addresses: [119.29.29.29]    #DNS服务器地址，多个DNS服务器地址需要用英文逗号分隔开
  version: 2
  renderer: networkd    #指定后端采用systemd-networkd或者Network Manager，可不填写则默认使用systemd-workd
```  
最后 使用 `sudo netplan apply` 使生效操作
