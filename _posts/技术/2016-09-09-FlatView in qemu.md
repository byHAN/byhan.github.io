---
layout: post
title: QEMU内存管理之FlatView模型（QEMU2.0.0）
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

![](http://i.imgur.com/01ieL8f.png)

注：本文转自[刘峰博客](http://blog.csdn.net/leoufung/article/details/48781203)

在QEMU的内存管理中的FlatView描述了QEMU虚拟机内存平坦展开的情况  
首先看一下FlatView模型  
![](http://i.imgur.com/yIFEtGM.png)

1. 首先FlatView模型是通过FlatView和FlatRange两个对象组成。  
2. FlatView是该段内存的整体视图的管理结构，一个FlatView由一组FlatRange组成。  
3. 每个FlatRange代表了虚拟机上的一段内存，多个FlagRange就组成了一个内存视图，这些FlatRange在物理地址空间上不一定是相邻的。  

![](http://i.imgur.com/vEqo9Ka.png)