---
layout: post
title: live migrate on FC SAN
category: nova
tags: nova-default
keywords: 
description: 
---

## 一句话总结 ##

一句话说不清楚，还是看全文吧  

## 背景介绍 ##

在之前的博客总有关config-drive有个BUG需要处理，[具体看这里](http://www.hanbaoying.com/2016/05/28/config-drive-bugs-avoid.html)  

最终使用博文中的规避的思路，但是没有做hard reboot  
是把虚拟机xml中的cdrom盘的值置空规避  

这篇博文题目是热迁移，为什么莫名其妙的说这么一通注入？  
别着急，向下看  

## 问题介绍 ##

热迁移过程中，发现勾选block_migration会报错如下  
![](http://i.imgur.com/r3IPoJq.png)


若迁移过程中，发现不勾选block_migration会报错如下  
![](http://i.imgur.com/yIECkKA.png)

诡异的是，测试的同事，测试的时候一次是成功的，一次是失败的  

补充说明，由于这个环境后端没有使用ceph  
故注释掉了相关配置项，如image_type  
![](http://i.imgur.com/yvteDen.png)  

好，案情已经陈述完毕，让我们一探究竟。

## 问题分析 ##

针对两种情况逐一分析  

### 勾选了报错 ###

根据勾选了的报错内容，找到代码如下  
![](http://i.imgur.com/d87p97K.png)  
通过日志看block_device_mapping应该是有值的，看下其来源  
![](http://i.imgur.com/wpxcou4.png)  
在外面的compute.py中找到来源是  
![](http://i.imgur.com/MfhgA70.png)  
进去看，发现是从数据库中获取到信息  
![](http://i.imgur.com/pqrBysD.png)  
然后根据其形式转换（这里是volume方式）  
![](http://i.imgur.com/6xbh5mq.png)  

总结下:  
虚拟机在block_device_mapping表中有如下格式的盘  
source_type=image destination_type=volume  
source_type=snapshot destination_type=volume  
source_type=snapshot destination_type=volume  
热迁移不能勾选block_migration，否则报错  
至于为什么，看[这篇文章](http://lists.openstack.org/pipermail/openstack-dev/2014-June/038163.html)  
简单说，为了防止错误，禁止挂在volume的虚拟机执行热迁移  

### 没有勾选报错 ###

根据报错内容找到源码位于  
libvirt/driver.py的check_can_live_migrate_source方法中  
也就是说，想执行带block_migration的热迁移，但是存储和虚拟机目录都不共享  
这里会报错  
注：这里这么判断是不太合理的，或者说很粗暴的  
看最新版的nova代码，会发现对相关情况进行了细化判断  
![](http://i.imgur.com/EZa3f5W.png)  
is_shared_block_storage和is_shared_instance_path都是在哪生成的  

在本方法上面，分别_check_shared_storage_test_file和_is_shared_block_storage  
![](http://i.imgur.com/MiPqxhT.png)

_check_shared_storage_test_file我们就不具体看了  
逻辑是，在热迁移的目标节点上，在instances_path目录下创建一个临时文件  
然后在热迁移的源主机上，查看刚刚创建的文件，如果能看到说明共享，看不到说明不共享  

下面重点分析_is_shared_block_storage  
![](http://i.imgur.com/C7bgdFC.png)  

还记得一开始，我说过image_type注释掉了吗？
因为是rbd的情况下is_shared_block_storage直接返回True  
注释后，这里使用的是default，也就是qcow2格式，is_shared_block_storage会使用父类方法  
也就是返回False  

如果dest_check_data.get('is_volume_backed')  
也就是说卷启动的虚拟机，这里会进入get_instance_disk_info方法  

is_volume_backed是在外层调用的时候设置进来的  
![](http://i.imgur.com/IKKJDlC.png)  

聚焦到get_instance_disk_info方法  
![](http://i.imgur.com/ZuTa5aF.png)  

首先获取到虚拟机的xml然后传入给_get_instance_disk_info获取磁盘信息  
![](http://i.imgur.com/Wj1OhJb.png)  
获取磁盘的挂载点，放到volume_devices中
本例中挂载点vol['mount_device']为/dev/vda  
![](http://i.imgur.com/zHZ1PEB.png)  
![](http://i.imgur.com/fme2N1b.png)  
过滤掉所有的cinder-volume，看看还有没有其他的盘信息  

本例中由于使用了config-drive，生成了一个光驱盘  
![](http://i.imgur.com/qMOI8lr.png)  
也就是说上述代码_get_instance_disk_info会返回光驱盘的信息  
![](http://i.imgur.com/S36hmF7.png)  
也就是说除了cinder-volume还存在其他盘  
亦即_is_shared_block_storage最终返回False   
触发上述错误  

但是，为什么测试的同事这里会测试成功呢？  
本文一开始的时候，说过，由于使用config-drive,做了规避：  
虚拟机启动后，直接在xml中将光驱内容置为空  
在使用ceph的场景下，次问题不出发，解决了热迁移失败的问题  
但是，不使用ceph的情况系下，也就是上述过程  
会触发问题  

虚拟机做过hard-reboot  
上面的规避方法，将数据库中的instances表中的config-drive标记位置为空  
在hard_reboot过程中，重新构建虚拟机xml的时候，发现标记位为空，则不会生成光驱磁盘  

由此，判断卷启动的虚拟机，是否是共享存储时，会只有cinder-volume  
会得到共享存储情况为True

## 源码分析 ##

有关热迁移的代码分析，之前的博客已经有了，[具体看这里](http://www.hanbaoying.com/2016/05/03/%E8%99%9A%E6%8B%9F%E6%9C%BA%E8%BF%81%E7%A7%BB%E4%B9%8B%E7%83%AD%E8%BF%81%E7%A7%BB%28live_migrate%29.html)  
注意：上面那篇之前的博文是基于liberty版本分析的，这里这篇文章是基于Kilo版本  
虽然两个版本代码有重构，但是逻辑大同小异  

现将kilo版本的代码时序图输出如下  
![](http://i.imgur.com/kFSOYPV.png)