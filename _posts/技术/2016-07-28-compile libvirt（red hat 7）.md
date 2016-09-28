y
---
layout: post
title: compile libvirt（red hat 7）
category: 技术
tags: 虚拟化层
keywords: 
description: 
---
#### 准备编译工具 ####

安装rpmbuild
    yum install rpm-build

#### 准备源码包 ####

从[源码包路径](http://vault.centos.org/centos/7/updates/Source/SPackages/)下载对应的源码包，如图：
![](http://i.imgur.com/Tj0azhQ.png)  

#### 准备编译环境 ####

    rpmbuild -bb libvirt.spec
（需要安装依赖包）












