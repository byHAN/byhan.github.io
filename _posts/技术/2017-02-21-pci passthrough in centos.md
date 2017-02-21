---
layout: post
title: centos下网卡设备直通（VT-d,pci passthrough）
category: 技术
tags: 
keywords: 
description: 
---

### 背景 ###

本文指导带host os为centos场景下的设备直通

### 步骤 ###

#### 1.vt-d ####

需要CPU支持VT-d，主板也支持该技术。
打开bios中的VT-d设置。
（默认情况下基本支持和打开）

#### 2.kernel ####

内核启动开启iommu,修改/etc/default/grub,增加图中红色部分  
![](http://i.imgur.com/XG4dFNT.png)

使得配置生效，执行如下命令（注意：不执行此命令，上述修改不会生效）  

    grub2-mkconfig -o /boot/grub2/grub.cfg  

重启host主机

#### 3.准备硬件设备 ####

使用如下命令查找需要直通的设备

    virsh nodedev-list --tree

这里以网卡enp2s0f1为例  
![](http://i.imgur.com/8EcVn4X.png)

#### 4.启动虚拟机 ####

根据上述查询到的pci槽口号,上例中为000：02:00.1  
修改虚拟机xml，增加如下部分  
![](http://i.imgur.com/sjiw57o.png)  

    virsh create vm.xml

#### 5.验证 ####

host侧将虚拟机dump出来  
![](http://i.imgur.com/whP1AGU.png)  

vnc登录到虚拟机  
![](http://i.imgur.com/Zxca46A.png)  


### 原理 ###

![](http://i.imgur.com/QDXfvAM.png)  
 
CPU 都提供将 PCI 物理地址映射到客户虚拟系统的方法。  
当这种映射发生时，硬件将负责访问（和保护），客户操作系统在使用该设备时，就仿佛它不是一个虚拟系统一样。  除了将客户机映射到物理内存外，新的架构还提供隔离机制，以便预先阻止其他客户机（或管理程序）访问该内存。
（vt-d主要是基于DMA地址重映射） 

参考文献：  
[    https://www.ibm.com/developerworks/cn/linux/l-pci-passthrough/](https://www.ibm.com/developerworks/cn/linux/l-pci-passthrough/)  
[http://bean-li.github.io/KVM-installation-in-RHEL7/](http://bean-li.github.io/KVM-installation-in-RHEL7/)  
[http://blog.csdn.net/halcyonbaby/article/details/37776211](http://blog.csdn.net/halcyonbaby/article/details/37776211)