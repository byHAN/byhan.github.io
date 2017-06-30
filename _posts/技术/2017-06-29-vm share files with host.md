---
layout: post
title: 主机和虚拟机文件共享处理的几种方法
category: 技术
tags: 虚拟化
keywords: 
description: kvm
---

## 背景 ##

有些时候需要在主机和虚拟机间共享文件  
最简单的当然是虚拟机内陪ip,起ssh server,然后ssh进去  

以下提供了其他的一些思路：  

### 本地修改 ###

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

### libguestfs ###

参考本人这篇介绍[libguestfs的博客](http://www.hanbaoying.com/2017/02/26/libguestfs.html)  

### Samba 服务器 ###

QEMU 内置了一个 Samba server, 按如下方式启动, 即可启用.

    $ qemu-kvm -net nic -net user,smb=shared_directory ~/Image/XP.img

之后 GUEST 就可以用 10.0.2.4(默认) 上访问这个文件夹  
例如在 Windows Explorer 中, 可以用 \\10.0.2.4,  
在 gnome nautilus 中可以用 smb://10.0.2.4 访问.


### 通过主机的 Samba Server  ###

Samba server 打开, 在 Guest 里面就可以用 10.0.2.2 访问

    $ /etc/init.d/samba start

### 通过客户机 SSH(主机的端口转发)  ###

    $ qemu-kvm -m 1024 -redir tcp:3456::22 ~/Image/ArchLinux.img

QEMU 将会在 3456 端口监听, 收到数据后把所有端口转发到 Guest 的 22 端口.  
在 Guest 上启动 ssh server, 然后在主机上就可以用下列的指令访问了.  

    $ ssh IP_Address_Of_Host -p 3456
    
### qemu-nbd  ###

 qemu-nbd 是一个能使用 NBD 协议将 QEMU Image 导出的工具.

**加载 nbd 驱动 **  

某些版本的 linux 不加 max_part 参数会导致没有没有设备节点 /dev/nbd0p{1,2,3,4…} 等. 用 kpartx 也不行.  

    $ sudo modprobe nbd max_part=8

**连接 qemu-nbd**  

    $ sudo qemu-nbd -c /dev/nbd0 path/to/image/file # 注意要写绝对路径  
    $ fdisk -l /dev/nbd0 # 列出分区类型 
    $ mount /dev/nbd0p3 /mnt # 挂载虚拟磁盘的分区到本机

部分摘自[这里](http://blog.chinaunix.net/uid-26000137-id-3957732.html)  
突然有种茴香豆有几种写法的感觉  