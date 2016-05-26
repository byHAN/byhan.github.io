---
layout: post
title: 底层虚拟化版本过低问题导致虚拟机热迁移失败的问题定位（Unkown savevm section type 5 win7）
category: nova
tags: nova-default
keywords: 
description: 
---

# 问题现象及其背景 #

虚拟机热迁移后，所属主机不变。
经过查阅日志，发现底层已经报错了，跟踪代码发现，由于绿色线程导致错误没有抛出到界面。

在qemu对应的虚拟机日志下可见Unkown savevm section type 5等字样

![](http://i.imgur.com/nfTJ7GX.png)

libvirt层迁移是失败的，openstack定时任务感知到迁移后虚拟机的状态和电源状态不一致，后台将虚拟机强制关机。

![](http://i.imgur.com/MkqnzWw.png)

![](http://i.imgur.com/6miQxTb.png)

在虚拟机被关机的间隙中，从portal继续执行热迁移（因为上层没有感知错误，还可以点击）会报错。

关机后，此虚拟机从portal上无法开机。
硬重启虚拟机后，发现虚拟机正常，但是登录到计算节点，会发现源节点和目标节点各有一个对应id的虚拟机。
原因是，上述热迁移过程中，虽然失败，但是没有信息回滚，相关信息在openstack中已经更新到目标节点中。
硬重启时，发现目标节点没有虚拟机，又重新按照既有信息创建了一个，这就导致了两个节点都有虚拟机，从而导致脑裂。

这也就解释了为什么，虚拟机个数审计的时候一直出告警，如下图：

![](http://i.imgur.com/VAnoRut.png)

# 问题调查 #

进过查阅，怀疑是qemu的bug，详情[点击这里](https://lists.nongnu.org/archive/html/qemu-devel/2015-02/msg02325.html)
(需要注意题主描述的有误，本人验证内存为6G同样有问题)

当前qemu版本
![](http://i.imgur.com/En6aznV.png)

同时，直接安装Liberty版本，发现对应的qemu是2.3.0
遂计划直接用Liberty版本验证。

# 问题验证 #

按照官方安装指导书安装环境，虽然这里也遇到很多坑，但为了聚焦这里不展开。

使用raw格式的镜像创建虚拟机

（这里一定要分段，专门吐槽下ceph，太坑了，只支持raw格式的镜像，从远方服务下载镜像耗费了10小时,30G啊，懒得做镜像是另外一个问题了，好吗？）

然后vnc登陆进去发现，直接卧槽蓝屏,错误码是0x0000005d（懒得改回去截图了，凑合理解吧）

经查阅是虚拟化层qemu对win7虚拟机支持有问题，需要使用kvm,即nova-compute.conf文件里需要改为kvm，如图：

![](http://i.imgur.com/c1FfHIp.png)

好，我按照参考文档加载kvm模块。

但是执行到

    modprobe kvm-intel

报错，具体看本人的其他博文

至此，创建出虚拟机，热迁移，发现可以正常迁移。
问题验证结束


正在考虑既有环境，如何升级虚拟化层。
另外，官方的qemu版本已经升级了，从另外一个角度确认此处问题。