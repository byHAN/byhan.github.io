---
layout: post
title: openstack-nova-conductor-rebuild
category: 技术
tags: openstack
keywords: 
description: 
---

# 背景知识 #

本节重点分析nova conductor服务的rebuild_instance方法后续服务
此处单独成文原因如下：

- 为evacuate服务（具体内容见链接）提供功能性支持
- 为rebuild服务提供功能性支持

# 源码分析 #

### nova-conductor ###

如果没有指定目标主机，则调度产生一个主机，作为目标主机

![](http://i.imgur.com/nwBEz4k.png)

然后调用对应目标节点上的nova-compute服务，执行rebuild操作。

![](http://i.imgur.com/XezFYUx.png)

### nova-compute ###

计算节点rebuild_instance方法，根据是否重创虚拟机，组织所需的网络和存储等内容。

假如需要新创虚拟机(recreate为true,也就是业务场景为evacuate)

#### 需要判断底层虚拟化是否支持recreate ####

如果不支持则抛出异常，意味着无法支持evacuate功能。

![](http://i.imgur.com/P3lBOnC.png)

由下图可见只有libvirt支持此功能。

![](http://i.imgur.com/LjyLlwa.png)

#### 判断当前目标节点是否存在待操作的虚拟机 ####

![](http://i.imgur.com/8oz75t2.png)

#### 存储性质是否共享存储 ####

如果入参没有指定存储性质，则从虚拟机底层存储获取到存储性质
如果有指定，则需要和底层实际的存储性质保持一致，不能瞎指定
共享存储情况下，直接用虚拟机既有的磁盘进行后续操作。
非共享存储情况下，需要获取到虚拟机的原有的镜像信息，后续创建用镜像。

![](http://i.imgur.com/rKJA02S.png)

#### 更新虚拟机的主机信息 ####

![](http://i.imgur.com/rFI4e4T.png)

#### 更新虚拟机状态 ####

![](http://i.imgur.com/BWFQnBr.png)

#### 网络处理 ####

重新创建虚拟机，则需要处理在新节点上的网络，将当期虚拟机的端口，绑定到目标机器上
![](http://i.imgur.com/KjpxUwF.png)

具体如下
![](http://i.imgur.com/eRpWVxr.png)

#### 磁盘处理 ####

- 获取到虚拟机对应的磁盘信息  
  ![](http://i.imgur.com/9AUOFeu.png)
- 提供一个解挂磁盘的方法  
  ![](http://i.imgur.com/ZKqR6QN.png)
- 提供一个挂载磁盘的方法  
  ![](http://i.imgur.com/oSFUwPy.png)

#### 组织参数，调用底层rebuild接口 ####

libvirt场景下没有实现rebuild接口，这里实际是调用的_rebuild_default_impl方法

![](http://i.imgur.com/5UFOLzZ.png)

#### _rebuild_default_impl方法 ####

- 解挂磁盘
- 如果需要，销毁已有虚拟机
- 挂载磁盘
- 调用driver新孵化一个虚拟机  

![](http://i.imgur.com/yHBOneI.png)

#### 后续处理 ####

![](http://i.imgur.com/wqgfIP3.png)
