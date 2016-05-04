---
layout: post
title: vm resize/revert/cold migration bug
category: 技术
tags: openstack
keywords: 
description: 
---

# 背景 #

在Kilo版本发现此问题，经验证Liberty此问题依然存在
此外需要注意，是从镜像启动的虚拟机。

目标已提交bug:[https://bugs.launchpad.net/nova/+bug/1549194](https://bugs.launchpad.net/nova/+bug/1549194)

# 问题描述 #

对一台虚拟机testVM，执行resize操作  
（resize的实质用一个新的flavor去构建一个新的虚拟机，默认情况下，新创建的虚拟会在另外一台计算节点上。另外，openstack要求resize的flavor只能规格更大）

 revert刚刚的resize操作
 默认情况下，resize执行完了，需要用户去确认操作。确认完后会清理掉一些缓存的东西，类似于数据库的commit操作。
 当然，也可以取消刚刚的resize操作，也就是revert resize,虚拟机的相关东西会回滚至执行resize之前。

 对虚拟机testVM执行冷迁移
 对openstack熟悉的同学会知道，冷迁移和resize实际走的是一套流程。
 执行这个操作会失败，此后虚拟机会进入error状态。*

# 问题定位 #

conductor接受到请求后会选出一个计算节点作为目标节点，默认配置下会是另外一个计算节点。
然后向目标节点发送请求，让其开始准备迁移。
目标节点接受到请求后，准备必须的资源，然后向源节点（虚拟机原来在的节点）发送请求，让其开始迁移。

源节点接受到请求后，执行迁移操作，最后向目标节点发送请求finish_resize，让其结束此次迁移
目标节点接受到请求后，会新起个虚拟机进程，网络存储什么的该绑上的绑上。这里需要注意标红的地方。

![](http://i.imgur.com/P8JK1Up.png)

_create_image方法中，有这么一段，翻译一下就是，当传入的size大于克隆盘容量大小时，扩容。
具体点说就是，resize的时候更大的那个flavor的系统盘大小就是传入的size来源。

![](http://i.imgur.com/pr8WBGS.png)

刚说过，执行冷迁移会是一套代码。
冷迁移还是会走到刚刚的_create_image，不过这里我们关注verify_base_size，它会判断大小。
根据刚刚的操作步骤，执行冷迁移前，我们已经revert了resize,也就是说当前虚拟机的规格是小的。
但是，上面变大的镜像拷贝没有变小，导致这里校验出错。

![](http://i.imgur.com/MEEpHmu.png)

# 问题修改 #

revert resize后缩容
就是执行revert resize后，将克隆盘减小回原来的大小
目前已验证，ceph可以变小，变小后，虚拟机正常
优点：符合整体的逻辑，可以解决bug
缺点：1.与openstacck只增大不缩小的思路悖逆。 2.rbd是瘦分配，支持缩小，其他不支持类型如何统一。
confirm resize的时候再扩容
用户执行确认执行resize后进行容量的调整。
优点:可以解决问题，不牵扯缩容的问题缺点：1.resize整体的逻辑变化较大

目前这问题在社区滞留。