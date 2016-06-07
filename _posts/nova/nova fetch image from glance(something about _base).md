---
layout: post
title: _base
category: 技术
tags: nova-storage
keywords: 
description: 
---

### 背景 ###

有同事反映  
计算节点虚拟机路径的_base下会有镜像  

按理说我们使用的ceph  
为什么还会把镜像下载到计算节点呢  
会好慢  

让我们一探究竟  

### 一句话总结 ###

镜像方式启动虚拟机  
如果镜像格式不是raw或者iso类型  
nova会把镜像download到计算节点的_base目录  
然后把镜像转换为raw格式，再启动虚拟机  

虚拟机的磁盘文件会在ceph

总结，使用ceph的情况下会支持qow2等非raw格式  
不过要经过下载，上传，格式转换等步骤，效率较差  

### 源码分析 ###

创建虚拟机在driver层面，和镜像相关的处理流程在  
livirt/driver.py文件中的_create_image方法中  
（有关这个方法的分析详细见之前[分享的博文](http://www.hanbaoying.com/2016/05/03/_create_image.html)）

这里聚焦到create_image方法的部分调用，见下面  
判断是镜像启动虚拟机，会走到如下流程  
![](http://i.imgur.com/HDHStxH.png)  

方法_try_fetch_image_cache方法会调用到clone_fallback_to_fetch方法  
而clone_fallback_to_fetch方法会调用backend.clone  
如果backend.clone抛出异常，则使用
下面分别分析

#### backend.clone ####

通过上下文可知，在使用ceph的时候  
backend是imagebackend.py中的Rbd  
也就是上述调用会调用到Rbd.clone()  
![](http://i.imgur.com/MVaP6Eu.png)  
会发现，调用的时候如果镜像不是raw也不是iso  
直接抛出异常  
然后执行下面的libvirt_utils.fetch_image

#### libvirt_utils.fetch_image ####

透传调用的fetch_to_raw方法  
![](http://i.imgur.com/eDn5DLz.png)


先从glance下载镜像到_base目录下  
![](http://i.imgur.com/VcR43j4.png)  

然后使用qemu的convert方法将镜像转换为raw格式  
![](http://i.imgur.com/oeIJF23.png)

### 系统盘 ###

传一份到ceph作为虚拟机的系统盘  
格式为vms/XXXXXX_disk  
![](http://i.imgur.com/choVHXz.png)  

其实也是调研的ceph方法  
![](http://i.imgur.com/XRI1I31.png)  