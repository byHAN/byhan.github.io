---
layout: post
title: vm of openstack reset to UTC time when reboot
category: nova
tags: nova-default
keywords: 
description: 
---

## 一句话总结 ##

windows类型的虚拟机需要在glance中设置metadata  
否则，每次重启，时间都会被重置为UTC时间  
（经过验证centos，在不增加metadata的前提下，时间不会被重置）  
![](http://i.imgur.com/od0RqAT.png)


## 背景知识 ##

今天在虚拟机内部部署业务的时候发现时间会被重置  
明明修改为本地时间了，一会就会被重置为UTC时间  
具体什么原因，让我们一探究竟  

## 溯源过程 ##

记得之前看到过libvirt的虚拟机的XML中有时间相关选项  
![](http://i.imgur.com/N0Aq9lV.png)  

查阅[libvirt的文档](http://libvirt.org/formatdomain.html)发现，设置为clock的offset设置为utc  
在虚拟机重启的时候时间会被重启  

接下来在nova中查看，生成clock的部分代码  
会发现在driver的生成xml的方法中_get_guest_config  
![](http://i.imgur.com/V7HXOur.png)  

可以看到os_type为windows时进行了特殊处理  
将时间设置为了localtime  

那么os_type是哪里来的，逆向追踪  
最终发现在compute/api中有相关代码  
![](http://i.imgur.com/x1rs8T4.png)  
os_type从对应的image中的metadata中读取  

## 其他实验验证 ##

1.把windows相关镜像加上os_type：windows后时间不会再被重置  
2.linux不论加不加时间都不会被重置  

通过上面分析可知道，加上os_type的话，会使用localtime  
否则会使用utc

## 时间详解 ##

虚拟机的时钟是基于宿主机的时钟的。  
window需要设置为localtime   
windows外的大多数虚拟机希望物理时钟设置为UTC  

![](http://i.imgur.com/Xmywf4A.png)