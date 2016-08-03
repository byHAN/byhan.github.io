---
layout: post
title: nfs install and config
category: 技术
tags: 技术
keywords: gdb调试libvirt
description: 
---

## 一句话总结 ##

最近验证sanlock,其租约存放地需要多个计算节点共享  
sanlock默认nfs作为共享存储  
现将相关内容记录如下

## 服务端 ##

服务器端提供存储，并设置相应的权限内容  
存储的提供者  

### 1.安装 ###

sudo apt-get install portmap  
sudo apt-get install nfs-kernel-server  

### 2.配置 ###

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

### 3.启动服务 ###

    #sudo /etc/init.d/portmap restart <---重启portmap，
    #sudo /etc/init.d/nfs-kernel-server restart <---重启nfs服务
    #showmount -e <---显示共享出的目录

## 客户端 ##

### 1.安装 ###

sudo apt-get install nfs-common

如果忘记安装，直接mount的话会报错如下  
![](http://i.imgur.com/psf2IDR.png)

### 2.设置 ###

在/etc/fstab中追加  
192.168.2.208:/export/sanlock /var/lib/libvirt/sanlock nfs hard,nointr 0 0  
![](http://i.imgur.com/ZR7NKUb.png)  

使用mount挂载

`mount /var/lib/libvirt/sanlock ` 

![](http://i.imgur.com/oI082pC.png)

## 参考文档 ##

[http://xouou.iteye.com/blog/2142842](http://xouou.iteye.com/blog/2142842)