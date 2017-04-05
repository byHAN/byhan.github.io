---
layout: post
title: ceilometer取不到memory.usage指标
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

## 背景知识 ##

自研的DRS（均衡负载）服务基于虚拟机内存使用情况做均衡。  
有同事说ceilometer的memory.usage取不到  

历经重重否定，发现是个大乌龙，记之以自省。  

## ceilometer调用关系 ##

memory.usage在ceilometer中是通过如下插件获取的：  

    memory.usage = ceilometer.compute.pollsters.memory:MemoryUsagePollster

根据ceilometer机制得知，相关指标的获取是调用get_samples方法  
![](http://i.imgur.com/tDYn5Uf.png)

我们这里使用的是libvirt找到对应的实现driver  
是调用memoryStats获取到返回值，根据available-unused得到的  
![](http://i.imgur.com/5ekWo09.png)

根据libvirt.py中对应virsh命令可知，其实就是类似于virh的dommemstat  
![](http://i.imgur.com/xAS5lkI.png)

由于环境受限，只好本地virsh启动虚拟机验证，如下图可见没有对应的available  
![](http://i.imgur.com/aHzrDZW.png)  

找个之前版本的环境验证，是有对应字段的  
![](http://i.imgur.com/Mzek6Sr.png)

## qemu版本过低？ ##

由于之前版本是好的，其hostOS是Ubuntu14，对应的qemu是2.2  
现在版本的hostOS是centos7,对应的qemu版本是1.5.3  
首先怀疑qemu版本过低导致？

从libvirt代码分析起，直接调用到了虚拟化层对应的driver  
![](http://i.imgur.com/a19lsy2.png)

我们这里虚拟化层是qemu，找到qemu_driver.c  
对应方法转换为qemuDomainMemoryStats  
![](http://i.imgur.com/L8NdZ1m.png)  

在qemuDomainMemoryStats中看到获取想要信息的前提条件是virtio_balloon  
![](http://i.imgur.com/LuumzY4.png)


先不管继续跟进qemuMonitorGetMemoryStats  
最终跟到如下部分，向qemu发送guest-stats获取到的值  
![](http://i.imgur.com/GTpD2tP.png)

然后找到qemu代码，继续分析  
virtio_balloon设备初始化的时候设置了guest-stats方法由balloon_stats_get_all方法承载  
![](http://i.imgur.com/d6D7c8f.png)

分析balloon_stats_get_all方法，如下，可见从对应数据结构中取出数据  
![](http://i.imgur.com/UOg0pDs.png)

答案是virtion_ballon的统计特性写入的，具体分析看[本人这篇博文](http://www.hanbaoying.com/2017/03/20/Virtio-Balloon.html)

分析到这里，我们拿qemu2.2和qemu1.5.3针对这块特性进行分析  
一分析，还真不一样。  
中间看见代码有重构反合等等  

好，下面在qemu2.2的版本上virsh启动虚拟机验证  
马蛋，还是不行，取不到所需的数据。  

## 和镜像相关么？ ##

分别取guestOS为centos和ubuntu测试，都获取不到所需数据

## 回归代码？ ##

再次细读virtio_ballon的代码  
通过如下方法设置一个周期  

    qemu-monitor-command 3 --pretty '{ "execute": "qom-set","arguments": { "path": "balloon0","property": "guest-stats-polling-interval", "value": 2 } }'

    qemu-monitor-command 3 --pretty '{"execute":"qom-get","arguments":{"path":"balloon0","property": "guest-stats"}}'  

![](http://i.imgur.com/8ovfmol.png)

## 问题来了 ##

这个guest-stats-polling-interval哪里设置的？  
逆向从qemu带libvirt到nova（就不一一贴代码了）  
可见，在nova的中_get_guest_config方法中补充了一个字段balloon.period  
![](http://i.imgur.com/KOwEWWT.png)

balloon.period的来源是CONF.libvirt.mem_stats_period_seconds  
![](http://i.imgur.com/6ldIlqg.png)

## 问题小结 ##

为什么本地virsh启动的没有呢？  
原因是virsh create的虚拟机没有填写ballon相关内容  
libvirt给补了一个balloon设备，但是没有开始统计特性  
![](http://i.imgur.com/kcuVqkE.png)  

## windows呢？ ##

windows装了balloon的驱动，上面讲的统计特性也开启了，依然采集不到相关信息  

经过分析balloon驱动，发现windows下也是实现了的  
但是为什么取不到呢？  
经过分析发现除了上面步骤，需要在winodows的guest中启动blnsvr服务  
这个服务由virtio_balloon提供  
