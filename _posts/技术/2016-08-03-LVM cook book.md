---
layout: post
title: LVM操作指南
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

本文转自参考文档

LVM：logical volume manager(逻辑卷管理)；LVM屏蔽了底层磁盘布局，方便于动态调整磁盘容量。

## 创建逻辑卷的步骤 ##

1）通过fdisk 工具将磁盘转换为linux分区；  
2）通过pvcreate命令将linux分区转换成物理卷（PV)；  
3）通过vgcreate命令将创建好的物理卷处理成卷组（VG)；  
4）通过lvcreate命令将卷组分成若干个逻辑卷（LV)；  
5）对逻辑卷进行格式化，挂载，动态调整逻辑卷的大小，并且该操作不会影响逻辑卷（Lv)上的数据。  

## 物理卷(PV)创建及管理具体操作步骤 ##

### 1.先查看linux分区，将未使用空间转换为物理卷(先使用fdisk建立普通分区)  ###

    [root@RHEL5 ~]# fdisk -l /dev/sdb   #查看linux分区情况
    
    Disk /dev/sdb: 21.4 GB, 21474836480 bytes
    255 heads, 63 sectors/track, 2610 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    
       Device Boot  Start End  Blocks   Id  System
    /dev/sdb1   1 500 4016218+  83  Linux
    /dev/sdb2 5011000 4016250   83  Linux
    /dev/sdb310011500 4016250   83  Linux
    /dev/sdb415012610 89160755  Extended
    /dev/sdb515012610 8916043+  83  Linux 

备注：/dev/sdb是一块新增加的磁盘，上面没有任何数据，也未挂载  

### 2.将linux物理分区转变为物理卷  ###

     [root@RHEL5 ~]# pvcreate /dev/sdb{1,2}  #将物理分区/dev/sdb{1,2}转变为物理卷
      Physical volume "/dev/sdb1" successfully created
      Physical volume "/dev/sdb2" successfully created 

### 3.使用Pvscan查看物理卷信息 ###

    [root@RHEL5 ~]# pvscan   #查看物理卷信息，会显示所有物理卷信息
      PV /dev/sda2   VG VolGroup00   lvm2 [39.88 GB / 0free]
      PV /dev/sdb1   lvm2 [3.83 GB]
      PV /dev/sdb2   lvm2 [3.83 GB]
      Total: 3 [47.54 GB] / in use: 1 [39.88 GB] / in no VG: 2 [7.66 GB] 

### 4.使用pvdisplay查看各物理卷详细参数 ###

    [root@RHEL5 ~]# pvdisplay  #查看各物理卷详细参数
      --- Physical volume ---
      PV Name   /dev/sda2
      VG Name   VolGroup00
      PV Size   39.90 GB / not usable 20.79 MB
    Allocatable   yes (but full)
      PE Size (KByte)   32768
      Total PE  1276
      Free PE   0
      Allocated PE  1276
      PV UUID   aJlaad-NHPT-Cgg3-7yu4-a2RJ-kJJ1-qxSFgD
      --- NEW Physical volume ---
      PV Name   /dev/sdb1
      VG Name  
      PV Size   3.83 GB
    Allocatable   NO
      PE Size (KByte)   0
      Total PE  0
      Free PE   0
      Allocated PE  0
      PV UUID   v2VajD-yS53-SiQA-yTzu-KOiD-RyT3-p0wTvt
      --- NEW Physical volume ---
      PV Name   /dev/sdb2
      VG Name  
      PV Size   3.83 GB
    Allocatable   NO
      PE Size (KByte)   0
      Total PE  0
      Free PE   0
      Allocated PE  0
      PV UUID   iOoK3V-yuww-ZlLF-cRLq-v7hC-CL7c-0bQU1x

当物理卷没有被使用时可删除物理卷 

    [root@RHEL5 /]# pvremove /dev/sdb2   #删除物理卷，
    Labels on physical volume "/dev/sdb2" successfully wiped

## 卷组(VG)创建及管理具体操作步骤 ##

### 1.使用vgcreate将物理卷转化为卷组 ###

     [root@RHEL5 /]# vgcreate vg01 /dev/sdb{1,2}  #将已经是物理卷的/dev/sdb{1,2}转化为卷组名为vg01的卷组
      Volume group "vg01" successfully created

备注：以上未加参数，扩展块(PE)大小默认4M,若通过 vgcreate -s 8M vg01 /dev/sdb｛1，2｝，则指定了扩展块大小为8M  

### 2.使用vgdisplay 查看所有卷组详细信息 ###

    [root@RHEL5 /]# vgdisplay   #看所有卷组详细信息
      --- Volume group ---
      VG Name   vg01
      System ID
      Formatlvm2
      Metadata Areas2
      Metadata Sequence No  1
      VG Access read/write
      VG Status resizable
      MAX LV0
      Cur LV0
      Open LV   0
      Max PV0
      Cur PV2
      Act PV2
      VG Size   7.66 GB
      PE Size   4.00 MB
      Total PE  1960
      Alloc PE / Size   0 / 0  
      Free  PE / Size   1960 / 7.66 GB
      VG UUID   1g8QL0-0cGM-TJji-Q98P-LJ3f-PhDN-2ouSM3
      --- Volume group ---
      VG Name   VolGroup00
      System ID
      Formatlvm2
      Metadata Areas1
      Metadata Sequence No  3
      VG Access read/write
      VG Status resizable
      MAX LV0
      Cur LV2
      Open LV   2
      Max PV0
      Cur PV1
      Act PV1
      VG Size   39.88 GB
      PE Size   32.00 MB
      Total PE  1276
      Alloc PE / Size   1276 / 39.88 GB
      Free  PE / Size   0 / 0  
      VG UUID   AhhisY-vDrc-s4jx-XIsn-QmCp-wMiT-2v01YZ

备注：也可以查看具体某一卷组详细信息

    [root@RHEL5 /]# vgdisplay -v /dev/vg01 

### 3.查看卷组信息 ###

     [root@RHEL5 /]# vgscan#查看卷组信息
      Reading all physical volumes.  This may take a while...
      Found volume group "vg01" using metadata type lvm2
      Found volume group "VolGroup00" using metadata type lvm2

### 4.扩展卷组vgextend,将某个物理卷添加到已存在的卷组中 ###

     [root@RHEL5 /]# pvcreate /dev/sdb3   #创建一个新的物理卷
      Physical volume "/dev/sdb3" successfully created
      [root@RHEL5 /]# vgextend vg01 /dev/sdb3 #将新增的物理卷添加到vg01卷组中
      Volume group "vg01" successfully extended

使用vgremove删除卷组

    [root@RHEL5 /]# vgremove /dev/vg01
     Volume group "vg01" successfully removed

## 逻辑卷(LV)创建及管理具体操作步骤 ##

### 1.创建逻辑卷大小为6G卷名为data，从vg01生成 ###

     [root@RHEL5 /]# lvcreate -L 6G -n data vg01  #从卷组vg01上划分6G的空间为逻辑卷data
      Logical volume "data" created 

## 2.对划分的逻辑卷进行格式化 ##

    [root@RHEL5 /]# mkfs -t ext3 /dev/vg01/data  #以ext3的文件格式化逻辑卷
    mke2fs 1.39 (29-May-2006)
    Filesystem label=
    OS type: Linux
    Block size=4096 (log=2)
    Fragment size=4096 (log=2)
    786432 inodes, 1572864 blocks
    78643 blocks (5.00%) reserved for the super user
    First data block=0
    Maximum filesystem blocks=1610612736
    48 block groups
    32768 blocks per group, 32768 fragments per group
    16384 inodes per group
    Superblock backups stored on blocks:
    32768, 98304, 163840, 229376, 294912, 819200, 884736
    
    Writing inode tables: done   
    Creating journal (32768 blocks): done
    Writing superblocks and filesystem accounting information: done
    
    This filesystem will be automatically checked every 35 mounts or
    180 days, whichever comes first.  Use tune2fs -c or -i to override

备注：也可通过下面命令格式化

`[root@RHEL5 /]# mkfs.ext3 /dev/vg01/data`  

### 3.使用lvsacn查看逻辑卷的信息 ###

    [root@RHEL5 /]# lvscan#查看逻辑卷的信息
    ACTIVE'/dev/vg01/data' [6.00 GB] inherit
    ACTIVE'/dev/VolGroup00/LogVol00' [38.88 GB] inherit
    ACTIVE'/dev/VolGroup00/LogVol01' [1.00 GB] inherit 

### 4.使用lvdisplay查看逻辑卷的具体参数 ###

    [root@RHEL5 /]# lvdisplay   #查看逻辑卷的具体参数
      --- Logical volume ---
      LV Name/dev/vg01/data
      VG Namevg01
      LV UUIDQUmuTB-ofgI-9BbG-1DvN-gWzo-7Vqb-Twmf45
      LV Write Accessread/write
      LV Status  available
      # open 0
      LV Size6.00 GB
      Current LE 1536
    Segments   2
    Allocation inherit
      Read ahead sectors 0
      Block device   253:2
      --- Logical volume ---
      LV Name/dev/VolGroup00/LogVol00
      VG NameVolGroup00
      LV UUIDSrNP2L-bOWm-4clq-22Lh-Fg10-ydeg-7dNpdH
      LV Write Accessread/write
      LV Status  available
      # open 1
      LV Size38.88 GB
      Current LE 1244
    Segments   1
    Allocation inherit
      Read ahead sectors 0
      Block device   253:0
      --- Logical volume ---
      LV Name/dev/VolGroup00/LogVol01
      VG NameVolGroup00
      LV UUIDe7u6Wx-MXhq-Nc2o-lrF9-yea1-Hia5-Cv7d7e
      LV Write Accessread/write
      LV Status  available
      # open 1
      LV Size1.00 GB
      Current LE 32
    Segments   1
    Allocation inherit
      Read ahead sectors 0
      Block device   253:1

备注：也查看某一逻辑卷详细参数

    [root@RHEL5 /]# lvdisplay -v /dev/vg01/data

### 5.使用lvextend增大逻辑卷大小，在线扩容 ###

     [root@RHEL5 /]# lvextend -L +1G /dev/vg01/data   #从卷组vg01上对逻辑卷/dev/vg01/data进行扩容，逻辑卷大小变为7GB
      Extending logical volume data to 7.00 GB
      Logical volume data successfully resized 

### 6.使用resize2fs命令更新系统识别的文件系统大小，立即生效 ###

oot@RHEL5 /]# resize2fs /dev/vg01/data   #使增加的逻辑卷大小立即生效
resize2fs 1.39 (29-May-2006)
Resizing the filesystem on /dev/vg01/data to 1835008 (4k) blocks.
The filesystem on /dev/vg01/data is now 1835008 blocks long.

### 7.使用lvreduce减小逻辑卷大小，必须是离线方式(即先卸载文件系统) ###

    [root@RHEL5 /]# lvreduce -L -1G /dev/vg01/data   #将逻辑卷/dev/vg01/data容量减小1GB
      /dev/cdrom: open failed: Read-only file system
      WARNING: Reducing active logical volume to 6.00 GB
      THIS MAY DESTROY YOUR DATA (filesystem etc.)
    Do you really want to reduce data? [y/n]: y
      Reducing logical volume data to 6.00 GB
      Logical volume data successfully resized
     [root@RHEL5 /]# resize2fs /dev/vg01/data#使减少的逻辑卷大小立即生效
    resize2fs 1.39 (29-May-2006)
    Resizing the filesystem on /dev/vg01/data to 1572864 (4k) blocks.
    resize2fs: Can't read an block bitmap while trying to resize /dev/vg01/data

备注：缩小逻辑卷通常要先卸载文件系统，并且缩小后空间容量必须大于等于文件当前占用的容量，若操作不当，会导致数据丢失，须谨慎。

     [root@RHEL5 /]# lvscan   #查看逻辑卷大小变为6GB
    ACTIVE'/dev/vg01/data' [6.00 GB] inherit
    ACTIVE'/dev/VolGroup00/LogVol00' [38.88 GB] inherit
    ACTIVE'/dev/VolGroup00/LogVol01' [1.00 GB] inherit

## 参考文档 ##

[http://www.server110.com/linux/201403/7158.html](http://www.server110.com/linux/201403/7158.html)