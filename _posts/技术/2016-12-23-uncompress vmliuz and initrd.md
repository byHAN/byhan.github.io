---
layout: post
title: 解压initrd和vmlinuz
category: 技术
tags: 
keywords: 
description: 
---

### 解压vmlinuz ###

    # od -t x1 -A d vmlinuz| grep "1f 8b 08 00"
    
    0013920 f3 a5 fc 5e 8d 83 80 b8 38 00 ff e0 1f 8b 08 00
    
    13920+12=13932
    
    # dd if=vmlinuz bs=1 skip=13932 |zcat > vmlinux
    # strings vmlinux|grep /sbin/
    # strings vmlinux|grep 'init=' -----查看第一个执行的程序，默认是/sbin/init
    # strings vmlinux|grep 'Linux version' -----查看内核版本号
    
### 解压initrd.img ###

    # cp initrd.img /tmp/initrd.img.gz
    # cd /tmp/ && gzip -d initrd.img.gz
    # mount -o loop initrd.img /mnt----2.4内核
    # mkdir initrd && cd initrd && cpio -ivmd <../initrd.img ---2.6内核


转载自[http://www.veryopen.org/?p=1477](http://www.veryopen.org/?p=1477)