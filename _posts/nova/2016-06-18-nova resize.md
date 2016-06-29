---
layout: post
title: nova resize
category: nova
tags: nova-default
keywords: 
description: 
---

## 一句话总结 ##

## 流程概览 ##

![](http://i.imgur.com/YaeJmkU.png)  
由上面的流程图可见，resize和其他很多业务进行了代码服用  
如，和热迁移、冷迁移等  

需要注意，conductor-api进行RPC调用conductor的时候调用的是**migrate_server**  

真正resize的时候，compute-source与compute-dest是同一台机器  
由于这里和冷迁移代码复用，会有区分  

## 源码分析 ##

### compute/api ###
前面的略过，从compute/api开始  

resize方法中，根据指定的flavor进行一系列的判断  
比如原来的flavor中指定为非零，新flavor中不允许再指定为零。  
![](http://i.imgur.com/bPUgMrj.png)  


根据新flavor与原来flavor的差异，quotas预占资源  
![](http://i.imgur.com/VGDjqU3.png)

默认情况下，resize操作，虚拟机会在跑到其他计算节点上去  
![](http://i.imgur.com/rlUk8Wr.png)

### conductor ###

![](http://i.imgur.com/4i7g7b2.png)  


接下来就是调用scheduler选出主机，然后向其发起prep_resize  
![](http://i.imgur.com/1poN6ae.png)  

### comute-dest ###

目标节点的prep_resize调用私有方法_prep_resize  
先进行一系列校验，比如虚拟机中是否有源节点信息，比如是否满足allow_resize_to_same_host设置  
然后，调用源节点resize_instance方法  
![](http://i.imgur.com/dBTMMMM.png)  

### comute-source ###

更新migration状态  
更新instance的任务状态  
向消息队列发送resize.start信号  
![](http://i.imgur.com/rJB9SrV.png)  

获取相应的信息后，调用虚拟化层的migrate_disk_and_power_off方法  
（这一块内容后面详述）

![](http://i.imgur.com/N8q6TGU.png)  

更新状态  
![](http://i.imgur.com/NW3MYbj.png)  

向目标节点发送消息触发finish_resize  
![](http://i.imgur.com/2MD9Qxq.png)


terminate_connection是用来通知后端存储删除映射关系的  
在使用ceph的情况下，ceph不存在映射关系，所以直接pass，如果是其他存储则需要实现

向消息队列发送resize.end消息  

#### 虚拟化层（migrate_disk_and_power_off） ####

进行相关的校验  
虚拟机在源节点关机  
将相关配置移动到_resize目录  



### comute-dest ### 

目标节点先执行_finish_resize子方法，成功后，quotas才最终提交  
![](http://i.imgur.com/9eZWQ4s.png)  

_finish_resize干了什么呢？  

首先更新相关flavor（时刻准备出错回滚）  
![](http://i.imgur.com/BrS6V8w.png)  

再把虚拟机的网卡信息更新到新节点上  
![](http://i.imgur.com/iA2xByx.png)  

更新虚拟机任务状态为resize_migrated，发送finish_resize.start到消息队列  
![](http://i.imgur.com/PgxR4Ab.png)  

调用虚拟化层，执行finish_migration（后面详细述）  
![](http://i.imgur.com/94ICARX.png)

更新migration状态为finished  
更新虚拟机状态为resized  
更新虚拟机launched_at为当前utc时间  
更新虚拟机任务状态为resize_finish  
向消息队列发送resize_finish消息  
![](http://i.imgur.com/7Henxdd.png)  

#### 虚拟化层（finish_migration） ####

目标节点的虚拟化层接收到执行finish_migration的命令后执行这里  
这一步是resize的最后一步，目的是在目标节点上构建虚拟机  

根据需要判断是否需要扩展磁盘  
![](http://i.imgur.com/B8ImGvu.png)
处理镜像  
生成虚拟机xml  
创建虚拟机及网口  
![](http://i.imgur.com/o272xlv.png)


