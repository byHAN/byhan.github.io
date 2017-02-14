---
layout: post
title: cirrus漏洞分析CVE-2017-2615 XSA-208
category: 技术
tags: 
keywords: 
description: 
---

### 背景介绍 ###

最近360爆出一个qemu的安全漏洞，具体[可见这里](http://mp.weixin.qq.com/s/dzyc5OHdCeu532L4ARw9FA)

本文基于openstack kilo和qemu2.3分析  
源码分析openstack mitaka和qemu1.5.3同样存在此问题  

### 问题确认 ###

可以通过如下两种方式任何一个确认：  
1.带外从libvirt层面 virsh dumpxml  
如果类型为cirrus会存在此问题  
![](http://i.imgur.com/i3zF0zF.png)

2.带内从虚拟机内部 lspci  
如果vga型号为cirrus会存在问题  
![](http://i.imgur.com/dsSphx4.png)

### 问题根因 ###

可以详细见[漏洞描述](http://mp.weixin.qq.com/s/dzyc5OHdCeu532L4ARw9FA)  
![](http://i.imgur.com/yqnc0rU.png)

其实就是边界没有处理好

### 解决方法 ###

1.升级到问题解决的qemu版本，或者在原版本qemu上打补丁修复问题。  
2.虚拟机启动的时候不使用cirrus，使用其他的显示格式代替，如qxl格式代替  
（如果是openstack还可以通过镜像meatadata绕开,待拉通验证可用性，详细见下面nova分析）
  
### 源码层面问题溯源 ###

下面我们从上向下分析，即从nova到qemu

#### nova ####

我们知道，在nova中组装虚拟机的xml(就是给libvirt提供的虚拟机配置)是在_get_guest_config方法中实现的  
找到显示对应的部分,如下图  
默认情况下CONF.vnc_enabled是True,并且virt_type是kvm  
也就是说会走进上面的那个分支，进而add_video_driver会被设置为True  
![](http://i.imgur.com/U03DgMS.png)


下面重点关注_add_video_driver方法  
![](http://i.imgur.com/g8eN7uD.png)  

默认情况下guest的架构是x86,virt_type是kvm  
如果不启用spice，且镜像没有设置hw_video_model的话  
是vconfig.LibvirtConfigGuestVideo()里的默认值cirrus  
![](http://i.imgur.com/MtIcTJi.png)


综上所述，默认情况下，nova传给libvirt的显卡是cirrus  

因此，在nova层面修改，有如下两种方式：
1.修改nova代码，LibvirtConfigGuestVideo默认类型换为其他方式，如qxl  
2.运维层面规避，镜像使用hw_video_model这个值

这里使用qxl简单验证  
![](http://i.imgur.com/54NrULC.png)  

#### qemu层 ####

众所周知，qemu/kvm这种架构下，vt-d及其SRIOV外的基本设备模拟是在qemu里实现的  
在qemu的入口vl.c中的main()方法中，如果有vga相关的参数，会如下处理  
![](http://i.imgur.com/jLuSf43.png)  

接下来根据传入的vga参数调用select_vgahw设置vga相关硬件  
（当然，如果没有传入，qemu有自己的规则去生成默认的vga设备，不详述）  
![](http://i.imgur.com/TjtnXaN.png)

我们以cirrus为例，继续跟踪，其他类似  
![](http://i.imgur.com/Mq2RBy4.png)  

根据qemu的硬件设备加载逻辑（具体不详述，这块内容太绕，如果详细了解需要补齐下qemu启动逻辑，有机会后续专门输出文档）  
根据刚才的设置，会调用到/hw/display/cirrus_vga.c  

简单可以这么理解  
进程启动前，会调用到type_init，调用到cirrus_vga_info  
设置好类的初始化方法cirrus_vga_class_init  
cirrus的默认设备号是CIRRUS_ID_CLGD5446  
设置好设备实例化的方法pci_cirrus_vga_realize  
![](http://i.imgur.com/210J1I4.png)  

设备示例话的时候，会初始化设备，然后设置一系列的钩子函数  
也就是说后续有数据传输的时候，会调用钩子函数  
钩子函数实现了数据的传输，传输前会调用blit_is_unsafe判断当前传输是否合法  
问题就出现在这里，判断的时候边界没有处理清楚，导致会存在越界可能   

另外，具体数据如何处理的，具体涉及到qemu的io处理机制，详细描述又是一大块内容了（后续有机会补齐文档）


