---
layout: post
title: No PCI buses available
category: 技术
tags: 虚拟化层
keywords:
description: 
---

## 一句话总结 ##


## 背景介绍 ##

自己编译libvirt源码包：libvirt:1.2.12  
创建虚拟机的时候报错
虚拟机的xml没有指定controller  
No PCI buses available  
![](http://i.imgur.com/AnLVloW.png)

## 原因分析 ##

通过搜索源码发现No PCI buses available错误在domain_addr.c  
![](http://i.imgur.com/6lCnFjA.png)

开启gdb调试（具体看本人相关博文：gdb debug libvirt）  
在virDomainPCIAddressGetNextSlot处打断电  
触发断点  

可见确实是由于addrs->nbuses == 0导致的错误  
![](http://i.imgur.com/b22acXj.png)

gdb下bt一下，查看函数调用栈  
![](http://i.imgur.com/TnY9hD1.png)  
根据错误原因我们知道是addr->nbuses导致的错误  
通过函数栈，分析代码知道，addr是这里生成的  
![](http://i.imgur.com/wN55CqC.png)  
此处打断点，进入看就会发现addrs->nbuses就是函数的nbuses值  
在qemuDomainAssignPCIAddresses中nbuses来源于max_idx  

    nbuses = max_idx + 1;
    
从代码上分析，应该是没有进这里  
![](http://i.imgur.com/Lbzz5T2.png)  
导致max_idx是默认值-1，断点调试之  
![](http://i.imgur.com/KFHjKv2.png)  
11q  
7是VIR_DOMAIN_CONTROLLER_TYPE_PCI  

此外def->ncontrollers等于1说明只有一个usd控制器  
也就是说def中没有pci的控制器导致的错误  

接下来分享def->controllers的来源  
在回到刚才的bt结果的函数栈  
![](http://i.imgur.com/TnY9hD1.png)  
猜测def来源于qemuDomainCreateXML  
![](http://i.imgur.com/WR6O6dn.png)  

进入qemuDomainCreateXML经过层层调用，最终生成def是这里  
![](http://i.imgur.com/WMZaDzm.png)  
我们找到controller相关地方  
如果虚拟机xml中指定了controller  
![](http://i.imgur.com/dxcas7T.png)  

然并卵，我们这里没有指定controller，应该是哪里会生成默认的  
通过打断点不停调试，发现是在这里指定的controller  
![](http://i.imgur.com/vB3RaeN.png)  
看下方法virDomainDefPostParse源码  
![](http://i.imgur.com/oq7tKRT.png)  

通过调试知道，这里调用到qemu/qemu_domain.c:919  
![](http://i.imgur.com/Z9uoCqB.png)  
看下源码  
![](http://i.imgur.com/UlX4dFL.png)  
根据虚拟机的arch机器machine类型决定添加那种controller设备  

![](http://i.imgur.com/qx4GkpQ.png)  
可见，这里machine的类型是ubuntu,因此没有添加  

有些朋友可能会说，machine类型的就是引用的pc-i440fx-utopic  
如，用virsh capabilities查得  
![](http://i.imgur.com/8D6YzHP.png)  

虽然后期os.machine最终会用qemuCanonicalizeMachine替换为pc-i440fx-utopic  
但是controller此时已经设定好了，也就是书已经决定么有pci-root这个controller了  
![](http://i.imgur.com/sPGjcDK.png)

### 问题解决 ###

在上述代码中补充"ubuntu"  
如下，在ubuntu的libvirt-1.2.12的包中已经做了这个patch  
![](http://i.imgur.com/5jCc45H.png)  

![](http://i.imgur.com/GOynXrD.png)