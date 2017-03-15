---
layout: post
title: slabtop输出
category: 技术
tags: 
keywords: 
description: 
---

kernel通过slab来分配小容量的内存  
它引入了cache,slab,object等一系列概念  

通过slabtop命令可以查看  
由于对列内容不清楚，查询后归纳如下：  

![](http://i.imgur.com/WArfzUA.png)  


OBJS — The total number of objects (memory blocks), including those in use (allocated), and some spares not in use.  
ACTIVE — The number of objects (memory blocks) that are in use (allocated).  
USE — Percentage of total objects that are active. ((ACTIVE/OBJS)(100))  
OBJ SIZE — The size of the objects.  
SLABS — The total number of slabs.  
OBJ/SLAB — The number of objects that fit into a slab.  
CACHE SIZE — The cache size of the slab.  
NAME — The name of the slab.  