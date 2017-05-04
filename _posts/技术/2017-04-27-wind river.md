---
layout: post
title: 风河虚拟化技术点分析
category: 技术
tags: 虚拟化层
keywords: wind river
description: 
---

## 背景 ##

从网上发现一篇风河的文档，不知道是否涉密，侵删  
先将虚拟化相关内容输出如下

## 内容 ##

#### kvm ####

![](http://i.imgur.com/VSnVHQe.png)  
![](http://i.imgur.com/Pdw3aCY.png)
1. RT_PREEMPT  
   我们知道linux是一个弱实时的操作系统，真正实时的操作系统参考vxworks。  
   这个patch通过优化一系列的锁机制，是的linux变身为实时os  
   [详见这里](https://rt.wiki.kernel.org/index.php/RT_PREEMPT_HOWTO)
2. Preemption Timer  
   intel的此机制，它可以使得虚拟机周期性的vm exit到vmm  
   具体可以参考[这篇博客](http://blog.csdn.net/xelatex_kvm/article/details/17761415)
3. interrupt remapping  
   intel的中断重定向，vt-d的核心  
   具体可以参考[intel的指导文档](http://www.intel.com/content/dam/www/public/us/en/documents/product-specifications/vt-directed-io-spec.pdf)  
4. modifications to vm enter and vm exit  
   vmenter和vmexit的条件那么多，猜测这里应该是优化了相应的进入退出条件，提高实时性
5. msi irq affinity
   这里应该是优化了msi设备的中断对于cpu的亲和性  
   [IRQ-affinity看这里](https://www.kernel.org/doc/Documentation/IRQ-affinity.txt)
   

#### guest-api-sdk ####
![](http://i.imgur.com/eqZKNI8.png)  
![](http://i.imgur.com/qm09Umm.png)  
风河提供了一套接口，可以用来实现虚拟机的心跳，伸缩，虚拟机组间的；消息传递  
相关文档的[参考代码见这里](https://github.com/Wind-River/titanium-cloud/tree/master/guest-API-SDK/16.10)


#### 虚拟机性能 ####

![](http://i.imgur.com/I5kheVh.png)


#### 伸缩 ####

![](http://i.imgur.com/WZ7Qb8T.png)
1. 热插拔vcpu，此特性需要guestOS支持（横向扩展）  
2. 手动或者使用heat自动伸缩虚拟机数目（纵向扩展）

----------

附电信级需求  
![](http://i.imgur.com/wCA5Igs.png)