---
layout: post
title: 虚拟机迁移之热迁移(live_migrate)
category: nova
tags: nova-default
keywords: 
description: 
---

# 背景知识 #

迁移主要是将虚拟机从一个地方挪动到另外一个地方，一般情况下为从一个物理机移动至另外一个物理机。
这样，为管理员迁空物理机提供了途径，进而管理员可以对物理机维护。再就是，迁移可以作为DRS的基础，实现资源的负载均衡等。

迁移分为热迁移和冷迁移。
冷迁移在openstack中同resize类似，本文不做分析。
热迁移，迁移过程中虚拟机保持不当机，保证了服务的可持续性，提高了云使用者的体验。

# 核心总结 #

openstack的热迁移是调用了libvirt接口virDomainMigrateToURI2 或者virDomainMigrateToURI
进行的，openstack本质上只做了两件事。

1. 迁移前的源主机和目标机之间的协商确认。看看，两者之间能否迁移。
2. 迁移的后续处理。成功了，需要在源主机上进行清理操作；失败了，需要到目标主机上进行清理操作，回滚。

# 源码分析 #

## nova-api ##

热迁移的入口在nova/nova/api/openstack/compute/migrate_server.py中
主要是对请求相关参数进行读取，获取到当前实例信息后，调用compute/api对应服务。
这里注意下，热迁移一般情况下是指定目标主机的，当然没有指定主机后续代码会遴选一个。
磁盘超量参数目前没有实现（具体见迁移任务实施部分）

![](http://i.imgur.com/LOUoEOA.png)

## compute/api ##

对实例状态等进行一系列校验
- 虚拟机必须处于非锁定状态
- 环境启用cell场景下，校验cell是否存在是否只读
- 虚拟机实例状态必须是运行态或者暂停态


更新虚拟机实例的任务状态为迁移中
在instance_action中记录一条action为live-migration信息
调用conductor的api进行后续的业务处理

![](http://i.imgur.com/UDK1hHY.png)

## nova-conductor ##

将目标主机名称作为调度条件传入，rpc方式调用conductor服务

![](http://i.imgur.com/aabFizx.png)

在conducotr的rpc后端(nova/nova/conductor/manager.py)根据前面的入参走到热迁移分支

![](http://i.imgur.com/ZWOBEQE.png)

_live_migrate方法的主要功能是构造出migration对象，migration对象入库,数据库字段如下

![](http://i.imgur.com/RaOEU2y.png)

然后基于migrate对象和已有的数据，通过_build_live_migrate_task
构造了一个迁移任务，并且运行迁移任务，见下图

![](http://i.imgur.com/e7BWTbs.png)

### 迁移任务执行 ###

在conductor/tasks/live_migrate.py中_execute方法
校验实例状态，确保为running获取paused
校验迁移的源主机是否启动（详细见后）
校验目标主机是否适合进行迁移（没有目标主机，根据相关条件选出一个，详细见后）
向源主机发送rpc请求，通知源主机开始热迁移。

![](http://i.imgur.com/4hBGq9H.png)

#### 验证源主机是否启动（_check_host_is_up） ####

![](http://i.imgur.com/dzfKwSE.png)

主要根据配置的driver不同，servicegroup_api.service_is_up会采用不同的方式判断当前主机是否启动

![](http://i.imgur.com/MsHp2ru.png)

这里的dirver有三种

- db(如果不指定，使用这种方式)
	- 从service表中读取出最后更新状态，与当前时间比较查看是否超出阈值。
- mc（使用memcache）
	- 我们在nova.conf中指定的是这种方式
	- ![](http://i.imgur.com/JfjerFj.png)
	- mc类型的driver对应的is_up方法如下  
	- ![](http://i.imgur.com/nycbj7P.png)
-zk（zookeeper）

#### _find_destination ####

我们看下没有有目标主机，生成目标主机策略（conductor/tasks/live_migrate.py方法_find_destination）
不停的调度获取主机，然后判断主机是否可以进行热迁移，直到找到一个满足要求的主机。
判断是否迁移主要从两个方面
1.hpyervisor类型是否满足要求
2.目标主机能否适合这次热迁移（这里和有主机的情况下一致，详细分析见后）

![](http://i.imgur.com/0Hqo6NF.png)


#### 有目标主机的情况下，判断目标主机是否适合进行热迁移 ####

主要有以下五个判断项目
这里只重点分析：
内存判断（_check_destination_has_enough_memory）
是否支持热迁移判断（_call_livem_checks_on_host）

![](http://i.imgur.com/RxXqw1B.png)

- 内存判断（_check_destination_has_enough_memory）  
  数据库的compute_nodes表中记录了计算节点的内存大小。  
  ![](http://i.imgur.com/KZMl2ru.png)  
  ![](http://i.imgur.com/t1g28Ug.png)

- 是否可以热迁移（_call_livem_checks_on_host）  
   ![](http://i.imgur.com/N5On9tn.png)
- 目标节点上的底层判断  
  ![](http://i.imgur.com/Adlu541.png)  
  ![](http://i.imgur.com/r1x4oXK.png)

- 反向rpc调用源主机。  
  ![](http://i.imgur.com/nQXp2uz.png)  
  ![](http://i.imgur.com/mOTm9Dl.png)  
  ![](http://i.imgur.com/7KuKmZ3.png)

## nova-compute ##

计算节点接收到rpc请求后孵化一个线程，调用_do_live_migration执行迁移过程

![](http://i.imgur.com/r2R3U8s.png)

线程内主要干了两件事情：
1，与目标主机进行协商准备进行迁移，到目标节点创建一系列必要资源等（后面详细分析）
2，调用底层接口进行迁移，同时处理后事：迁移成功，则清理源节点，失败则清理目标节点（后面详细分析）

![](http://i.imgur.com/oXKZnKJ.png)

### 与目标主机进行协商预迁移（pre_live_migration） ###

1.调用driver层预迁移 （后面详细分析）
2.绑定网络至目标主机（什么没做）
3.目标主机增加虚拟机的过滤规则（什么没做）

![](http://i.imgur.com/F2ngmxA.png)

#### dirver层的预迁移（nova/virt/libvirt/driver.py） ####

实例路径不共享，则需要在目标主机上创建相应的目录
块设备不共享（虚拟机实例路径共享）需要从虚拟机指定路径下载相关镜像文件到目标主机

![](http://i.imgur.com/pxcz8qS.png)

不进行块迁移（如果进行的话上面会进行操作）且虚拟机目录不共享（共享的话不用处理）
需要在虚拟机实例目录下创建console文件，并尝试获取kernel和ramdisk

![](http://i.imgur.com/HJbAlqQ.png)

由于后端使用的是rbd，这里什么也没有执行
'rbd=nova.virt.libvirt.volume.net.LibvirtNetVolumeDriver'

![](http://i.imgur.com/aYHWl26.png)

有附加盘，情况下不允许进行块迁移
todo１．block_device_mapping转换（需要深入理解）
todo 2 .在理解块迁移实质的基础上，理解这里做此规定的原因

![](http://i.imgur.com/XACwOtD.png)

下面这段代码比较重要，会在创建虚拟机时候复用
主要是在目标主机上为虚拟机准备网络，当虚拟机迁移过来后直接和对应的网络进行对接即可。

![](http://i.imgur.com/vDhhKHx.png)

整理必要的信息进行返回

![](http://i.imgur.com/robKaDr.png)

#### 绑定网络至目标主机（我们用的是neutronv2/api这里什么也没干） ####\

![](http://i.imgur.com/1adJaXs.png)


### 调用底层进行迁移（driver中 _live_migration） ###

在driver层面是通过_live_migration_operation进行迁移操作，然后通过_live_migration_monitor检测迁移进度

#### 迁移操作（_live_migration_operation） ####

迁移操作是通过调用libvirt对应接口（virDomainMigrateToURI2 或者virDomainMigrateToURI）实现的，接口入参中很重要的一项就是迁移配置项。
配置项约束了迁移过程中的动作，作为入参传给上述的libvirt迁移函数，间接决定了迁移过程中的相应动作，相关参数可以如下：

![](http://i.imgur.com/vBacHCj.png)

![](http://i.imgur.com/ZN2ijhN.png)

那么下面的代码就不难理解了，读取出迁移配置项目
配置项之间逻辑或出一个值logical_sum = six.moves.reduce(lambda x, y: x | y, flagvals)分析见后
没配置VIR_DOMAIN_XML_MIGRATABLE，或者没有VNC和附加盘走接口1，其他情况走接口2

![](http://i.imgur.com/4XS89ox.png)
![](http://i.imgur.com/1bQggMi.png)

#### 迁移检测（_live_migration_monitor） ####

由于迁移是一个比较耗时的操作，所以在上述的迁移动作执行后，由这里进行迁移状态的侦测。
通过host.DomainJobInfo.for_domain(dom)获取到当前操作的信息，根据info.type获取迁移的状态。

如果状态完成，则调用外面传入进来的_post_live_migration方法处理（代码太长逐段分析）

1. 调用driver的post_live_migration,其实是想在libvirt层面上去断开虚拟机与卷的链接（由于使用的rbd实际pass）  
  ![](http://i.imgur.com/2VqzEse.png)
2. 通知cinder去断开链接  
   ![](http://i.imgur.com/n4voo1M.png)
3. 源主机网络处理
	1. 清理防火墙规则（driver.unfilter_instance）
	2. 对应网卡信息从源主机拔出  
   ![](http://i.imgur.com/lCtShIB.png)
    
4. 目标主机上执行后续操作  
   ![](http://i.imgur.com/nhnHkWf.png)  
   ![](http://i.imgur.com/WSSZAT5.png)
5.进行一系列的清理操作及其节点信息的更新
如果状态失败，则调用传入进来的_rollback_live_migration方法进行回滚
1.目标节点卷断连
2.目标节点清理实例，包括之前创建的网络信息等。nplug vif等



附加：
logical_sum = six.moves.reduce(lambda x, y: x | y, flagvals)

1.lambda算式，类似于一个小函数
2.x|y  x和y的逻辑或
3. six.moves.reduce 从flagvals中取出两个值调用lambda算式，得出结果作为x，再从flagvals中取出一个值
(lambda x, y: x+y, [1, 2, 3, 4, 5])类似于 ((((1+2)+3)+4)+5)
dfaf
综上所述，这句代码是求算出flagvals所有值的一个逻辑或。
   


