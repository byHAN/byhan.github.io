---
layout: post
title: 控制寄存器概览
category: 技术
tags: 
keywords: 
description: 
---

最近看虚拟化的书经常遇到CR寄存器  
目前只知道CR3和页表相关  

现查阅维基百科[对应页面](https://en.wikipedia.org/wiki/Control_register)


汇总如下：

(x86平台)

### CR0 ###

可以用来修改处理器的基本操作，如下：  
![](http://i.imgur.com/cyx4TkF.png)

### CR1 ###

保留


### CR2 ###

保存PFLA(Page Fault Linear Address)的值  
当发生缺页页错误时候，这里保存产生缺页错误的虚拟地址。  
缺页错误处理通常会从这获取错误的虚拟地址。  

### CR3 ###

虚拟地址启用且CR0中PG位设置为1的情况下，CR3可以协助处理器将线性地址转换为物理地址。  
一般情况下为MMU提供页表的入口实现。

CR3的前20位页目录的物理地址基地址（PDBR，page directory base register）  
![](http://i.imgur.com/JhpycLC.png)

### CR4 ###

在使用实模式情况下，用来控制以下特性：  

虚拟8086模式  
IO中断  
内存大页（PSE，page size extension）,同时可以使用指令CPUID(CPU IDentification)识别内存大页的可用性。  
Machine Check Exception (MCE）, it is a type of computer hardware error that occurs when a computer's central processing unit detects a hardware problem. 通常情况下，因此MCE的原因是无法甄别的硬件错误及其超负荷运行。

![](http://i.imgur.com/tBpOFr2.png)


----------

（x86-64平台）

### EFER ###

EFER，Extended Feature Enable Register  
AMD引入的MSR用来启用 SYSCALL/SYSRET指令，后来也被用来进入和退出64位模式（long mode）

![](http://i.imgur.com/c6Qcdot.png)

### CR8 ###

用来给外部中断排序，也被称为TPR（task-priority register）  
系统软件可以通过CR8暂时阻塞低优先级的中断，例如在CR8中写入9，则会阻塞优先级为9及其以下的中断，却可以使得10以上的中断不受影响。  
设置为0运行所有外部中断，设置为15会禁止所有外部中断。



### [附MSR](http://www.intel.com/Assets/en_US/PDF/manual/253668.pdf) ###

![](http://i.imgur.com/5RgjzJ7.png)

