---
layout: post
title: schedule的filter一览
category: 技术
tags: openstack
keywords: 
description: 
---

#### AggregateCoreFilter	 ####

使用 host-aggregate 元数据关键字 cpu_allocation_ratio 过滤掉超过过度分配（over-commit）比率（虚拟 CPU 和物理 CPU 分配的比率）的主机，它只在为实例指定了主机集合（host aggregate）的情况下有效。
如果没有设置这个值，过滤器会使用 /etc/nova/nova.conf 文件中的 cpu_allocation_ratio 的值。它的默认值是 '16.0'（每个物理 CPU 可以分配 16 个虚拟 CPU）。

#### AggregateDiskFilter ####	

使用 host-aggregate 元数据关键字 disk_allocation_ratio 过滤掉超过过度分配（over-commit）比率（虚拟磁盘和物理磁盘分配的比率）的主机，它只在为实例指定了主机集合（host aggregate）的情况下有效。
如果没有设置这个值，过滤器会使用 /etc/nova/nova.conf 文件中的 disk_allocation_ratio 的值。它的默认值是 '1.0'（每个物理磁盘可以分配一个虚拟磁盘）。

#### AggregateImagePropertiesIsolation ####

只通过那些元数据和实例镜像的元数据匹配的主机集合中的主机，它只在为实例指定了主机集合的情况下有效。如需了解更多相关信息

#### AggregateInstanceExtraSpecsFilter ####	

主机集合中的元数据必须与主机的主机类型元数据相匹配。如需了解更多相关信息

#### AggregateMultiTenancyIsolation ####

指定了 filter_tenant_id 的主机只能包括所指定的租户（项目）中的实例。请注意：租户仍然可以在其它主机上放置实例。

#### AggregateRamFilter ####	

使用 host-aggregate 元数据关键字 ram_allocation_ratio 过滤掉超过过度分配（over-commit）比率（虚拟内存和物理内存分配的比率）的主机，它只在为实例指定了主机集合（host aggregate）的情况下有效。
如果没有设置这个值，过滤器会使用 /etc/nova/nova.conf 文件中的 ram_allocation_ratio 的值。它的默认值是 '1.5'（每个物理内存可以分配 1.5 的虚拟内存）。

#### AllHostsFilter ####

通过所有有效的主机（但不禁用其它过滤器）。

#### AvailabilityZoneFilter ####

过滤器使用实例指定的可用域。

#### ComputeCapabilitiesFilter ####

确定 Compute 元数据被正确读取。':' 前的所有内容被认为是命名空间（namespace）。例如，'quota:cpu_period' 使用 'quota' 作为命名空间，'cpu_period' 是关键字。

#### ComputeFilter ####

通过所有可以正常工作并被启用的主机。

#### CoreFilter ####

使用 /etc/nova/nova.conf 文件中的 cpu_allocation_ratio 过滤掉超过过度分配比率（虚拟 CPU 到物理 CPU 分配比率）的主机。它的默认值是 '16.0'（每个物理 CPU 可以分配 16 个虚拟 CPU）。

#### DifferentHostFilter ####

在一个和指定主机不同的主机上创建实例。使用 nova boot 命令的 --different_host 选项来指定不同的主机。

#### DiskFilter ####

使用 /etc/nova/nova.conf 文件中的 disk_allocation_ratio 过滤掉超过过度分配比率（虚拟磁盘到物理磁盘分配比率）的主机。它的默认值是 '1.0'（每个物理磁盘可以分配 1 个虚拟磁盘）。
ImagePropertiesFilter	只通过匹配实例镜像属性的主机
IsolatedHostsFilter	只通过运行独立镜像（在 /etc/nova/nova.conf 文件中使用 isolated_hosts 和 isolated_images（以逗号分隔的值） 指定）的独立主机。

#### JsonFilter ####	

接受并使用实例的自定义 JSON 过滤器：
有效的操作包括： =, <, >, in, <=, >=, not, or, and
可以接受的变量包括：$free_ram_mb、$free_disk_mb、$total_usable_ram_mb、$vcpus_total、$vcpus_used

这个过滤器通过 nova boot 命令的 '--hint query' 指定。例如：
--hint query='['>=', '$free_disk_mb', 200 * 1024]'

#### MetricFilter ####

过滤掉带有无效指标数据的过滤器。

#### RamFilter ####

使用 /etc/nova/nova.conf 文件中的 ram_allocation_ratio 过滤掉超过过度分配比率（虚拟内存到物理内存的分配比率）的主机。它的默认值是 '1.5'（每个物理内存可以分配 1.5 倍的虚拟内存）。

#### RetryFilter ####

过滤掉调度失败的主机。它只在 scheduler_max_attempts 大于 0 的情况下有效（scheduler_max_attempts=3 是默认设置）。

#### SameHostFilter ####

通过一个或多个指定的主机。使用 nova boot 的 --hint same_host 选项指定主机。

#### ServerGroupAffinityFilter ####

只通过一个特定服务器组中的主机：

设置服务器组的 affinity 策略（nova server-group-create --policy affinity groupName）。
构建带有指定组的实例（nova boot 选项 --hint group=UUID）。

#### ServerGroupAntiAffinityFilter ####
	
只通过还没有包括任何实例的服务器组中的主机：

设置服务器组的 anti-affinity 策略（nova server-group-create --policy anti-affinity groupName）。
构建带有指定组的实例（nova boot 选项 --hint group=UUID）。

#### SimpleCIDRAffinityFilter ####

只通过特定 IP 子网范围内（由实例的 cidr 和 build_new_host_ip hint 指定）的主机。例如：
--hint build_near_host_ip=192.0.2.0 --hint cidr=/24