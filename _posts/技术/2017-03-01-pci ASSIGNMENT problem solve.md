---
layout: post
title: centos下网卡设备直通（VT-dpci passthrough）遇到的问题及其解决思路
category: 技术
tags: 
keywords: 
description: 
---

## 背景 ##

[本人这篇博文](http://www.hanbaoying.com/2017/02/21/pci-passthrough-in-centos.html)讲述了如何进行pci直通

![](http://i.imgur.com/3gaoAYT.png)  
PCI直通的时候，报上述错误  


## 参考文献 ##

请先[阅读这篇文章](http://vfio.blogspot.jp/2014/08/iommu-groups-inside-and-out.html)  
很多问题可以得到解答  


## 解决思路 ##

1. 将iommu_groups内的设备都直通给虚拟机（失败）  
![](http://i.imgur.com/gNqHdJ0.png)  
虚拟机启动后卡死在bios阶段

2. 将iommu_groups内设备拆分开，单个直通给虚拟机（失败）  
   经过查阅发现，iommu_groups是内核分配的，无法人工干预  
   
3. centos7.2上编译CONFIG_KVM_DEVICE_ASSIGNMENT（失败）  
4. 肖神在centos7.0上开启CONFIG_KVM_DEVICE_ASSIGNMENT编译**内核**（成功）  
   然后即可以使用libvirt中如下方式直通虚拟机  
   ![](http://i.imgur.com/ONWS60S.png)
