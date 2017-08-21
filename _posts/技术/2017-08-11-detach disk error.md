---
layout: post
title: 虚拟化层升级后磁盘无法卸载卷
category: 技术
tags: 虚拟化层
keywords: 
description: vmtouch
---

## 背景 ##

升级libvirt和qemu后，我们的CI环境用例跑不过了  
定位发现是磁盘无法detach  
让同事回退掉升级包，依然失败  
没有办法，直接从nova找到命令virsh detach-device然后验证  
按照环境原型  
我们是创建一个虚拟机，然后在这个虚拟机中安装nova-compute  
由于物理上同事没有开启nested，因此nova创建的虚拟机都是qemu类型的不是kvm的


验证结果如下：  
直接把nova安装到物理机上，用例无问题  
把nova安装到虚拟机中，用例就会失败  

最终确认是指令集不满足，虚拟机中虚拟机直接没有运行起来
（从libvirt到qemu到kernel一路gdb下来才发现，后来想想根本还是环境不好连，偷懒了）

最终确认是cpu-model的问题  

修改方法：
1. 开启nested特性，虚拟机使用kvm
2. cpu不指定为host-model模式  






