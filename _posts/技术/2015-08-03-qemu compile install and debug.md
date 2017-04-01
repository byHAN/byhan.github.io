---
layout: post
title: qemu编译安装调试
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

## 背景 ##

本文只是提供单步调试的基础  
具体商用版本的qemu打包，请参考相关博文  

## 编译qemu ##

下载好包并且解压后  

    ./configure --enable-kvm --target-list=i386-softmmu --enable-debug
    
    mak&&make install

## 调试qemu ##

    gdb --args /usr/local/bin/qemu-system-x86_64 -m 1024 -realtime mlock=off -smp 1,sockets=1,cores=1,threads=1 -no-user-config -nodefaults -rtc base=utc,driftfix=slew -global kvm-pit.lost_tick_policy=discard  -no-hpet  -no-shutdown -boot strict=on -drive file=/home/byhan/local.img,if=none,id=drive-virtio-disk0,format=raw  -device virtio-blk-pci,scsi=off,bus=pci.0,addr=0x2,drive=drive-virtio-disk0,id=virtio-disk0 --nographic -bios /usr/local/share/qemu/bios-256k.bin

然后就可以使用gdb调试了  
也可以使用layout src查看源码  

----------

(以下是自己第一次所撰写，已废弃，请弃读。)

日前，本人升级qemu，想到从官网下载对应的包，然后安装。
特记录如下：

    $ git clone git://git.qemu-project.org/qemu.git
    Cloning into 'qemu'...
    remote: Counting objects: 131834, done.
    remote: Compressing objects: 100% (29320/29320), done.
    remote: Total 131834 (delta 104345), reused 129302 (delta 102090)
    Receiving objects: 100% (131834/131834), 45.42 MiB | 300 KiB/s, done.
    Resolving deltas: 100% (104345/104345), done.
    Checking out files: 100% (2849/2849), done.
    $ cd qemu/
    $ ./configure
    $ make
    $ sudo make install


也可以从这里把包download下来，然后解压缩，配置，编译，安装。

需要注意几点：

需要注意安装路径问题
需要关注一系列配置项的配置问题（相关配置项看这里），由于不确认具体的配置项，用此路升级qemu并没有应用到版本中。  
即使知道配置项，如何整合到版本中？  
实时编译，会导致计算节点部署时间延长（10分钟以上），或者将编译好的直接应用，那后续平台更换，是否还要重新编译？  

--自问自答：每个os都有对应的包，取到对应os社区的包编会很简单的，可以参考本站其他博客。
