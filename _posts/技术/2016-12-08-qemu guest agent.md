---
layout: post
title: qemu guest agent验证
category: 技术
tags: 
keywords: 
description: 
---

## 前言 ##

版本封闭开发，被发配总部一个多月，出差没有心情写东西。  
药不能停，回来继续更新。

## 功能验证 ##

#### 准备qga包 ####

需要在getos中安装qga的包，由于虚拟机没法直接连接外网，计划直接下载好拷贝到虚拟机内。  
在一个能连接外网的环境中，可以直接使用yum命令下载包：  

    yum install --downloadonly --downloaddir=/home qemu-guest-agent

#### 挂载镜像 ####

这里使用一个centos的镜像作为guestOS  
把上述镜像挂载，把qga的包拷贝进去，顺手修改下root密码（因为不知道镜像的密码）

发现一个loop设备  

    loseup -f

将镜像虚拟成循环设备  

    losetup /dev/loop0 /home/CentOS-7-x86_64-GenericCloud-20160331_01.raw

使用kpartx装载镜像,装载之后，就可以在/dev/mapper/目录下看到  

    kpartx -a /dev/loop0

挂载文件系统  

    mount /dev/mapper/loop0p1 /mnt

挂载window的盘，可以使用如下方式  

    mount -t ntfs-3g /dev/mapper/loop0p2 /mnt/byhan

有脏数据的时候，可以这样恢复  

    ntfsfix /dev/mapper/loop0p2


然后就可以把qga的包拷贝到/mnt/home目录下了  
顺手把passwd中第一行的x去掉，这样root改为没有密码

#### 创建虚拟机 ####

这里使用virsh创建虚拟机，在虚拟机的xml中增加如下内容

    <channel type='unix'>
       <source mode='bind' path='/var/lib/libvirt/qemu/test.agent'/>
       <target type='virtio' name='com.163.spice.0'/>
    </channel>

创建虚拟机  

    virsh create vm.xml

可以检查，创建成功后，会在虚拟机内增加一个串口设备  
在主机上增加/var/lib/libvirt/qemu/test.agent这么一个unix socket  

#### 虚拟机启动qga ####

vnc登录虚拟机，在虚拟机内部安装qga  

![](http://i.imgur.com/yjuGxRp.png)

启动qga  

![](http://i.imgur.com/UrOrPjD.png)

#### 功能验证 ####

在主机上执行

    socat unix-connect:/var/lib/libvirt/qemu/test.agent readline

然后执行如下命令，查看结果 

![](http://i.imgur.com/yJCmQbc.png)

## 包编译 ##

[参考这里](http://zkread.com/article/715388.html)

    rpm -ivh epel-release
    下载 Microsoft VSS SDK: http://www.microsoft.com/en-us/download/details.aspx?id=23490
    提取Microsoft VSS SDK的源文件（qemu/scripts下提供了extract-vsssdk-headers，也可以在windows下解压后拷贝到linux上）
    wget http://wiki.qemu-project.org/download/qemu-2.6.0-rc1.tar.bz2
    yum install -y mingw64-pixman
    yum install -y mingw64-glib2
    yum install -y mingw64-gmp
    yum install -y mingw64-SDL
    yum install -y mingw64-pkg-config
    ./configure --enable-guest-agent --cross-prefix=x86_64-w64-mingw32-–with-vss-sdk=/vss_path
    make qemu-ga.exe

#### 验证 ####

    virsh qemu-agent-command byhan '{"execute": "guest-get-mem-usage"}'