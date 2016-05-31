---
layout: post
title:something　about ceilometer disk sample 
category: 技术
tags: ceilometer
keywords: 
description: 
---

后端存储使用ceph

libvirt查询磁盘相关的信息时会有如下不同

块信息（ceph没有提供对接的端）
![](http://i.imgur.com/IPCst9R.png)

块状态可查询的出(至于为什么可以读取的到，猜测是查询的os,后续阅读libvirt代码解答TODO)
![](http://i.imgur.com/9lz83e1.png)

因此，设计domblkstat的指标可以查询的到，如disk.read.requests等

涉及domblkinfo的指标查询不到，如disk.capacity等

具体的指标可以到ceilometer的setup.cfg中查看
形如：

    disk.read.requests = ceilometer.compute.pollsters.disk:ReadRequestsPollster
    disk.write.requests = ceilometer.compute.pollsters.disk:WriteRequestsPollster
    disk.read.bytes = ceilometer.compute.pollsters.disk:ReadBytesPollster
    disk.write.bytes = ceilometer.compute.pollsters.disk:WriteBytesPollster
    disk.read.requests.rate = ceilometer.compute.pollsters.disk:ReadRequestsRatePollster
    disk.write.requests.rate = ceilometer.compute.pollsters.disk:WriteRequestsRatePollster
    disk.read.bytes.rate = ceilometer.compute.pollsters.disk:ReadBytesRatePollster
    disk.write.bytes.rate = ceilometer.compute.pollsters.disk:WriteBytesRatePollster
    disk.device.read.requests = ceilometer.compute.pollsters.disk:PerDeviceReadRequestsPollster
    disk.device.write.requests = ceilometer.compute.pollsters.disk:PerDeviceWriteRequestsPollster
    disk.device.read.bytes = ceilometer.compute.pollsters.disk:PerDeviceReadBytesPollster
    disk.device.write.bytes = ceilometer.compute.pollsters.disk:PerDeviceWriteBytesPollster
    disk.device.read.requests.rate = ceilometer.compute.pollsters.disk:PerDeviceReadRequestsRatePollster
    disk.device.write.requests.rate = ceilometer.compute.pollsters.disk:PerDeviceWriteRequestsRatePollster
    disk.device.read.bytes.rate = ceilometer.compute.pollsters.disk:PerDeviceReadBytesRatePollster
    disk.device.write.bytes.rate = ceilometer.compute.pollsters.disk:PerDeviceWriteBytesRatePollster
    disk.latency = ceilometer.compute.pollsters.disk:DiskLatencyPollster
    disk.device.latency = ceilometer.compute.pollsters.disk:PerDeviceDiskLatencyPollster
    disk.iops = ceilometer.compute.pollsters.disk:DiskIOPSPollster
    disk.device.iops = ceilometer.compute.pollsters.disk:PerDeviceDiskIOPSPollster
    cpu = ceilometer.compute.pollsters.cpu:CPUPollster
    cpu_util = ceilometer.compute.pollsters.cpu:CPUUtilPollster
    network.incoming.bytes = ceilometer.compute.pollsters.net:IncomingBytesPollster
    network.incoming.packets = ceilometer.compute.pollsters.net:IncomingPacketsPollster
    network.outgoing.bytes = ceilometer.compute.pollsters.net:OutgoingBytesPollster
    network.outgoing.packets = ceilometer.compute.pollsters.net:OutgoingPacketsPollster
    network.incoming.bytes.rate = ceilometer.compute.pollsters.net:IncomingBytesRatePollster
    network.outgoing.bytes.rate = ceilometer.compute.pollsters.net:OutgoingBytesRatePollster
    instance = ceilometer.compute.pollsters.instance:InstancePollster
    instance_flavor = ceilometer.compute.pollsters.instance:InstanceFlavorPollster
    memory.usage = ceilometer.compute.pollsters.memory:MemoryUsagePollster
    memory.resident = ceilometer.compute.pollsters.memory:MemoryResidentPollster
    disk.capacity = ceilometer.compute.pollsters.disk:CapacityPollster
    disk.allocation = ceilometer.compute.pollsters.disk:AllocationPollster
    disk.usage = ceilometer.compute.pollsters.disk:PhysicalPollster
    disk.device.capacity = ceilometer.compute.pollsters.disk:PerDeviceCapacityPollster
    disk.device.allocation = ceilometer.compute.pollsters.disk:PerDeviceAllocationPollster
    disk.device.usage = ceilometer.compute.pollsters.disk:PerDevicePhysicalPollster

