---
layout: post
title: ovirt基于sanock的高可用(主机粒度HA)
category: 技术
tags: 虚拟化
keywords: 
description: 
---

译自[官网](https://www.ovirt.org/develop/release-management/features/storage/sanlock-fencing/)

## 背景 ##

当主机不可达的时候，ovirt会尝试隔离主机，从共享存储中清除不可达的主机，并期望使得主机恢复。  

ovirt3.4起支持两种类型的隔离（软隔离和硬隔离），软隔离是登录到被隔离主机重启对应服务（vdsm）;硬隔离提供一个隔离的agent，连接到电源隔离模块重启主机，前提是主机上有电源管理模块，并且正确配置（比如bmc）。  

通常情况下个的数据中心组网，分为管理面和存储面。  
那么存在管理面不通，而存储面通的情况，这种情况下上述隔离方式失效，只能人工干预。  
ovirt3.0起，每个主机运行sanlock daemon，且从共享存储上申请租约。  
隔离的时候从其中一个daemon进程向被隔离主机发送隔离申请，被隔离主机上的sanlock daemon重启主机。  


## 隔离目标 ##

1.释放主机占用的资源  
虚拟机使用共享存储，虚拟机还可能占用网络资源等，这些都是公共资源。 
这样就无法在其他主机上新启动虚拟机，因为可能两个虚拟机同时写，造成虚拟机脑裂后的数据踩踏。


2.使得主机恢复可用
因为隔离主机会重启主机。


## 隔离原理 ##

1.engine选择一个节点作为代理节点  
2.engine向选出的代理节点发送隔离请求  
3.代理节点发送sanlock隔离请求给sanlock daemon  
4.engine轮询隔离状态，直至主机被隔离成功  

