---
layout: post
title: 创建虚拟机流程之compute_api之虚拟机实例填充之配额检测
category: 技术
tags: openstack虚拟机创建流程
keywords: 
description: 
---

有人瞎捣蛋，创建一个100cpu 200G的巨无霸虚拟机（姑且不论创建成功与否），作为神一样的系统管理员，别人有这种想法也不能忍。

怎么办？给他设定配额吧。
常见配额如下：

![](http://i.imgur.com/ZL7yDsx.png)

# 默认配额 #

系统->default下，创建一个project时默认援引这里的配额，如下图：

![](http://i.imgur.com/9HwI0KF.png)

# 项目配额 #

![](http://i.imgur.com/ZE5aVPD.png)

# 代码流程 #

#### _check_num_instances_quota ####


- 根据创建虚拟机的数目和使用的flavor
	- 计算出所需vcpu的数目（max_count * instance_type['vcpus']）
	- 计算出所需内存的数量（max_count * (instance_type['memory_mb'] + vram_mb)）
	- 代表视频设备所需内存，是flavor中hw_video:ram_max_mb字段指定的数目
- 使用context初始化Quotas
- 预占资源（quotas.reserve）
- 异常处理

#### quotas.reserve ####

下图可见，这里透传调用的nova/nova/quota文件下的QUOTAS对象的reserve方法

![](http://i.imgur.com/9dTh2Rg.png)

#### QUOTAS ####

进入nova/nova/quota可见QUOTAS是一个QuotaEngine对象
实例完QuotaEngine，会声明一系列的资源，然后注册给QuotaEngine的_resources，资源有如下范围：

- 可预占资源（ReservableResource）:数据库中有真真儿的对象的资源
	- instances
	- cores
	- ram
	- security_groups
	- floating_ips
	- fixed_ips
	- server_groups
- 可计数资源（CountableResource）:数据库中有真真儿的对象的资源，但资源不严格隶属与某个project
	- security_group_rules
	- key_pairs
	- server_group_members
- 独占资源（AbsoluteResource）
	- metadata_items
	- injected_files
	- injected_file_content_bytes
	- injected_file_path_bytes

查看QuotaEngine对象的reserve方法,发现又把方法传递给本类的_driver的reserve，
这里有两种driver:

- DbQuotaDriver:    默认的配置的driver,使用本地数据库事物进行相应控制
- NoopQuotaDriver:与上述driver相比，这个driver只对有必要的配额进行校验,可以实现分层控制

#### DbQuotaDriver.reserve ####

查询配额及预占资源,对可计数的资源（指的是那些使用方法带同步锁的资源），这个方法校验当前已使用量和预占量

资源不在下述资源表中或者对应资源没有同步方法抛出QuotaResourceUnknown
任何一个资源超出限制，抛出OverQuota


- 校验下当前要校验配额的资源们是否可以预占（_valid_method_call_check_resources）
- 获得整个project的未删除的配额（project_quotas = db.quota_get_all_by_project(context, project_id)）
- 获得project针对每个资源的配额限制（_get_quotas）
- 获得用户的针对每个资源的配额限制（_get_quotas附加了用户id）
- 资源预占（db.quota_reserve）<<---点这里,真正实验预占是在这里，其他的都是梳理出准确的配额值

#### _get_quotas ####

- 过滤出请求配额校验的资源
- 请求配额校验的资源必须都满足要求，否则抛出QuotaResourceUnknown
	- 附加了用户id走这里根据资源信息获取用户及其项目配额（get_user_quotas）
	- 否则根据资源信息获取其项目配额（get_project_quotas）


#### get_user_quotas ####

![](http://i.imgur.com/3SLApJJ.png)

- 获得用户在project的配额（quota_get_all_by_project_and_user）
	- 返回hard_limit的值  
      ![](http://i.imgur.com/5zfJy6A.png)
      
- 使用quotas表中内容补齐project_user_quotas不存在的项目
- 配额处理（_process_quotas）

#### get_project_quotas ####

![](http://i.imgur.com/B0oYA7G.png)

#### _process_quotas ####

- 获得对应模板的配额（quota_class_get_all_by_name）
- 获得default模板的配额(get_defaults)
- 按照资源遍历，获取入参quotas中配额，没有则获取模板配额，再没有则获取default模板类的配额
	- user情况下入参为project_user_quotas读取的数据
	- project情况下为quotas表读取的数据
	- 注：猜测模板配额（class）是为了后续做扩展，资源创建时候主动带配额过来


# DB预占 #

资源配额控制默认情况下是使用如下几个表进行控制的：
quotas:hard_limit记录各个project对应的各种资源的配额，更新某个的配额，对应的hard_limit值会刷新

![](http://i.imgur.com/miKzNMk.png)

quota_classes:配额模板，会有一组class_name为default的配额，默认情况下，创建工程会从这里援引配额值
当然，你也可以创建一组新的模板，然后用你自己创建的模板创建工程等。这里只是提供了一中扩展性，
使用场景不多。

![](http://i.imgur.com/6QY3hbd.png)

project_user_quotas:后端支持用户级别在project上的配额限制（猜测）

![](http://i.imgur.com/xNjwDkj.png)

理由如下，创建配额的时候如果带了user_id则配额有可能会创建之上表，负责创建至quotas表

![](http://i.imgur.com/79SbwaV.png)

quota_usages：录了每个工程当前使用的各种资源的数量，in_use值就是保存正在使用的资源数量的，程序中就是通过更新这个in_use值来实现配额管理。

![](http://i.imgur.com/rsxlMRP.png)

reservations：记录的是每次分配的各种资源的变化值，即delta保存的值，它和quota_usages通过usage_id外键关联。这个表中的记录是不更新的，每次分配都会相应的增加记录

![](http://i.imgur.com/AssCfzI.png)

申请资源的时候使用quota_reserve检查下配额，记录到usage.reserved字段预占下
没有超额则使用reservation_commit提交确认，减掉reserved，记录到in_use
超额则调用reservation_rollback回滚掉usage.reserved字段

#### quota_reserve ####

- quota_usages表获取当前资源使用情况，返回项目使用情况（project_usages）及其用户使用情况（user_usages）
- 遍历要预占的资源
	- 判断是否需要创建记录
	- 判断是否需要更新记录
	- 缓存中更新使用信息
- 判断资源是否超限
	- 么有超，创建分配记录至reservations，更新quota_usages表记录
	- 超了，整理信息抛出OverQuota异常

#### reservation_commit ####

    usage.reserved -= reservation.delta
    usage.in_use += reservation.delta

#### reservation_rollback ####

    usage.reserved -= reservation.delta