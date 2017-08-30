---
layout: post
title: 虚拟机磁盘cache导致的host os kernel崩溃
category: 技术
tags: 
keywords: 
description: 
---

## 背景 ##

一个大数据局点，跑性能测试的时候一上业务，说虚拟机会被杀死  
此局点是POC  
虚拟机通过virtio挂的裸盘


## 定位 ##

赶过去查阅dmesg发现有报内存分页错误  
按照现场同学描述，跑测试一分钟左右虚拟机就会死掉  
由于此虚拟机分配的内存很大，怀疑是启用的virtio_ballon从主机上偷内存，导致主机crash  
将虚拟机内存给调整小，重跑测试，时间一长还是崩了

free -m查看（这是后续补图，在现场没有截图）  
![](http://i.imgur.com/A9k17Py.png)  
buff/cache部分会急剧增加，导致free和available的内存不停减少  
继而吃swap分区，吃完swap则host os就重启了  

打开虚拟机的xml，发现直通磁盘的cache没有限制，也就是default  
（具体none，writeback,writethrough,default代表含义[见这里](http://www.hanbaoying.com/2017/06/28/iocache.html)）

与现场同学交流，他们在跑的测试是io密集型脚本  

现场将磁盘cache禁止掉  

![](http://i.imgur.com/GmjZwvC.png)


## 解决 ##

通过设置虚拟机磁盘cache为none  
问题规避


