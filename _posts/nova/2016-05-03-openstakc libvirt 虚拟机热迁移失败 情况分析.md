---
layout: post
title: openstakc libvirt 虚拟机热迁移失败 情况分析
category: 技术
tags: openstack
keywords: 
description: 
---

# 背景知识 #

新搭建环境，发现执行虚拟机热迁移会报错

![](http://i.imgur.com/8FCxb7A.png)

虚拟机热迁移过程中报上述错误

经查阅代码发现是libvirt底层抛出的错误

![](http://i.imgur.com/kbtPxJd.png)


在计算结点上查询dmidecode -s system-uuid发现几个节点获取到的uuid都是一致的。
然后libvirt认为多个节点是同一个host进而在迁移的过程中抛出异常。

![](http://i.imgur.com/NUaH64U.png)
![](http://i.imgur.com/WlYM14u.png)

# 修改方法 #

修改/etc/libvirt/libvirtd.conf中的host_uuid字段，这里填上一uuid即可

然后，重启libvirt服务

    /etc/init.d/libvirtd restart

验证：

    virsh capabilities | grep uuid