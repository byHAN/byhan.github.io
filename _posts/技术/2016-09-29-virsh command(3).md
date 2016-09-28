---
layout: post
title: virsh命令（3）之domain部分
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

#### Interface (help keyword 'interface'): ####

iface-begin                    create a snapshot of current interfaces settings, which can be later committed (iface-commit) or restored (iface-rollback)(给当前interface设置创建一个快照文件)
iface-bridge                   create a bridge device and attach an existing network device to it(创建一个网桥设备且为其添加一个网络设备)
iface-commit                   commit changes made since iface-begin and free restore point(提交iface-begin到当前的改变，同时释放掉保存的恢复点)
iface-define                   define an inactive persistent physical host interface or modify an existing persistent one from an XML file(定义一个物理主机接口)
iface-destroy                  destroy a physical host interface (disable it / "if-down")(删除一个物理主机接口)
iface-dumpxml                  interface information in XML(dump出物理主机接口XML配置信息)
iface-edit                     edit XML configuration for a physical host interface(修改物理主机接口XML配置信息)
iface-list                     list physical host interfaces(显示物理主机接口列表)  
![](http://i.imgur.com/Etmi8O3.png)
iface-mac                      convert an interface name to interface MAC address(从物理接口名称获取接口mac地址)
iface-name                     convert an interface MAC address to interface name(从物理接口的mac地址获取接口的名称)
iface-rollback                 rollback to previous saved configuration created via iface-begin(恢复物理接口当前配置到之前通过iface-begin保存的配置信息)
iface-start                    start a physical host interface (enable it / "if-up")(启动物理主机接口（if-up）)
iface-unbridge                 undefine a bridge device after detaching its slave device(拔出添加到网桥上面的设备，同时删除网桥设备)
iface-undefine                 undefine a physical host interface (remove it from configuration)(删除物理主机接口)

#### Network Filter (help keyword 'filter'): ####

nwfilter-define                define or update a network filter from an XML file(更新或者定义一个网络过滤器的XML配置)
nwfilter-dumpxml               network filter information in XML(显示一个网络过滤器的XML配置)
nwfilter-edit                  edit XML configuration for a network filter(编辑一个网络过滤器的XML配置)
nwfilter-list                  list network filters(显示网络过滤器列表)
nwfilter-undefine              undefine a network filter(删除一个网络过滤器的定义)

#### Networking (help keyword 'network'): ####

net-autostart                  autostart a network(配置在系统启动时自动启动网络)
net-create                     create a network from an XML file(根据XML创建并启动一个网络)
net-define                     define an inactive persistent virtual network or modify an existing persistent one from an XML file(根据XML创建一个网络)
net-destroy                    destroy (stop) a network(关闭一个active状态的网络)
net-dhcp-leases                print lease info for a given network(显示网络的lease信息)
net-dumpxml                    network information in XML(显示一个网络的XML定义)
net-edit                       edit XML configuration for a network(编辑一个网络的XML定义)
net-event                      Network Events(列出事件类型，或者等待一个网络事件触发)
net-info                       network information(显示网络信息)
net-list                       list networks(显示网络列表)
net-name                       convert a network UUID to network name(根据网络UUID获取网络名字)
net-start                      start a (previously defined) inactive network(启动一个已创建的网络)
net-undefine                   undefine a persistent network(删除已网络定义)
net-update                     update parts of an existing network's configuration(更新现有的网络配置)
net-uuid                       convert a network name to network UUID(根据网络名称获取网络的UUID)