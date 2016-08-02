---
layout: post
title: 创建虚拟机流程之compute_api之基本参数生成
category: 技术
tags: openstack虚拟机创建流程
keywords: 
description: 
---

_validate_and_build_base_options

- 判断az是否可用
- 判断flavor状态是否可用
- user_data长度判断，解密判断
- 安全组校验
- 网络校验
- kernel和ramdisk处理
- config_drivr校验（待补充_check_config_drive）
- keypair处理
- 设备根目录处理（block_device.prepend_dev）
- 镜像meatadata 处理
- numa相关
- pci映射处理（get_pci_requests_from_flavor）
- sriov_ports的pci处理（create_pci_requests_for_sriov_ports）
- base_options生成
- 从镜像继承部分参数（_inherit_properties_from_image）