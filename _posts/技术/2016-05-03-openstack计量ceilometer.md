---
layout: post
title: openstack计量ceilometer
category: 技术
tags: openstack
keywords: 
description: 
---

计费服务一般分为三个层次：

**计量（Metering）**

获取到资源的指标信息，比如虚拟机的启动时间，停止时间，网卡流量等信息。
为上层的计费服务提供数据来源，在openstack中计量由ceilomete来处理。
详细见本文

**计费（Rating）**

根据收费的规则生成对应的话单，比如按照虚拟机的运行时长计费，根据启动时间和停止时间换算出费用。
为上层的收费服务提供对应的话单，在openstack中计费有模块CloudKitty来处理。
详细见另外一篇博文
**
账单（Billing）**

用户账号充值，对应服务的控制等。

 
 

由于本人接触openstack就是从ceilometer开始的，所以这里没有详细的分析，对ceilometer掌握以下几点：

ceilometer的本质是收集资源的指标(也就是sample)如，cpu使用率（指标cpu_util）等
收集数据有两种方式：
    
- 主动获取。
ceilometer启动定时器，定时的调用不同的Pollster插件getsample方法，进而调用各个组件服务接口去查询。
根据获取指标的不同启用不同的服务，比如计算节点上启动agent-compute服务收集计算相关指标，在控制节点上启动agent-central服务收集其他杂七杂八的指标。

- 被动获取。监听消息队列，获取到各个服务发送的消息。
指标管理。
收集的指标很多很杂，如何做到指标的易于扩展，易于管理？
通过pipeline.yaml（可以源码中，或者环境中找到这个文件看一下，然后找对应文档看下）
收集数据的服务启动时，将pipeline和Pollster组装成一个个的定时任务。

 
 
在以上基础上，看完下面文章，对ceilometer从架构层面，应该掌握的八九不离十了。
剩下就是细节问题和应用问题了。

http://www.cnblogs.com/sammyliu/p/4383289.html

http://docs.openstack.org/developer/ceilometer/

http://openstack.ru/wp-content/uploads/techtalks/Ceilometer_review.pdf



redhat的使用指南

https://access.redhat.com/documentation/zh-CN/Red_Hat_Enterprise_Linux_OpenStack_Platform/6/html/Administration_Guide/sect-Using_the_Telemetry_Service.html