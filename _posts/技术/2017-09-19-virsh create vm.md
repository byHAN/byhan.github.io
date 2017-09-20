---
layout: post
title: virsh创建虚拟机简介
category: 技术
tags: 
keywords: 
description: 
---

1.准备磁盘
------

用qemu创建一块硬盘，作为系统盘用来装OS  

这里创建一块名字为this_is_os_disk.img，容量为20G的盘
(这里规划，系统盘统一放到/home/raw下)
    qemu-img create this_is_os_disk.img 20G

![](https://i.imgur.com/Mu3hMU1.png)


2.修改虚拟机xml
----------

保存下面内容到centos7.3.xml中  

1. 如果用新的iso重装，则需要修改CentOS-7-fh-x86_64.iso处为所需的iso路径
2. 修改this_is_os_disk.img处路径为步骤1创建系统盘
3. 修改mac地址的内容fa:16:3f:68:e3:32（mac地址有格式，一般选后半部分一位修改即可）
4. 给虚拟机起个名字，修改<name>部分,这里虚拟机名字为centos

    <domain type='kvm'>
    <memory unit='GiB'>4</memory>
    <vcpu>2</vcpu>
    <os>
    <type arch='x86_64' machine='pc'>hvm</type>
    <boot dev='hd'/>
    <boot dev='cdrom'/>
    </os>
      <features>
       <acpi/>
      </features>
    
    <devices>
      <disk type='file' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source file='/home/image/CentOS-7-fh-x86_64.iso'/>
    <target dev='hda' bus='ide'/>
      </disk>
    
      <disk type='file' device='disk'>
    <driver name='qemu' type='raw'/>
    <source file='/home/raw/this_is_os_disk.img.img'/>
    <target dev='vda' bus='virtio'/>
      </disk>
    
      <graphics type='vnc' port='5900' autoport='yes' listen='0.0.0.0' keymap='en-us'>
    <listen type='address' address='0.0.0.0'/>
      </graphics>
      <video>
    <model type='cirrus'/>
    <alias name='video0'/>
      </video>
      <interface type='bridge'>
      <mac address='fa:16:3f:68:e3:32'/>
      <source bridge='br0'/>
      <model type='virtio'/>
      <alias name='net0'/>
      </interface>
    </devices>
    
    <name>centos</name>
    
    </domain>
    
3.创建虚拟机
-------

    virsh create /home/image/centos7.3.xml 

![](https://i.imgur.com/wa4uQqs.png)


4.查找虚拟机的vnc端口  
---------------

![](https://i.imgur.com/qAUApOv.png)

用vncviewer这个工具（可以去网上下载）  
我们虚拟机创建在192.168.2.207这个服务上  
另外我们刚查到虚拟机的vnc端口为2,qemu会给端口补个590也就是，我们这里链接5902
点击connect也就可以连接到虚拟机的  

![](https://i.imgur.com/MMx6m0n.png)  

注：  
连接不上，可能需要服务器上关闭防火墙  

    iptables -F


5.登录到虚拟机中根据os版本不同修改网卡信息





----------
删除虚拟机
=====

1. 查询虚拟机  

    virsh list

![](https://i.imgur.com/eEzqZWz.png)

2.销毁虚拟机

假设这里需要销毁虚拟机make_iso

    virsh destroy 5
    
    

----------
添加磁盘

1. 准备磁盘  
参照前文准备一块盘  
(这里假设创建的盘为data.img)
2. 准备xml
报错下面内容到文件disk.xml
修改磁盘路径为上面的创建的盘  
修改磁盘名(根据需要修改，不能和已有的重名，这里修改为vdb)

    <disk type='file' device='disk'>
    <driver name='qemu' type='raw'/>
    <source file='/home/raw/data.img'/>
    <target dev='vdb' bus='virtio'/>

3.添加磁盘

这里假设虚拟机的id为5
假设xml放置在/home/disk.xml

    virsh attach-device 5 /home/disk.xml


4.vnc进去使用fdisk -l验证