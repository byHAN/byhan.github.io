---
layout: post
title: libvirt hotplugged ide volume error
category: nova
tags: nova-storage
keywords: 
description: 
---

## 背景知识 ##

用ISO的镜像创建了虚拟机，虚拟机起来后挂载一个卷  
计划将系统装到挂载的卷上，然后解挂卷，再把卷作为虚拟机启动  
挂载卷的时候报错如下  
![](http://i.imgur.com/UIXJSNx.png)


## 一句话总结 ##

磁盘总线类型为IDE的情况下，libvirt不支持热插拔，虚拟机开机挂载会报错。  
也就是必须关掉虚拟机再挂载卸载卷  

磁盘的总线类型是怎么来的呢？  
是根据盘符决定的，vdb则bus是virtio，hdb则bus是ide。  

那盘符是怎么来的呢？
盘符是根据已有磁盘类型推算的，比如已有磁盘是vda,vdb，那么你这块盘将会是vdc  

那已有盘符是怎么来的呢？  
镜像启动虚拟机，镜像的类型决定的  
镜像类型是ISO则系统盘总线类型为IDE  

当然，也可以在虚拟机启动的时候，总过block_device_mapping选定总线类型  

## 源码分析 ##

### 系统盘总线获取 ###

在计算节点manager.py中的_build_instance方法中  
会设置设备名称，并更新到数据库中  
![](http://i.imgur.com/Ztwr14c.png)

_default_block_device_names方法如下  
首先通过index==0获取到系统盘，作为root_bdm  
依次从root_bdm和instance对象中获取device_name  
如果都获取不到，则使用默认的方法生成一个  
![](http://i.imgur.com/x4aDas8.png)  

分别获取磁盘和cdrom的总线类型  
默认情况下磁盘的总线类型为virtio，cdrom的总线类型为ide  
根据镜像的格式是不是ISO决定当前系统盘的总线类型  
即如果镜像的类型是ISO，则磁盘的总线类型为IDE，否则为virtio  
![](http://i.imgur.com/qNuQ5hW.png)

### 挂载盘总线 ###

挂载卷之前需要调用计算节点的reserve_block_device_name方法预占信息  
并且更新到数据库中  
![](http://i.imgur.com/zOj4u2Q.png)

真正的磁盘名字根据已有磁盘信息推算  
![](http://i.imgur.com/89Kd4zk.png)