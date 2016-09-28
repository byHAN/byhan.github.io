---
layout: post
title: 影子页表原理
category: 技术
tags: openstack
keywords: 
description: 
---

转自网络

Xen/KVM 为客户操作系统提供一个虚拟的物理地址空间，称之为客户物理地址空间。运行在Xen domain中的客户操作系统(Guest Operating System,GOS)认为自己在物理机器上运行并使用该客户物理地址空间中的客户物理地址(Guest Physical Address, GPA)进行寻址。
GOS的页表(Guest Page talbe,GPT)维护的是线性地址(Linear Address，LA)到GPA的映射，但这也页表并不能给硬件使用，因为硬件需要使用从LA到机器物理地址(Host Physical Address,HPA)的映射。
Xen/KVM将GPA到HPA的映射记录在P2M表中(Physical to Machine Mapping, P2M),并为GPT的每个页表维护一个与其对应的硬件使用的页表，其中保存LA->HPA的映射，我们称之为SPT。

映射示意:
GPT:  LA->GPA  (由GOS维护)
SPT:  LA->HPA (由Xen/KVM维护,由GPT派生(同步)得到)
P2M: GPA->HPA

SPT正常工作的一个关键问题是SPT的映射项如何得到，专业上称作如何使GPT与SPT同步：
以Xen为例：
1) 早期的Xen shadow代码(成为shadowI)采用的是Lazy同步的方式，也就是说：
当客户操作系统修改了GPT,如果是权限下降，比如W->R,则当客户操作系统作出修改后会立即使用INVLPG,使TLB无效，这时Xen捕获INVLPG指令并根据其线性地址对SPT/GPT进行同步
当客户操作系统修改了GPT,如果是权限上升，比如R->W,则但客户操作系统访问该页面时会引发#PF,这时Xen捕获#PF,根据线性地址同步SPT/GPT
2) 现在的Xen shadow代码(shadowII)，采用的Eager同步方式：
Xen监控对客户页表的操作(所有客户页所在页面对应的SPT项只读)，当客户操作系统试图修改GPT时会捕获#PF,模拟该指令的执行并且同步SPT/GPT,
相比较shaodwI,减少了一次Trap

Xen实现SPT的关键入口是:
sh_page_fault()
具体情况很复杂，参考我上传的slide(2007格式的ppt,因为论坛附件上限的原因)
大致的流程：

sh_page_fault()
1 检查是否由客户操作系统本身引发的#PF,如果是则向客户操作系统注入#PF, 并返回
2 遍历GPT,将每级GPT记录在某结构中以便之后查询
3 根据之前记录的每级GPT, 同步或者创建相应的SPT
4 判断是否客户操作系统要对GPT进行修改，如果不是就返回
5 模拟客户操作系统对GPT的修改，并且同步相应的SPT
6 返回

另一个入口点是对INVLPG指令的处理，
sh_invlpg()
不再详细说明