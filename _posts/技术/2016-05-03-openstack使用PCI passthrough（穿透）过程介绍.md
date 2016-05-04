---
layout: post
title: openstack使用PCI passthrough（穿透）过程介绍
category: 技术
tags: openstack
keywords: 
description: 
---

有些PCI设备不易于虚拟化或者虚拟化的效果不好，如何让虚拟机能够高效的使用这些设备是一个亟待解决的问题。

借助于Intel® (VT-d) 或 AMD (IOMMU) 的硬件支持，可用改进虚拟机对此类设备的使用效果。

基本思想是穿透虚拟化层，由虚拟机器直达PCI设备，看起来像PCI设备直接分配给虚拟机一样。

# 流程 #

### 准备 ###

确保计算节点支持PCI穿透
http://www.linux-kvm.org/page/How_to_assign_devices_with_VT-d_in_KVM

### 配置nova ###

#### (1)计算节点 ####

配置穿透白名单
注：lspci然后lspci -n |grep 可以查询到相应id

pci_passthrough_whitelist=[{ "vendor_id ":"8086","product_id":"1520"}]
定义了满足上述条件（供应商ID为8086和产品识别码1520）的PCI设备可备虚拟机穿透使用。

#### (2)控制节点 ####

指定待穿透设备的别名

pci_alias={"vendor_id":"8086", "product_id":"1520", "name":"a1"}
指定"vendor_id":"8086","product_id":"1520"这个设备的别名为"a1"

#### (3)调度节点 ####

启用按照PCI过滤配置
scheduler_driver=nova.scheduler.filter_scheduler.FilterScheduler
scheduler_available_filters=nova.scheduler.filters.all_filters  scheduler_available_filters=nova.scheduler.filters.pci_passthrough_filter.PciPassthroughFilter  scheduler_default_filters=RamFilter,ComputeFilter,AvailabilityZoneFilter,ComputeCapabilitiesFilter,ImagePropertesFilter,PciPassthroughFilter

#### （4）创建flavor ####

nova flavor-key m1.large set "pci_passthrough:alias"="a1:2"
设置此flavor需要两个a1别名指定的pci设备

#### （5）创建虚拟机 ####

nova boot --image new1 --key_name test --flavor m1.large 123

#### （6）验证生效 ####

登陆到虚拟机，lspci命令查看
http://www.xlcloud.org/bin/view/Blog_XLcloud/Feed_XLcloudBlog1379539601RunninganOpenGLapplicationonaGPU-acceleratedNovaInstancePart1

# 问题 #

虚拟机热迁移

PCI设备热插拔

# 参考文献 #

http://libvirt.org/guide/html/Application_Development_Guide-Device_Config-PCI_Pass.html

http://www.ibm.com/developerworks/cn/linux/l-pci-passthrough/