---
layout: post
title: libguestfs详解
category: 技术
tags: 虚拟化
keywords: libguestfs
description: libguestfs
---

## 背景 ##

libguestfs提供了一种编辑镜像的方法  
当然也可以直接把镜像挂载到本地进行编辑，可参见[本人这篇博文](http://www.hanbaoying.com/2016/12/08/qemu-guest-agent.html)相关章节  

## 安装 ##

    yum install install libguestfs-tools -y  

## 使用简介 ##


    guestfish -a ubuntu-14.04.2-server-amd64-v20160126.img  

执行上述命令进入shell  

![](http://i.imgur.com/YSBLIeV.png)  


这时可以看到，启动了一个虚拟机（后面详细过程部分，详细分析启动过程）  

![](http://i.imgur.com/9Y7xs3M.png)  

然后就可以对镜像做相关的修改，是不是很简单  

## 原理 ##

1.执行guestfish -a会动一个进程，也就是那个shell壳子，姑且称之为main program  
2.运行run的时候，会创建一个child process，在child process中，利用libvirt启动一个称为appliance的虚拟机。  
3.在appliance中，运行了linux kernel和一系列用户空间的工具(LVM, ext2等)，以及一个后台进程guestfsd  
4.main process中的libguestfs和这个guestfd通过RPC进行交互  
5.由child process的kernel来操作disk image  

![](http://i.imgur.com/3hxLChO.png)  

## 详细过程 ##

如果想看详细的启动过程可以导出如下内容  

    export LIBGUESTFS_DEBUG=1  

#### 1.启动guestfishb并连接libvirt ####

![](http://i.imgur.com/nOq4rDy.png)  

#### 2.运行supermin5 ####  

此步骤中包含了拷贝内核相关内容到/var/tmp/.guestfs-0/appliance.d目录  

![](http://i.imgur.com/DoTydhY.png)  

#### 3.为创建appliance一块盘 ####

使用qem-img创建一块磁盘，作为系统盘  

![](http://i.imgur.com/eIt3QDf.png)  

#### 4.创建虚拟机的xml ####

为libvirt准备创建虚拟机所需的xml  
然后调用libvirt命令启动虚拟机  

![](http://i.imgur.com/gfD2LBP.png)  

#### 5.启用bios ####

设置bios相关内容  

![](http://i.imgur.com/qBUpFml.png)  

#### 6.启动initrd ####

![](http://i.imgur.com/nvLd1BJ.png)

#### 7.加载内核模块 ####

![](http://i.imgur.com/2wwvrUD.png)  

#### 8.设定root设备 ####

![](http://i.imgur.com/gWqXbVs.png)

#### 9.运行init ####

挂载相关文件系统，udev挂载设备系统  

![](http://i.imgur.com/Sv9i2Pp.png)  

#### 10.启动guestfsd ####

![](http://i.imgur.com/wseiPH9.png)

#### 11.开通端口 ####

C类库可以通过RPC连接这个端口  

![](http://i.imgur.com/hIzdrzy.png)


命令全集[看这里](https://rwmj.wordpress.com/2013/03/13/guestfish-now-supports-502-commands/)