---
layout: post
title: ubuntu14中使用sanlock指导
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

## 一句话介绍 ##

本文介绍了如何在ubuntu中启用sanlock,以及相关的配置，验证等。

## 安装nfs-server ##

nfs作为共享存储，用来存放sanlock所需的租约文件。
注意：这个步骤在一个存储结点上执行即可。

#### 安装 ####

寻找一个存储节点，安装nfs-server服务  

    sudo apt-get install portmap
    sudo apt-get install nfs-kernel-server

注意：有可能nfs服务已经安装过了，可以跳过安装步骤。

#### 配置 ####

    mkdir -p /export/sanlock
    sudo vim /etc/exports  

在后面追加

    /export/sanlock *(rw,sync,no_root_squash,no_subtree_check)

![](http://i.imgur.com/2PyAUNY.png)  

这一行的含义  
/export/sanlock :nfs服务客户端共享的目录  
*：允许所有的网段访问，也可以使用具体的IP  
rw：挂接此目录的客户端对该共享目录具有读写权限  
sync：资料同步写入内存和硬盘  
no_root_squash：root用户具有对根目录的完全管理访问权限。  
no_subtree_check：不检查父目录的权限。 


#### 服务启动及其验证 ####

    #sudo /etc/init.d/nfs-kernel-server restart <---重启nfs服务
	#showmount -e <---显示共享出的目录

## 安装nfs-client ##

注意：在每个计算节点上执行如下步骤  
如果忘记安装，直接mount的话会报错如下  
![](http://i.imgur.com/psf2IDR.png)

#### 1.安装 ####

    sudo apt-get install nfs-common

#### 2.设置 ####

在/etc/fstab中追加（注意：ip换成刚才的存储节点的ip,切注意不要丢失bg参数）  
192.168.2.208:/export/sanlock /var/lib/libvirt/sanlock nfs hard,nointr,bg 0 0  
![](http://i.imgur.com/ZR7NKUb.png)  

使用mount挂载

`mount /var/lib/libvirt/sanlock ` 

![](http://i.imgur.com/oI082pC.png)

可以使用df -h 进行检查挂在情况  

## 安装libvirt ##

注意：本步骤在每一个计算节点执行

### 预置 ###

安装libsanlock-client1的包

    apt-get install libsanlock-client1

注意,这里可能提示是否修改配置，我们这里选择N不修改配置  

![](http://i.imgur.com/u0PoiP5.png)

### 安装libvirt ###

因为ubuntu的环境中，默认libvirt包中sanlock的相关内容没有打进去  
按照本人[这篇博文](http://www.hanbaoying.com/2016/07/28/compile-libvirt-ubuntu.html)方法打出包  
需要如下三个包(这里我们已经打出相关包，无需再重新打包)  
![](http://i.imgur.com/ZMcqdwh.png)

进入包所在目录，执行如下命令安装  

    dpkg -i *.deb

注意,这里可能提示是否修改配置，我们这里选择N不修改配置  

![](http://i.imgur.com/u0PoiP5.png)

使得动态链接库生效,执行如下命令

    ldconfig

安装成功后在/etc/libvirt下会有qemu-sanlock.conf的配置文件

### 配置libvirt ###

修改配置/etc/libvirt/qemu.conf,设置如下：

    lock_manager = "sanlock"

![](http://i.imgur.com/TiSgbZx.png)

修改配置/etc/libvirt/qemu-sanlock.conf，设置如下：

    auto_disk_leases = 1
    disk_lease_dir = "/var/lib/libvirt/sanlock"
    host_id = 1

注意：  
1.host_id每个计算节点需要保持不同  
2.如果sanlock的daemon进程不是以root启动的，需要在配置文件设置：

    user = "sanlock"
    group = "sanlock"

![](http://i.imgur.com/LcHfH1o.png)


### 重启服务 ###

重启sanlock和libvirtd的服务

    service sanlock restart
    service libvirt-bin restart

### 验证 ###

如果一切配置没有问题，在/var/lib/libvirt/sanlock会有__LIBVIRT__DISKS__  
![](http://i.imgur.com/InTZQre.png)

#### 功能验证 ####

请参考[本人这篇博文](http://www.hanbaoying.com/2016/08/19/test-case-for-sanlock.html)

## 参考文档 ##

[http://libvirt.org/locking-sanlock.html](http://libvirt.org/locking-sanlock.html)