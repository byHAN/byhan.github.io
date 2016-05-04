---
layout: post
title: ceilometer与ceph对接
category: 技术
tags: openstack
keywords: 
description: 
---

# 背景知识 #

### 获取数据 ###

ceilometer有两种获取数据的方式

- 主动  
  ceilomter周期性的调用polling agents获取数据。
- 被动  
  各个组件发送事件信息到消息队列，ceilometer监听消息队列，获得数据。

### 入库 ###

ceilomete将收集到的数据发送到ceilometer-collector，由其将数据进行存储。
可以是文件，数据库，http,Gnocchi等。
一般情况下使用nosql的mongodb作为后端存储。

### 告警（liberty版本现在由Aodh项目接管） ###

可以定制告警，比如定制的告警是vm cpu使用率大于70%
轮询存储的数据，发现cpu使用率的数据大于70%时，发送告警。

![](http://i.imgur.com/y7aDgxU.png)

# 与ceph对接 #

![](http://i.imgur.com/nZJLUtJ.png)

# 当前版本 #

当前版本ceilometer已支持的指标如下：

![](http://i.imgur.com/4vx4irm.png)

# 验证 #

由于我们没有使用ceph中的对象存储，因此ceilometer无法获取到相应数据。

当前版本使用了ceph的块功能，确切说针对ceilometer来说块存储是隐藏在cinder之后的，也就是说卷的相关监控信息通过cinder检测到。