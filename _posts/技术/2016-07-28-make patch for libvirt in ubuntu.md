---
layout: post
title: make patch for libvirt in ubuntu
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

## 一句话介绍 ##

在debian系（ubuntu）中修改上游软件的源码，需要遵守约定  
这里介绍用dquilt制作patch的过程  

## 补丁流程 ##

### 1.新建补丁 ###

执行quilt new新建一个补丁  

    quilt new sanlock-add-lease-for-rdb.patch

![](http://i.imgur.com/7YTiPmT.png)

### 2.添加文件 ###

    quilt add ./src/locking/domain_lock.c

将要修改的文件添加到当前patch中  
![](http://i.imgur.com/CpGXgfD.png)

### 3.修正bug ###

修正软件包代码中的上游 Bug  
在源码中修改bug  
这里修改./src/locking/domain_lock.c
去掉相关的校验  

### 4.将修改记录到patch ###

通过refresh命令将上一步骤的修改保存到debian/patches中

    quilt refresh

![](http://i.imgur.com/v6iAy8Q.png)

这一步骤需要与上面的步骤在同一路径下执行  
执行完后可以到debian/patches中查看  
![](http://i.imgur.com/AHCgTbN.png)

也可以查看修改内容  
![](http://i.imgur.com/8tTsyDo.png)

### 5.添加修改描述 ###

使用 header命令为patch添加描述性内容  

![](http://i.imgur.com/RFn8ayk.png)

### 6.修改changelog  ###

使用命令dch更新changelog  

    dch -i

![](http://i.imgur.com/0HysEtk.png)

### 7.重编包 ###

按照博文：《compile libivrt(ubuntu)》重新制作deb包  

### 8.验证 ###

安装deb,验证修改的功能  


## 参考文档 ##

[https://www.debian.org/doc/manuals/maint-guide/update.zh-cn.html](https://www.debian.org/doc/manuals/maint-guide/update.zh-cn.html)