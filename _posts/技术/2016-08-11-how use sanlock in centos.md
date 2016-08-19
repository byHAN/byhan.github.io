---
layout: post
title: centos使用sanlock指导
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

需要安装sanlock，需要安装augeas



**重要**
需要关闭selinux  
否则sanlock会没有权限读取lockspace,会导致Lunix系统重启  
创建虚拟机报错如下：  
![](http://i.imgur.com/epAqhvV.png)  

在/var/log/messages中报错如下：  
![](http://i.imgur.com/q62PVMc.png)

通过阅读sanlock的代码发现是open方法调用lockspace的时候没有权限导致的  


关闭selinux的方法如下：  

    vi /etc/sysconfig/selinux

将SELINUX项目修改为disabled
（此方法需要重启才能生效）
![](http://i.imgur.com/2NIxTba.png)

或者直接执行命令setenforce 0  
（此方法重启会失效）

验证方法getenforce  
![](http://i.imgur.com/nTlWAaF.png)


**重要**

注意修改libvirt的相关配置，否则会热迁移报错
https://www.mirantis.com/blog/tutorial-openstack-live-migration-with-kvm-hypervisor-and-nfs-shared-storage/