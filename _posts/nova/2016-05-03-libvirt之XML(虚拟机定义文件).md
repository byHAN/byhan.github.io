---
layout: post
title: libvirt之XML(虚拟机定义文件)
category: 技术
tags: 虚拟化
keywords: 
description: 
---

libvirt通过XML格式管理虚拟机实例的，具体说是通过XML详细描述了虚拟机的各项配置。
先[点击这里](http://www.lygrzs.com/wp-content/uploads/2015/10/libvirt.txt) 感受下大体上的配置都有那些。
详细配置见[官网](http://libvirt.org/format.html)。

# 路径 #

在nova.conf中instances_path项目定义了你虚拟机在计算节点上储存的位置，这里我配置为/opt/nova/instances

![](http://i.imgur.com/fim7AUk.png)

# 基本信息 #

libvirt中虚拟机称之为domain(另外需要注意的是，xen中domain是宿主机计算节点)

### 通用 ###

    <uuid>8cc9bcbe-8179-4195-88ae-839257fc539f</uuid>  --计算节点上唯一标识
    <name>instance-00000111</name>--虚拟机名字，与openstack创建指定不同，自动生成
    <memory>2097152</memory>--内存大小（2048M）
    <vcpu>2</vcpu>--核数


### metadata ###

    <metadata>
       <nova:instance xmlns:nova="http://openstack.org/xmlns/libvirt/nova/1.0">
          <nova:package version="2015.1.1"/>
          <nova:name>ETCD</nova:name> --这个是我在opesnstack创建的用户名
          <nova:creationTime>2015-10-31 01:40:52</nova:creationTime>
          <nova:flavor name="etc_flavor">   --这个是创建时候指定的flavor模板
              <nova:memory>2048</nova:memory>
              <nova:disk>40</nova:disk>
              <nova:swap>1024</nova:swap>
              <nova:ephemeral>0</nova:ephemeral>
              <nova:vcpus>2</nova:vcpus>
          </nova:flavor>
       <nova:owner>--创建者信息
          <nova:user uuid="48d8cd602577412c978f0184544e9e1c">admin</nova:user>
          <nova:project uuid="d04021d5a4144b4c9f579fdc1d1c2a9a">admin</nova:project>
       </nova:owner>
       </nova:instance>
    </metadata>

### BIOS ###

    <sysinfo type="smbios">   ---System Management BIOS 主板信息
       <system>
          <entry name="manufacturer">OpenStack Foundation</entry>
          <entry name="product">OpenStack Nova</entry>
          <entry name="version">2015.1.1</entry>
          <entry name="serial">00000000-0000-0000-0000-0cc47a159c98</entry>
          <entry name="uuid">8cc9bcbe-8179-4195-88ae-839257fc539f</entry> --复用上面虚拟机ID
       </system>
    </sysinfo>

### 操作系统 ###

    <os>
       <type>hvm</type>　　　--hardware virtual machine 基于硬件的虚拟化
       <boot dev="hd"/>　　　--首先选择硬盘(hard disk)作为启动介质
       <smbios mode="sysinfo"/>　--使用上面的sysinfo定义的内容
    </os>

[todo]这里的boot和openstack中nova boot的下面参数是否有关联，需要确认  
![](http://i.imgur.com/zzcJUbH.png)

### 高级配置 ###

    <features>
       <acpi/>  --Advanced Configuration and Power Management Interface
       <apic/>
    </features>

### CPU糖（cpu tune） ###

    <cputune>
       <shares>2048</shares>
    </cputune>

主要可以设置一些cpu相关的参数做到调配，可以通过这里设置做到虚拟机CPU qos（见本人另外一篇博文）

### 时钟 ###

    <clock offset="utc">
       <timer name="pit" tickpolicy="delay"/>
       <timer name="rtc" tickpolicy="catchup"/>
       <timer name="hpet" present="no"/>
    </clock>

the guest clock is typically initialized from the host clock. Most operating systems expect the hardware clock to be kept in UTC, and this is the default. Windows, however, expects it to be in so  called 'localtime'.timer

 ![](http://i.imgur.com/GFRPH0K.png)

### CPU ###

    <cpu mode="host-model" match="exact">
       <topology sockets="2" cores="1" threads="1"/>
    </cpu>

#### 模式（mode） ####

- host-model（默认选项）按照虚拟机的cpu标签指定的给整，不许瞎整。保证了虚拟机在哪起cpu属性都一样。

- custom按照宿主机的cpu样子整。虚拟机启动的的时候直接抄袭宿主机的cpu信息。
这种情况下match标签就瞎白活了，不起作用。
迁移的时候会保持原来的样子，重启时候会按照新宿主机来。
注意：由于libvirt获取宿主机cpu mode的方式和创建虚拟机的cpu mode时候不与kvm交互,导致虚拟机整出来的情况比较混乱。尽量避免使用这种模式。

- host-passthrough
穿透模式，虚拟机直接看得到硬件，和硬件强相关。
PCI设备的穿透可以参考本人另外博文

#### 匹配(match) ####

虚拟机对所声明的cpu需求的苛求程度。

- minimum  
  The specified CPU model and features describes the minimum requested CPU.
- exact（Since 0.8.5 版本默认）  
  The virtual CPU provided to the guest will exactly match the specification
- strict  
  The guest will not be created unless the host CPU does exactly match the specification.

#### topology ####

具体例子，某个服务器是：2路4核超线程（一般默认为2个线程），那么，通过cat /proc/cpuinfo看到的是2*4*2=16个processor，很多人也习惯成为16核了

- sockets：socket就是主板上插cpu的槽的数目，也即管理员说的”路“
- cores：core就是我们平时说的”核“，即双核，4核等
- threads：thread就是每个core的硬件线程数，即超线程

# 外设 #

### 磁盘 ###

#### type ####

指明当前资源的类型

- "file"
- "block"
- "dir" (since 0.7.5)
- "network" (since 0.8.7)
- "volume" (since 1.0.5)

#### device ####

指明当前资源是虚拟机的什么设备，默认是"disk"

- "floppy"
-  "disk"
- "cdrom"
- "lun"

#### source ####

- type=file  --文件全路径
- type=block --当前磁盘在主机上的设备全路径
- type=dir --目录的全路径
- type=network --protocol表明访问镜像的方式 "nbd", "iscsi", "rbd", "sheepdog" or "gluster"
![](http://i.imgur.com/g2bl0xT.png)

#### target ####

挂载成虚拟机的哪个盘

- bus 访问磁盘设备方式 有ide", "scsi", "virtio", "xen", "usb", "sata", or "sd"等方式。

#### driver ####

可选参数，用来进一步指定磁盘的类型等。

- authtype  有如下几种类型"raw", "bochs", "qcow2", and "qed"

![](http://i.imgur.com/YQlhx8s.png)

# 网卡 #

多种方式，这里只分析bridge模式。
多个宿主机的物理网卡之间虚拟出一条 桥接器  ovs

![](http://i.imgur.com/CX9oMcI.png)

![](http://i.imgur.com/WAotn0C.png)

# 串口 #

![](http://i.imgur.com/RIVuvvB.png)

# 输入设备 #

mouse', 'tablet' or (since 1.2.2) 'keyboard

![](http://i.imgur.com/mhHmZ0y.png)

# 显示 #

启一个vnc server
![](http://i.imgur.com/0IUk00H.png)

# 视频 #

"vga", "cirrus", "vmvga", "xen", "vbox", or "qxl"

![](http://i.imgur.com/xotinLp.png)

# 内存气球 #

通常来说，要改变客户机占用的宿主机内存，是要先关闭客户机，修改启动时的内存配置，然后重启客户机才能实现。而内存的ballooning（气球）技术可以在客户机运行时动态地调整它所占用的宿主机内存资源，而不需要关闭客户机。

![](http://i.imgur.com/MnnFfTs.png)

