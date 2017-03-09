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
（磁盘是挂的裸盘）


## 附1 ##

设置numa的时候发现qemu不支持跨cpu号设置  
比如不允许1,3,5，7号cpu绑定为一个node  
![](http://i.imgur.com/zEMXYil.jpg)  

解决方法：  
cpu进行pin的时候讲计划在一个node的cpu都pin到连续  
![](http://i.imgur.com/vbiHnqO.jpg)

## 附2 ##

上述的测试只是模型测试  
真正上大数据业务的时候，发现测试结果依然有差距  

裸机测试（Pi）  
![](http://i.imgur.com/KNeYb1A.png)  

虚拟机测试  
![](http://i.imgur.com/ge71cjV.png)  

抓出信息，可见iowait较高  
![](http://i.imgur.com/xSLRXV1.png)  

virtio前后端ring间数据拷贝会占用大量资源  
hadoop跑起来也会占用大量资源  
导致两者相互抢资源  

最可怜的是当时主机只有12个cpu，开启了超线程有24核  
外挂了11块磁盘，每块盘2T左右  


优化方向？
