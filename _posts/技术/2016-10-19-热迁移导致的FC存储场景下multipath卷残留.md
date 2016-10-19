---
layout: post
title: 热迁移导致的FC存储场景下的multipath卷残留问题分析
category: 技术
tags: openstack
keywords: multipath call failed exit 1
description: 
---

### 背景知识 ###

生产环境中，有个region使用的FC存储。  
运维同事发现热迁移失败。  
经过分析是卷残留导致的错误。  

### 问题溯源 ###

在nova-compute.log中有如下日志  
![](http://i.imgur.com/zuz8INI.png)  

经过分析，迁移失败的原因是，挂卷失败，挂卷失败的原因是因为上一次热迁移的时候，源节点存在卷残留。  
![](http://i.imgur.com/1Ct1Kuu.jpg)

一开始怀疑热迁移的结束时，源节点qemu进程还在，或者说libvirt状态有延迟  
导致还有io的情况下进行了拔盘操作，导致的卷残留。
即nova中这块代码
![](http://i.imgur.com/RCzPlvW.png)  

在实验室中，用fio加io，然后热迁移复现问题  
反复试验，发现qemu进程确实不再了，问题还是存在。


排查进一步下移，人工模拟nova中拔盘操作  
还是起fio  
然后kill掉fio进程  
使用multipath -f模拟拔盘  
![](http://i.imgur.com/N6oNM9b.png)

问题复现了  

反观nova中拔盘代码  
尝试了三次，增加重试次数，问题解决。  
![](http://i.imgur.com/kAviVQp.jpg)

### 问题处理 ###


这个问题是multipath中状态没有做到同步导致的问题。
由于修改其内容成本较高，暂用上面方法规避。