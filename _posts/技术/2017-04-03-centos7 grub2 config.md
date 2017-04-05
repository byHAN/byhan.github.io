---
layout: post
title: 修改centos7默认启动项目
category: 技术
tags: linux
keywords: linux，grub2
description: 
---

### 1、修改配置文件，改变优先级 ###

将GRUB_DEFAULT=saved 改成 GRUB_DEFAULT=0  
（注意第一个是0，按照需要改成相关值）

### 2、配置文件生效 ###

    grub2-mkconfig -o /boot/grub2/grub.cfg 