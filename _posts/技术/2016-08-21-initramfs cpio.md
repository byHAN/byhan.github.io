---
layout: post
title: initramfs解压
category: nova
tags: nova-default
keywords: 
description: 
---

## 背景 ##

initramfs是一个内存文件系统  
linux启动的时候引导程序（例如grub）将vmlinuz和initramfs加载到内存  
然后按照linux启动流程启动开始启动（启动流程自行google）  
启动到一定阶段，需要运行init进程，也就是第一个进程  
假设os是centos7的话,我们知道init进程是systemd进程  
问题是systemd是一个程序，是存在于存储中的文件，是如可运行起来的呢？因为刚启动的时候存储设备的驱动什么的文件系统都还没加载呢。

这就是initramfs的作用了  
将内核启动的一个最小集合打包到initramfs中  
启动的时候将initramfs拷贝释放到内存，这样内存中含有所需信息  
作为一个内存文件系统，协助内核正常启动  

（注：这里initramfs只关注内核2.6后的形式，老版本的如有使用自行google，本人[这篇](http://www.hanbaoying.com/2016/12/23/uncompress-vmliuz-and-initrd.html)是指定老版本）

## 解压缩 ##

查看得知initramfs是一个cpio压缩包  

![](http://i.imgur.com/0khdtdZ.png)


可是用如下方法解压，只得到一个GenuineIntel.bin  
这明显有问题  
![](http://i.imgur.com/OjTBIMH.png)


下载个binwalk详细看下就知道原因了

    git clone https://github.com/devttys0/binwalk.git
    cd binwalk
    python setup.py install 

原来我们上面解压的时候只得到的外层的东西  
真正的数据在29696这个偏移量处  
![](http://i.imgur.com/lhCzf9x.png)


将偏移量29696处开始的数据读出来，放到initramfs.gz中  
然后解压initramfs.gz这个文件  
得到cpio文件  
再通过cpio命令解压，得到最终的文件

    dd f=initramfs-3.10.0-514.6.1.rt56.429.el7.x86_64.img of=initramfs.gz bs=29696 skip=1
    gunzip initramfs.gz
    mkdir ramdisk
    cd ramdisk/
    cpio -i -d -H newc --no-absolute-filenames < ../initramfs

可以看到他的文件系统只是一个真正系统的子集  
![](http://i.imgur.com/so0z08L.png)


最后,initramfs是怎么来的呢？

安装os的时候，仔细观察，最后会提示生成initramfs  
他是根据主机架构等信息生成的  