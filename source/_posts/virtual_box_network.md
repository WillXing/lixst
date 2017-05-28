---
title: VirtualBox中CentOS网络配置
thumbnail: /img/mine/Samui01.jpeg
date: 2017-05-28 12:00:00
tags: 
  - VirtualBox
  - CentOS
---

### 初始状态

在VirtualBox中安装CentOS7之后，网络无法访问，在CentOS7下使用 ip ad命令，可以看见如下输出：

![初始状态 ip ad 信息](http://upload-images.jianshu.io/upload_images/277896-0047ba4cd32a6b07.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如果我们尝试ping某个地址，肯定会直接unknown host：

![初始状态 ping 网络](http://upload-images.jianshu.io/upload_images/277896-b5d4819e2cc3c8f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 确定网络连接方式
因为我是在公司的网络下，公司对ip应该是有限制的，桥接模式`(Bridged Adapter)`肯定是不行的，因为会分配和宿主机同样的ip，所以我们使用NAT的方式来联网，但是NAT的话从宿主机到虚拟机的通信会有问题，那么可以再加一个Host-only的adapter来解决，这个后面提到。我们先来解决外网连接。

### 外网连接配置(NAT)
1. 虚拟机网络设置Adapter1中，如下进行配置
![虚拟机 network adapter 1](http://upload-images.jianshu.io/upload_images/277896-3b461ad856850d1d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
2. vi /etc/sysconfig/network-scrips/ifcfg-eth0, 添加如下内容
> DEVICE=eth0
HWADDR=08:00:27:6D:2A:D0
ONBOOT=yes
NM_CONTROLLED=yes
BOOTPROTO=dhcp

3. halt --r重启机器
4. 再次ping 一个网络地址

![ping again](http://upload-images.jianshu.io/upload_images/277896-d622a1ac1b6037a5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看见外网已经通了。

5. ping一下公司网络中的其它机器

![ping others](http://upload-images.jianshu.io/upload_images/277896-97d439f01cb1c7d9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到也是通的。

6. 最后我们试一下从自己的机器上ping通虚拟机。使用ip ad看一下eth0的信息
![eth0 info](http://upload-images.jianshu.io/upload_images/277896-d92635cc25db6d1a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后我们从本机ping 10.0.2.15: 

![ping 虚拟机](http://upload-images.jianshu.io/upload_images/277896-73555b23bfdb9ef7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
并不能ping通，但是如果你有多个虚拟机，你会发现虚拟机之间是可以ping通的。

### 主机和虚拟机之间的网络配置(Host-only)
1. 在VirtualBox中添加宿主机器网卡：
![add new host-only network](http://upload-images.jianshu.io/upload_images/277896-87a61d714f07cfff.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

2. 关闭虚拟机，并在虚拟机的设置中启用network adapter 2，并设置如下:
![虚拟机 network adapter 2 配置](http://upload-images.jianshu.io/upload_images/277896-fd6df296352a98e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

3. 打开虚拟机，vi /etc/sysconfig/network-scrips/ifcfg-enp0s8, 内容如下:
> TYPE=Ethernet
BOOTPROTO=dhcp
DEFROUTE=yes
PEERDNS=yes
PEERROUTES=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_PEERDNS=yes
IPV6_PEERROUTES=yes
IPV6_FAILURE_FATAL=no
NAME=enp0s3
UUID=9221f2fd-d20c-4655-8500-f34b5e6fb225
DEVICE=enp0s3
ONBOOT=no

4. halt --r 重启机器
5. ip ad 查看网络信息 如下：
![ip ad info](http://upload-images.jianshu.io/upload_images/277896-43336a9108b6a174.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看见多出来了第三项，enp0s8，地址是192.168.56.102，这个就是可以从宿主机ping通的地址，让我们来试一下.
6. 从宿主机ping 192.168.56.102 :
![ping 虚拟机](http://upload-images.jianshu.io/upload_images/277896-e9a9f4c6095b48b8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

已经通了。

那么到现在，我们的虚拟机已经可以访问外网，可以互相通信，也可以被我们的宿主机ping通了。