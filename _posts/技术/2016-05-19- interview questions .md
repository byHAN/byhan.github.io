---
layout: post
title: interview questions
category: 技术
tags: openstack
keywords: 
description: 
---


**1.openstack中虚拟机启动的时候有可能会需要向其写入admin密码，指定网卡信息，甚至指定keypair及其写入特定文件。**  
以下选项和上述内容最不相关的是哪项？  
A.userdata/metadata  
B.config-drive  
C.inject_partition  
D.block_device_mapping  

**2.以下哪项技术是部署大规模场景，如1000+节点时，推荐使用的技术？**  
A.nova-cert  
B.nova-cells  
C.nova-console  
D.nova-consoleauth  

**3.只有nova-compute进程僵死的情况下，云平台之前创建的正在运行的虚拟机表现是什么样的？**  
A.虚拟机运行不正常，虚拟机内服务运行也不正常  
B.虚拟机正常运行，但是虚拟机内服务运行不正常  
C.虚拟机正常运行，虚拟机内服务运行也正常  
D.以上说法全不对  

**4.引入nova-conductor的原因？**  

A.提供了一个用于注册S3接口的镜像服务  
B.调度虚拟机在物理宿主机上部署  
C.避免nova-compute直接操纵DB  
D.以上说法全不对  

**5.nova-scheduler中filter的作用？**  
A.选择出满足条件的特定主机  
B.排除掉不满足条件的主机  
C.将主机按照相应算法排序  
D.以上说法全不对  

**6.有关KVM下列说法不正确的是**  
A.可以将Linux内核转化为一个hypervisor  
B.硬件辅助虚拟化（Hardware Assisted Virtualization）  
C.全虚拟化（Full Virtulization）  
D.目前仅可在Inter VT平台运行  

**7.一下哪项不是nova的进程？**  
A.nova-virt  
B.nova-compute  
C.nova-scheduler  
D.nova-conductor  

**8.以下何种操作可以针对nova-compute 资源占用进行资源释放**  
A. nova stop <server>  
B. nova pause <server>  
C. nova suspend <server>  
D. nova shelve-offload <server>  

**9.下面哪一个不是Nova RPC通信模型中的四個重要角色之一**  
A. Exchange  
B. Routing key  
C. Publisher  
D. Composite  

**10.下面那个服务可单独存在于计算节点, 进而搭配 neutron组建提供虚拟机服务?**  
A. nova-api  
B. nova-conductor  
C. nova-compute  
D. nova-schuduler  

**简述题**  

**1.尽可能详细的描述nova启动虚拟机的流程**  

按照此图串起来即可  
![](http://i.imgur.com/sisSSLy.png)


覆盖到nova-conductor,nova-scheduler,nova-compute，libvirt层的内容，即说明对nova有初步了解。  
如果能够理解掌握libvirt乃至qemu-kvm层的东西，则对虚拟化有较深的理解。  

	
**2.nova中虚拟机热迁移和冷迁移的异同点**  
相同点：  
都会使得虚拟机从一个节点迁往另外一个节点  
不同点：  
热迁移是虚拟机保持运行过程中进行的，不会中断业务。  

如果能够答出，热迁移时基于libvirt或者qemu-kvm实现的，冷迁移时其实就是resize,则说明对迁移有一定理解。  


**3.如何控制project内配额的使用？**  
讲出quota即可  
可以进一步提问quota具体是如何做到配额限制的  
只要答出资源预占，成功commit，失败rollback即可。  

**4.虚拟机HA有什么想法。**  
开放性问题。  

主要观察  
1）HA满足条件，比如计算节点掉电，管理网络不通，存储网络不通等情况是如何检测的  
2）HA迁移的手段，如何进行的迁移  
3）HA功能正常的确保手段，如何保证HA过程中的异常情况，如何确保虚拟机不脑裂等  

**5.虚拟化层监控有什么想法？**  
开放性问题，能够较为完整的讲出方案即可。  
如果讲到使用ceilometer采集虚拟化的指标，则可以进一步问这样ceilometer后端使用mongodb的话是否有性能问题？  
如何解决？  A:社区使用gnocchi替代mongdb,业界可能会自己用时间序列数据库替换mongo（自己写driver）  

另外，也有可能直接在计算节点部署agent实现虚拟化直播的采集，等等。  
