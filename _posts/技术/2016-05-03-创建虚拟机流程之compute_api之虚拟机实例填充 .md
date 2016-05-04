---
layout: post
title: 创建虚拟机流程之compute_api之虚拟机实例填充
category: 技术
tags: openstack虚拟机创建流程
keywords: 
description: 
---

## _provision_instances ##

- 数目配额校验（_check_num_instances_quota）
- 使用context初始化实例（objects.Instance(context=context)）
- 使用基本参数更新实例（instance.update(base_options)）
- 数据库中创建实例对象（create_db_entry_for_new_instance）
- 组配额校验（check_server_group_quota）
- 发送BUILDING至消息队列更新虚拟机状态
- 异常
	- 尝试清理DB中实例（instance.destroy()）
	- 配额回滚（quotas.rollback()）
- 配额提交（quotas.commit()）