---
layout: post
title: Raid卡配置丢失导致服务器无法启动
category: 技术
tags: 硬件
keywords: 
description:
---

### 背景 ###

今天干了件蠢事  
有台自用的服务器，bios密码忘记  
试图把主板电池清理bios密码  
不小心碰到了raid卡，把配置弄丢了  
导致机器起不来  

![](http://i.imgur.com/zYG0cY0.png)

### 修复 ###

将raid卡的线重插  
刷出raid卡，然后将配置有unconfigured bad勾选Make Unconf Good

![](http://i.imgur.com/eKvREEr.png)

![](http://i.imgur.com/NdEGH7y.png)

