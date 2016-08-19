---
layout: post
title: gdb debug libvirt
category: 技术
tags: 虚拟化层
keywords:
description: 
---

## 一句话总结 ##

最近调试了libvirt，将过程记录如下 

## gdb简介 ##

openstack是用的python，我们调试的时候用的是pdb  
libvirt是c语言的，可使用gdb(或者其他类似工具)进行调试  

## 0.下载源码 ##

先下载libvirt的源码  

    wget http://libvirt.org/sources/libvirt-1.2.7.tar.gz

也可以直接谷歌，有时候这里下载速度较慢  

注意：  
上面取到的是libvirt的原生代码。  

如果需要取相关操作系统下的代码会有不同，如我的libvir是安装到ubuntu上的，则需要到[这里取](https://launchpad.net/~ubuntu-cloud-archive)  
我[这里](https://launchpad.net/~ubuntu-cloud-archive/+archive/ubuntu/kilo-staging/+packages)下载kilo版本对应的包  
![](http://i.imgur.com/N8LjKLW.png)
并且，如果是ubuntu下的话，编译请参考本人[这篇博客](http://www.hanbaoying.com/2016/07/28/compile-libvirt-ubuntu.html)  
然后安装gdb样式的deb包就可以调试了  

下面还是依照原声代码做通用介绍  

## 1.编译 ##

编译libvirtd的时候需要将调试打开  
在执行configure的时候带上--enable-debug=yes  
![](http://i.imgur.com/P2vbcxL.png)

具体如何编译，请参考本人相关博文  

## 2.gdb进程 ##

编译安装(具体参见本人相关博文)  
找到libvirtd的进程  
（libvirtd是守卫进程，调试的时候需要调试libvirtd,看有篇博文gdb virsh是不对的）  
![](http://i.imgur.com/PWo6Ain.png)

执行gdb（以上例进程id为25704作为例子）

    gdb libvirt 25704

![](http://i.imgur.com/ehErYQQ.png)

（这里由于把日志打开了，会有一系列的相关输出）

## 3.断点 ##

使用gdb的命令就可以进行调试了  
（具体命令请google）

这里以打断点为例子  
![](http://i.imgur.com/tow5zSf.png)

具体在哪打断点就要靠对源码的理解了。  
或者根据错误信息搜索源码  

## 4.触发 ##

另外开一个shell，执行virsh create local.xml  
![](http://i.imgur.com/dJbBaXQ.png)  
可见，已经进入到断点了  

可以执行bt查看调用栈  
![](http://i.imgur.com/jsq0Kax.png)

在不同的调用层次上9