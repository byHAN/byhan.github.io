---
layout: post
title: anti-virus for cloud platform
category: 技术
tags: openstack
keywords: openstack
description: 
---

## 背景 ##

最近为云平台测试杀毒软件  
针对相关的杀毒情况作一个汇总  

厂商涉及，圆，银水，亚洲信心，瑞雪丰年星光大道  

交流的结果，基本上分为有代理和无代理两种模式  

## 有代理 ##

在虚拟机里安装轻型的杀软  
然后杀软有自己的控制中心，与虚拟机内的杀软通过ip通信  

## 无代理 ##

所谓的无代理，也是需要在虚拟机里面安装轻型代理。  
然后在计算节点上安装杀软。  

通过内存共享技术，如Ivshmem把物理机的一块内存映射给虚拟机进行通信  
这也是为什么无代理无需基于ip通信的原因  

## 测试结果 ##

### 圆 ###

#### 有代理 ####

查杀Linux下的病毒，在离线状态下没有检测出病毒  
windows检查出7/50  

检出率不高，圆厂商提供的解释是，他们的查杀基于联网环境，当前测试是离线状态，效果不佳  
也就是说如果在线情况下，使用云查杀会有更好的表现。  

#### 无代理 ####
计算节点起服务，反向调用其总控程序  
计算节点和虚拟机通过内存映射实现通信  
windows 检出 44/50
Linux 检出138/150

### 银水 ###

不支持无代理，至少没有提这回事  
有代理模式windows检出29/50  
Linux支持特定版本，比如ubuntu64不支持，比如cetos7.1检测出101/102  
（注：Linux病毒样本采用的银水家的，故检出率较高）  

整个软件涉及较为合理，比如存在多级中心级联的情况下，可以逐层跳转  
对虚拟机停机开机感知较为敏感  

### 亚洲信心 ###

#### 有代理 ####

#### 无代理 ####
在计算节点上部署的软件不支持我们宿主机的OS，无法安装  
需要适配

### 瑞雪丰年星光大道 ###

他们家的产品不满足我们云的特殊要求，没有通过初选  
故没有测试  