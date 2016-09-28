---
layout: post
title: virsh命令（1）monitor,host,nodedev部分
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

#### Domain Monitoring (help keyword 'monitor'): ####  
**domblkerror**:                    Show errors on block devices(显示块设备的错误信息)  
**domblkinfo**:                     domain block device size information(虚拟机块设备的大小信息)  
![](http://i.imgur.com/0P8vlBC.png)  
**domblklist**:                     list all domain blocks(显示所有虚拟机的块设备)  
![](http://i.imgur.com/aNwIjwW.png)  
**domblkstat**:                     get device block stats for a domain（获取虚拟机块设备的状态信息）  
![](http://i.imgur.com/GuNJEdL.png)  
**domcontrol**:                     domain control interface state（虚拟机控制接口状态）  
![](http://i.imgur.com/diBOt7Y.png)  
**domiflist**:                      list all domain virtual interfaces（显示虚拟机的所有虚拟接口）  
![](http://i.imgur.com/Bl0cVJh.png)  
**domif-getlink**:                  get link state of a virtual interface（获取虚拟机接口的链路状态）  
![](http://i.imgur.com/5K0KBuz.png)  
**domifaddr**:                      Get network interfaces' addresses for a running domain  
![](http://i.imgur.com/DzmnqAC.png)  
**domifstat**:                      get network interface stats for a domain（获取虚拟机的虚拟机接口信息）  
![](http://i.imgur.com/wigo6SX.png)  
**dominfo**:                        domain information（虚拟机信息查询）  
![](http://i.imgur.com/n0G7gXd.png)  
**dommemstat**:                     get memory statistics for a domain（获取虚拟机内存状态信息）  
![](http://i.imgur.com/J2m4Tme.png)  
**domstate**:                       domain state（虚拟机状态查询）  
![](http://i.imgur.com/74RX1jj.png)  
**domstats**:                       get statistics about one or multiple domains  
![](http://i.imgur.com/2YlViVz.png)  
**domtime**:                        domain time（获取和设置虚拟机的系统时间）  
![](http://i.imgur.com/ypHIwC1.png)  
**list**:                          list domains（显示所有的虚拟机）

#### Host and Hypervisor (help keyword 'host'): ####

**allocpages**:                     Manipulate pages pool size  
**capabilities**:                   capabilities(主机能力查询)  
**cpu-models**:                     CPU models(获取cpu架构类型)  
![](http://i.imgur.com/sar96Gc.png)  
**domcapabilities**:                domain capabilities(获取qemu的能力)  
**freecell**:                       NUMA free memory(显示NUMA单元格中的可用内存)  
![](http://i.imgur.com/3dNbmFQ.png)  
**freepages**:                      NUMA free pages(显示NUMA节点的可用剩余内存页)  
![](http://i.imgur.com/hGXLmdJ.png)  
**hostname**:                       print the hypervisor hostname(显示主机名)  
**maxvcpus**:                       connection vcpu maximum  
![](http://i.imgur.com/wMIPW68.png)  
**node-memory-tune**:               Get or set node memory parameters(获取或者设置节点内存参数)  
**nodecpumap**:                     node cpu map(显示主机上的cpu位图信息)  
**nodecpustats**:                   Prints cpu stats of the node.(显示主机上cpu状态信息)  
![](http://i.imgur.com/0A6r9Pg.png)  
**nodeinfo**:                       node information(获取主机信息)  
![](http://i.imgur.com/XyCHCqF.png)
**nodememstats**:                   Prints memory stats of the node.(打印主机上内存状态信息)  
![](http://i.imgur.com/CfEiaUj.png)  
**nodesuspend**:                    suspend the host node for a given time duration(在设定的时间段内让主机suspend)  
**sysinfo**:                        print the hypervisor sysinfo(打印主机的hypervisor的信息)  
**uri**:                            print the hypervisor canonical (URI打印hypervisor(基本连接URI)  
![](http://i.imgur.com/THi4bBq.png)  
**version**:                        show version(显示libvirt版本号信息)  


#### Node Device (help keyword 'nodedev'): ####

可以用来将设备分配给虚拟机，[参考这里](https://www.suse.com/documentation/sles11/book_kvm/data/sec_libvirt_config_pci_virsh.html)
**nodedev-create**:             create a device defined by an XML file on the node(根据主机上的XML定义创建一个设备)  
**nodedev-destroy**:            destroy (stop) a device on the node(删除主机上的一个设备)  
**nodedev-detach**:             detach node device from its device driver(通过设备驱动移除主机上的设备)  
**nodedev-dumpxml**:            node device details in XML(显示主机上设备详细的XML信息)  
**nodedev-list**:               enumerate devices on this host(枚举主机上的设备列表)  
**nodedev-reattach**:           reattach node device to its device driver(重新连接主机上的设备到他的驱动)
**nodedev-reset**:              reset node device（重置主机上的设备）
