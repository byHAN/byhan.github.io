---
layout: post
title: 使用qga的好处
category: 技术
tags: 虚拟化层
keywords: virsh,libvirt，qemu guest agent,qga
description: 
---

## 背景 ##

之前的博文分析过virsh命令，[看这里](http://www.hanbaoying.com/2016/09/26/virsh-command(1).html)  
也分析过qga，[看这里](http://www.hanbaoying.com/2016/12/08/qemu-guest-agent.html)

也可以换句话说，使用qga有如下好处  

### 命令 ###

#### 1.关机 ####

    vish shutdown --mode=agent  

这种情况下回调用qga来进行关机操作  
相较于默认情况下的acpi事件注入进行关机有以下几个特点：  
1. 有些guest可能会屏蔽acpi事件
2. 使用qga可以确保把虚拟机带到一个干净的状态


#### 2.快照 ####

    virsh snapshot-create --quiesce

让虚拟机刷新io缓存到存储，确保处于一种稳定状态  
然后再开始做快照  
这样就可以不用fsck进行磁盘检查了，因为已经保证了数据的一致性

#### 3.文件系统 ####

    virsh domfsinfo
    virsh domfsfreeze
    virsh domfsthaw

查询guestos中挂载的文件系统信息  
冻结解冻文件系统

#### 4.裁剪文件系统 ####

    virsh domfstrim

使用fstrim命令裁剪文件系统中没有使用的块  
discard unused blocks on a mounted filesystem

#### 4.时间 ####

    virsh domtime

用来查询虚拟机的时间

#### 5.cpu ####

    virsh guestvcpus
    virsh setvcpus --guest

cpu相关的操作，设置cpu状态等  

#### 6. 网卡信息 ####

    virsh domifaddr --source agent

通过qga查询guestos中的网络信息


#### 7.设置密码 ####

    virsh set-user-password

设置密码