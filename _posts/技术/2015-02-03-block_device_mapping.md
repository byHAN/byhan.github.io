---
layout: post
title: block_device_mapping
category: nova
tags: nova-storage
keywords: 
description: 
---

在代码中，文档中，接口中经常看见bdm或者block_device_mapping，直译就是块设备映射？  
但它到底是什么东东？有没有一直心存困惑？  
让我们来一探究竟。  

#### 块设备 ####

指对其信息的存取以“块”为单位，如通常的光盘、硬磁盘、软磁盘、磁带等  
本人机器上块设备信息如下：  

![](http://i.imgur.com/VSxjvpI.png)

Nova服务创建的虚拟机，如何为其指派块设备？  
猜的没错，可以通过块设备映射。  
把原有的一些东东（比如cinder的卷，glance的镜像）映射给虚拟机，作为块设备使用。  

### 命令行（CLI） ###

在nova help boot可以看到参数如下：  
![](http://i.imgur.com/ysd6lCF.png)

mapping 的格式是 <dev-name>=<id>:<type>:<size(GB)>:<delete-on-terminate>  
其中：  

#### dev-name ####

A device name where the volume is attached in the system at /dev/dev_name .  
在虚拟机实例中的挂载点  

#### id ####

The ID of the volume to boot from, as shown in the output of nova volume-list.  
卷id，就是nova volume-list  

#### type ####

Either snap, which means that the volume was created from a snapshot,   
or anything other than snap (a blank string is valid). In the example above,  
the volume was not created from a snapshot, so we leave this field blank in our example below.  
类型，一般留空。  

#### size (GB) ####

The size of the volume in gigabytes.  
It is safe to leave this blank and have the Compute Service infer the size.  
卷大小（G），可留空  

#### delete-on-terminate ####

A boolean to indicate whether the volume should be deleted when the instance is terminated.  
True can be specified as True or 1. False can be specified as False or 0.  
实例挂卷之后卷是否删除此设备  


这里我们用镜像启动一个虚拟机，同时挂载磁盘818f15ec-3c5d-4791-b72e-528f82e97584到虚拟机vdc(type和size留空):  

    root@controller1:~# nova boot --image 6eacf3fe-7fe2-4262-b42a-98f624e688c4 --flavor 189aed07-77e3-43ce-a687-56abd5990974 --block-device-mapping vdc=818f15ec-3c5d-4791-b72e-528f82e97584:::0 --nic net-id=47921b66-3e9d-4939-a260-db913d0550f4  test_bdm_win7

![](http://i.imgur.com/Jp9UY1F.png)  

虚拟机创建完成后，查看创建的虚拟机详情  
会发现818f15ec-3c5d-4791-b72e-528f82e97584(名为block_device_mapping)的块被挂载  

![](http://i.imgur.com/wXSGASW.png)

仔细一瞅，nova boot里怎么还有block-device这么个参数，看看说明，好像也是块映射  

![](http://i.imgur.com/7BABTdf.png)

Block Device Mapping  
看到这句注释一阵阵想抽过去，到底和上面--block_device_mapping啥关系啊？  
后面还有一大堆东西。  
先不管，太乱了，不能忍，东西整理如下：  

    id=UUID(image_id,snapshot_id,volume_id)
    souce=源类型(image,snapshot,volume,blank)
    dest=目的类型（volume,local）
    bus=总线类型（uml,lxc,virtio等，常用virtio）
    type=设备类型（disk,cdrom,Floppy,Flash,默认是disk）
    device=设备名称（vda,xda）
    size=块大小
    format=格式（ext4，ISO,swap,ntfs）
    bootindex=定义启动盘，是启动盘的话需要是０
    shutdown=关机对应动作（prserve,remove）

优雅的打开代码，找到nova-client，找到创建虚拟机流程部分（cli在novaclient数据流分析点击这里）  
代码在novaclient/v2/shell.py的_boot方法  
可见从block_device_mapping取出值放入block_device_mapping  
从_parse_block_device_mapping_v2取出值放入block_device_mapping_v2  
两者不能兼容，也就是指定了--block_device_mapping，就不能指定参数--boot_volume、--snapshot、--block_device  

![](http://i.imgur.com/nrBxSGj.png)  

为什么整出两套不兼容的东东？  
因为EC2就是按照--block_device_mapping这种方式搞的，openstack照抄不误。  
后来openstack 兼容多种虚拟化，有的虚拟化不支持直接指定虚拟机的挂载点。  
因此推出了加强版块设备映射（block_device_mapping_v2）  
同时，为了向上兼容block_device_mapping没有干掉，就导致了这种局面。下面让我们看看--block_device参数  
由于源类型(image,snapshot,volume,blank)有这么几种可选，目的类型（volume,local）有这两种可选  
上面参数可以组合出几种形态，依据不同的形态有不同的含义：  

![](http://i.imgur.com/o7SEigZ.png)

### API接口中 ###

创建虚拟机实例的API有block_device_mapping_v2字段。（为什么是v2?我们上面已揭晓）  
在nova/api/openstack/compute/servers.py的create方法中只有这么一段代码  

![](http://i.imgur.com/5vt0Y4N.png)

然后通过handler调用到同目录下的block_device_mapping.py中的servier_create  
(不明白这么调用到的，猜测应该是通过某种统一的方式，调用到各个扩展部分，回头再看吧,有知道的同学也请告知)  
具体原因如下：依次会调用到setup.cfg的nova.api.v3.extensions.server.create的各个插件的server_create 

block_device_mapping.py文件的开头声明了几个常量字符串，对，就有我们此节的主角block_device_mapping_v2  

![](http://i.imgur.com/Uy31qlg.png)

代码逻辑如下图，从入参server_dict中根据上述常量值，取出对应的参数bdm，或者legacy_bdm（legacy遗产的意思）  
然后调用block_device.BlockDeviceDict.from_api将取出的bdm转化为BlockDeviceDict对象  
最后将参数放入create_kwargs['block_device_mapping'],传入底层去创建虚拟了。  
（这里需要注意block_device_mapping_v2和block_device_mapping互斥  
因为block_device_mapping_v2是升级版，为了解决老版问题引入的，详细情况容后表）  

![](http://i.imgur.com/CLKcuy3.png)  

![](http://i.imgur.com/1neefrk.png)  

### 数据表 ###

在nova的数据表中有block_device_mapping  
查询上面一个创建的虚拟机  
可见有个/dev/hda 其source是image　dest是local 对应到上表发现是镜像启动，和我们的启动方式相符。  

![](http://i.imgur.com/1Ih8OX5.png)

### 代码流程 ###

#### 对象转换 ####

一般情况会从数据库block_device_mapping表中查出磁盘信息  
![](http://i.imgur.com/AUjVNwZ.png)  

然后将数据库类型的bdms封装成对于的blockdevice类  
![](http://i.imgur.com/yaKqTtf.png)  
也就是  
![](http://i.imgur.com/ZSfnKDw.png)  
这里我们重点关注下convert_all_volumes这个方法  
就是依次判断每个磁盘的类型，然后返回对应的对象  
![](http://i.imgur.com/WfSNRwX.png)  

这里以convert_volumes为例  
![](http://i.imgur.com/W0JLjH4.png)  
就是说，使用_convert_block_devices方法判断block_device_mapping是不是对于的类型  
出错就跳过  
![](http://i.imgur.com/R7jrjAN.png)  

下面上几个类的类图  
![](http://i.imgur.com/DdgeB0P.png)  
所有的类初始化，都会调用DriveBlockDevice的init方法  
init方法最后会调用一下_transform方法  
![](http://i.imgur.com/3O4h1aR.png)  

总结一下  
![](http://i.imgur.com/jHEzwaN.png)  

**需要注意**
这里不会转换下面这种情况  
也就是说这种磁盘类型会过滤掉  
![](http://i.imgur.com/ZeZQ6fZ.png)

#### 磁盘操作 ####

磁盘操作，以磁盘挂载为例，是调用的各个不同类型磁盘的attach方法，见上面的类图。  
![](http://i.imgur.com/FSDJ5ff.png)

#### 给虚拟机挂载磁盘 ####

实际上也是这段代码，注意参数do_driver_attach为true  
因为创建虚拟机的时候挂在磁盘也会调用这里  
如果do_driver_attach会在虚拟机启动的时候挂载  
![](http://i.imgur.com/xA1TB52.png)  
但是构造bdm对象的时候，指定source_type和destination_type都是volume  
![](http://i.imgur.com/SZt8UX7.png)  

也就是说bdm.attach的时候，bdm是 DriverVolumeBlockDevice  
调用的也是DriverVolumeBlockDevice.attach方法  

1. 从cinder中获取磁盘的连接信息  
  ![](http://i.imgur.com/CXHCrhP.png)  
  通过断电调试发现connection_info如下图（我们使用的ceph）  
  ![](http://i.imgur.com/41FoIFc.png)  
  注意：这里调用的cinder的接口/v2/​{tenant_id}​/types/​{volume_type}​/action  
  接口文档中说没有返回，其实是有返回的  
  ![](http://i.imgur.com/L6UHPbo.png)  

2. 调用hypervisor的attach_volume方法  
   ![](http://i.imgur.com/8XDk8Xb.png)  

3. 如果需要的话，在cinder上建立instance和volume的对应关系  
   cinder数据库中记录  
   ![](http://i.imgur.com/ql5xGw3.png)  

这里深入看下第2步hypervisor的attach_volume方法  
也就是libvirt的driver.py中的attach_volume方法  
实际就是获取到虚拟机的domain然后调用attachDeviceFlags方法  
入参是根据磁盘信息生成的xml，整理后如下  

![](http://i.imgur.com/NeBZcN0.png)  



另外，如果需要的话，会把盘和当前计算节点进行connect  
我们这里用的ceph，什么也不做  
也就是connection_info.get('driver_volume_type')为rbd  
从而rbd=nova.virt.libvirt.volume.LibvirtNetVolumeDriver  
中connect_volume  
![](http://i.imgur.com/IR52lcW.png)  


参考文档：
http://docs.openstack.org/developer/nova/block_device_mapping.html#id3  
https://wiki.openstack.org/wiki/BlockDeviceConfig