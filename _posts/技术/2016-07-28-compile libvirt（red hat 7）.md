---
layout: post
title: compile libvirt（red hat 7）
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

#### 背景介绍 ####

本文主要介绍centos中，利用os有的源码包编译libvirt  
qemu等同理 

#### 准备编译工具 ####

安装rpmbuild

    yum install rpm-build

#### 准备源码包 ####

从[源码包路径](http://vault.centos.org/centos/7/updates/Source/SPackages/)下载对应的源码包，如图：  
![](http://i.imgur.com/Tj0azhQ.png)  

rpm2cpio  *.rpm |cpio -div  
解压 

#### 准备编译环境 ####

将源码及其补丁拷贝到/root/rpmbuild/SOURCE/目录下  

#### 编译 ####

具体的编译命令请参考rpmbuild，这里以bb为例  

    rpmbuild -bb libvirt.spec
（此过程需要安装依赖包）












