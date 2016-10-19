---
layout: post
title: openstack/ceilometer/gnocchi杂谈
category: 技术
tags: openstack
keywords: 
description: 
---

注1：

当前在ceilometer中event可以和meter分别存储。  
event数量少，且形式规整，也就是说怎么存都行。  
故下文所提的sample,meter均指的是除了event外的meter  
(ceilometer里面指标叫meter,对应的值叫sample,gnocchi里的指标叫metric)  

注2：官网地址：[http://gnocchi.xyz/architecture.html](http://gnocchi.xyz/architecture.html)

------------------------------------------------------------

# 序言 #

ceilometer设计的目标是作成一个通用框架，各式各样的数据都可以在这个框架内流动。  
指标数据从“虚拟机的cpu使用率”到“物理主机的风扇转速”等等，涵盖了云系统的方方面面。  
[（ceilometer的meter合集见这里）](http://docs.openstack.org/admin-guide-cloud/telemetry-measurements.html)

![](http://i.imgur.com/EZ82GKJ.png)  

另外，为了数据的弹性，ceilometer没有严格的要求数据格式，比如数据中允许存在meatadata，这就造成了数据的不规整。  
数据不规整给数据的存储（见上图左下角database部分）造成一定的困扰。  

首先SQL无法满足要求，因为数据长短不一，规划短了存不下，规划长了浪费空间，不好规划。  
有人提出，不是有NOSQL的数据库（比如mongodb）嘛，数据过来直接扔到里面。  

这样貌似解决了问题。  
可这种方案，数据量小的时候可行.  
ceilometer中收集到的数据量是亿为单位的。  
这样一来，检索的瓶颈集中到后端存储了。  
(mongo官方推荐一次把数据全查出，再逻辑处理，不推荐条件查询，这不扯嘛)。  

# 思路 #

其实归根结底，是一开始ceilometer没有想好数据如何处理、查询、存储。  
仔细想想，收集到的数据就是时间序列上的资源值，比如某个时间点特定虚拟机的cpu使用率等。  
有个时间点，有个值，上游的监控组件、告警组件、计费组件无非就是要这些东西。  
  
由此，jd整出了gnocchi项目。  
将数据裂化成两部分（index和storage）：  

## index driver：存资源索引值（resource) ##

如下图，是我环境中的资源列表，可见从计算到存储到网络，涵盖了几大类。  

![](http://i.imgur.com/0hdVwuw.png)  

show一个resource的结果，可见这个资源（也就是虚拟机网卡）关联了如图这么多指标。  

![](http://i.imgur.com/4vRbfx5.png)

![](http://i.imgur.com/Ci9gdjh.png)

![](http://i.imgur.com/yPArmco.png)

![](http://i.imgur.com/M3JVIm3.png)

# 小结 #

好，然我们回头梳理下所以内容，也就是下图：

![](http://i.imgur.com/i8VrRHR.png)

数据让gnocchi来处理，把数据拆成两部分，一部分通过storage driver存储，一部分通过index driver存储。  
这样检索数据的时候，通过index找到资源，然后再找到对应的metric，然后获取到metirc的measure信息  
复杂度由o(n)降为O(1)或者o(r)  

# 后端处理 #

![](http://i.imgur.com/MXg2sYQ.png)  
MetricReporting的审计进程，实时审计数据处理状态  
包含处理了多少metirc，还有多少metirc亟待处理。  


# 安装配置 #

## 安装 ##

#### devstack ####

（安装devstack时候建议用干净的系统，本人在此浪费较多时间）  
enable_plugin gnocchi https://github.com/openstack/gnocchi master  
enable_service gnocchi-api,gnocchi-metricd  
在devstack的local.conf中增加这两个配置项即可。  

#### pip ####

    pip install gnocchi
    pip install gnocchi[postgresql,ceph,keystone]

注意：pip安装的不是最新的版本，会存在一系列bug，需要进行升级。  

#### souce code ####

从源码安装

## 配置 ##

![](http://i.imgur.com/FW8TeZj.png)

# 与ceilometer对接 #

![](http://i.imgur.com/Z7YCSj5.png)

# 与Grafana对接 #

具体可参见本人Grafana的那篇博文


