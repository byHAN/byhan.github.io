---
layout: post
title: mem_add(exec.c)
category: 技术
tags: qemu
keywords: 
description: 
---


![](http://i.imgur.com/xXTUJJp.png)


在exec.c文件中，有mem_add方法，初始化虚拟机内存结构的时候调用到  address_space_update_topology_pass或者address_space_init_dispatch会调用这里的方法，可参考上图。

查看mem_add，本方法就是把section分割为TARGET_PAGE_SIZE大小，然后调register_subpage或者register_multipage添加到对应的数据结构中。

注：TARGET_PAGE_SIZE为目标体系结构的页大小，这里以x86架构为例，是4k  
![](http://i.imgur.com/H6XYduf.png)

首先根据入参listener查到保护此listener的AddressSpace（具体看container_of方法，对于这种使用，表示服了）  
先关注下AddressSpaceDispatch *d也就是as->next_dispatch，具体as->next_dispatch是怎么来的，回头另开篇讨论。  
