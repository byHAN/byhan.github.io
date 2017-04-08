---
layout: post
title: 虚拟机串口配置及其导出到主机pts和console.log
category: 技术
tags: 虚拟化
keywords: serial,pts,pty,tty,console,vm,console.log
description: config serial for vm
---

## 背景 ##

本文介绍如何配置虚拟机的串口  
如何将串口信息导出到主机的pts(伪终端)和console.log  
关于tty，pts(pty),console看[这篇博文](http://ytliu.info/blog/2013/09/28/ttyde-na-xie-shi-er/)还有[这篇博文](http://blog.chinaunix.net/uid-20543672-id-3225777.html)  

简单一说,看下图  
![](http://i.imgur.com/mgIkSKk.png)  
主机有/dev/console  
可以通过/dev/tty0将/dev/console引到tty1-ttyn(tty1-ttyn就是直接登录的设备，比如alt+f1可以切到tty1)  
可以将/dev/console引到串口上去，比如ttyS0
(本文就是修改linux启动信息，把console引导tty0和ttyS0上去)  

ssh等远程登录的时候，使用的是pts(如/dev/pts/0，多个ssh登录去pts目录下查看下就明白什么意思了)  
（本文就是把虚拟机的启动信息引导主机的pts上）


## 配置虚拟机串口 ##

这里以ubuntu14（引导程序是grub）作为例子  
vnc登录到虚拟机内部做如下配置  

### 1、修改/etc/default/grub ###

    ## Modify this line by leekwen
    GRUB_CMDLINE_LINUX="console=tty0 console=ttyS0,115200n8"
    
    ## Modify this line by leekwen
    GRUB_TERMINAL=serial
    
    ## Add this line by leekwen
    GRUB_SERIAL_COMMAND="serial --speed=115200 --unit=0 --word=8 --parity=no --stop=1"

### 2.刷新grub.conf ###

将步骤1的变动，刷新到启动配置中去  

    update-grub

### 3.创建 /etc/init/ttyS0.conf文件 ###

    # ttyS0 - getty
    #
    # This service maintains a getty on ttyS0 from the point the system is
    # started until it is shut down again.
    
    start on stopped rc or RUNLEVEL=[2345]
    stop on runlevel [!2345]
    
    respawn
    exec /sbin/getty -L 115200 ttyS0 vt102

### 4. 使用getty启动ttyS0 ###  

    sudo start ttyS0

### 验证 ###

虚拟机重启查看配置是否生效  
![](http://i.imgur.com/OOIuySz.png)

如果生效说明虚拟机的串口已经配置OK  

## 配置虚拟机串口到主机pts ##

虚拟机的xml中将虚拟机的串口0和主机的一个pts设备对应  

![](http://i.imgur.com/7nLKiRq.png)

## 配置虚拟机串口到主机文件 ##

虚拟机的xml中将虚拟机的串口0和主机的一个pts设备对应  
![](http://i.imgur.com/bz1qgkB.png)  

![](http://i.imgur.com/N38rMgy.png)