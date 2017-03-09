---
layout: post
title: 虚拟机cpu和memory性能优化测评
category: 技术
tags: 
keywords: 
description: 
---

## 背景 ##

有兄弟在客户那测试虚拟机性能  
测试结果说cpu和内存损耗40%以上  

按理说使用了kvm创建的虚拟机，即使用的vt-x和ept，都是硬件进行的不应该损失这么大  

## 修正 ##

将cpu pin住  
![](http://i.imgur.com/jSmJBKu.png)

设置numa

![](http://i.imgur.com/wi2ecdH.png)


## 结果 ##

![](http://i.imgur.com/nmFwpVH.png)  

![](http://i.imgur.com/csG07A6.png)  

![](http://i.imgur.com/XF64Fx1.png)  

![](http://i.imgur.com/VG0fToA.png)  
(网卡是pci直通的网卡)  

![](http://i.imgur.com/nLzgDVB.png)
（磁盘是直通的裸盘）


## 附 ##

设置numa的时候发现qemu不支持跨cpu号设置  
比如不允许1,3,5，7号cpu绑定为一个node  
![](http://i.imgur.com/zEMXYil.jpg)  

解决方法：  
cpu进行pin的时候讲计划在一个node的cpu都pin到连续  
![](http://i.imgur.com/vbiHnqO.jpg)