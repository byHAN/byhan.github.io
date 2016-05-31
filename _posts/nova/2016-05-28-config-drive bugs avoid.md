---
layout: post
title: config-drive bugs avoid
category: nova
tags: nova-default
keywords: 
description: 
---

根据[上一篇博文](http://www.hanbaoying.com/2016/05/13/nova%E8%99%9A%E6%8B%9F%E6%9C%BA%E6%B3%A8%E5%85%A5%E7%9B%B8%E5%85%B3.html)  
我们知道可以使用config-drive进行数据注入

但是注入后，热迁移虚拟机的时候会报错  
根据选择文件格式的不同，报错如下  

## config-drive注入格式 ##

### iso9660 ###

通过分析代码发现，当格式为iso9660时，nova直接抛出了异常。  
因为，libvirt存在bug，并且建议使用vfat  
在Kilo版本是这样  
![](http://i.imgur.com/ibHG1m0.png)

在Liberty版本是这样  
![](http://i.imgur.com/hFGKFWy.png)

（两个代码的逻辑其实是有差异的，害的我感觉不通，还进行了调试）  

### vfat ###

既然社区推荐使用vfat，我们把格式改为vfat试试  
改完后会发现下面错误  
![](http://i.imgur.com/3eneLvB.png)

附disk.conf文件信息  
![](http://i.imgur.com/F3EzV7t.png)

## 解决方法 ##

能想到的几个解决方法：

### 1.共享存储 ### 
挂载文件系统  
指定instances_path为共享的文件系统

### 2.升级libvirt ###

### 3.规避 ###

我们知道使用了config_drive会在数据库的instances表中有个标志位  
把这个标志位置为空  
然后要么修改对应虚拟机的libvirt.xml  
要么，直接硬重启，会重新根据instance数据信息构造xml


