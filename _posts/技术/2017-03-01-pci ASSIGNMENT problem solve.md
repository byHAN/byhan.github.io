---
layout: post
title: centos下网卡设备直通（VT-dpci passthrough）遇到的问题及其解决思路
category: 技术
tags: 
keywords: pci passthrough,vfio,group is not visble,iommu,iommu_groups 
description: 
---

## 背景 ##

[本人这篇博文](http://www.hanbaoying.com/2017/02/21/pci-passthrough-in-centos.html)讲述了如何进行pci直通

![](http://i.imgur.com/3gaoAYT.png)  
PCI直通的时候，报上述错误  


## 参考文献 ##

请先[阅读这篇文章](http://vfio.blogspot.jp/2014/08/iommu-groups-inside-and-out.html)  
很多问题可以得到解答  

然后详细阅读[这个slider](https://www.linux-kvm.org/images/b/b4/2012-forum-VFIO.pdf)（超级推荐）  
内容主要讲述了vfio，讲述了为什么不使用老版本的kvm设备直通，而是推出了vfio  
原因有如下几点：  
- 1.架构上kvm是虚拟化层面的东西，不是设备驱动（ldd）  
- 2.安全层面的考虑，比如vfio是一套完整接口，非简单对pci系统的暴露，比如严格按照iommu对设备进行隔离访问，等等
- 3.vfio使用iommu_groups进行资源分组，这样解决了pci桥层面的分组，解决了transactions访问必须过iommu，因此更加安全
- 4.vfio设备属主移交至iommu_group层面，可以进行权限控制
- 5.vfio不仅仅支持x86,pci,kvm

## 解决思路 ##

1. 将iommu_groups内的设备都直通给虚拟机（失败）  
![](http://i.imgur.com/gNqHdJ0.png)  
虚拟机启动后卡死在bios阶段

2. 将iommu_groups内设备拆分开，单个直通给虚拟机（失败）  
   经过查阅发现，iommu_groups是内核分配的，无法人工干预  
   
3. centos7.2上编译CONFIG_KVM_DEVICE_ASSIGNMENT（失败）  
4. 在centos7.0上开启CONFIG_KVM_DEVICE_ASSIGNMENT编译内核（成功）  
   然后即可以使用libvirt中如下方式直通虚拟机  
   也就是说不使用vfio这种方式，而是使用kvm直通是可以的  
   原因是因为centos7相对于centos6把kvm直通设备给废弃了  
   ![](http://i.imgur.com/ONWS60S.png)  
5. 将iommu_groups内的设备全部使用 nodedev-detach（成功）  
   也就是说把对应设备的挂到vfio-pci上，使得和原来的设备驱动解关联。  
   附：前后对比如下  
   ![](http://i.imgur.com/IwhoPig.png)  
   ![](http://i.imgur.com/wS4dX8j.png)  



# 总结 #

归根结底，这个问题是由于  
使用了vifio，设备接口使用了pcie,然后设备多个function被划分到一个iommu_group  
导致vfio无法区分  
具体内容可以参考[redhat的文档](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Virtualization_Deployment_and_Administration_Guide/IOMMU-strategies.html)