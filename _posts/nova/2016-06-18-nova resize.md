---
layout: post
title: nova resize
category: nova
tags: nova-default
keywords: 
description: 
---

## 一句话总结 ##

## 流程概览 ##

![](http://i.imgur.com/YaeJmkU.png)  
需要注意，conductor-api进行RPC调用conductor的时候调用的是**migrate_server**

真正resize的时候，compute-source与compute-dest是同一台机器  
由于这里和冷迁移代码复用，会有区分  


可以看到前端多个业务复用同一套代码  

## 源码分析 ##

