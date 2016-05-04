---
layout: post
title: 创建虚拟机流程之compute_api
category: 技术
tags: openstack虚拟机创建流程
keywords: 
description: 
---

## create ##

- 校验创建的policy(_check_create_policies)
- 校验分配ip指定端口（_check_multiple_instances_and_specified_ip，_check_multiple_instances_neutron_ports）
- 调用实现方法（_create_instance）


#### _create_instance ####

- 相关参数处理
- 没有指定flavor则使用默认配置default_flavor的favor
- 处理镜像相关参数
- 判断auto_disk_config
- 处理az，获得forced_host, forced_node
- 校验生成基本参数（_validate_and_build_base_options）
- 磁盘映射（_check_and_transform_bdm）
- 创建判断（_checks_for_create_and_rebuild）
- 分组情况处理(_get_requested_instance_group)
- 虚拟机实例填充（_provision_instances）
- 为scedule生成过滤条件（_build_filter_properties）
- 调用conductor的rpc接口