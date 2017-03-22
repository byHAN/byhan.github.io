---
layout: post
title: Sanlock防脑裂场景功能测试用例
category: 技术
tags: 技术
keywords: 
description: 
---


项目/软件	FitOS	版本	V0.0.1  
作者		功能模块名	sanlock（虚拟机防脑裂）  
用例编号	TEST_sanlock_001  
功能特性	防止虚拟机HA后防脑裂  
测试目的	虚拟机HA场景下，正常疏散虚拟机后，源节点和目标节点应该只有一个虚拟机进程与磁盘关联，不会出现两个或者多个虚拟机进程关联同一个磁盘（脑裂）的情况。  
预置条件	HA服务正常启动， nfs服务正常，sanlock服务正常。  
测试数据	N/A 
测试步骤	a)在compute1上从镜像启动虚拟机testha-1；  
b)设置testha-1为高可用虚拟机nova meta testha-1 set ha=True；  
c)触发虚拟机testha-1的HA 
d)待HA完成后，恢复触发HA的条件（如触发HA的条件是compute1掉电，则此时恢复compute1的供电）  
e)观察compute1节点和HA的目标节点上虚拟机存在情况  
（可以使用virsh list 或者ps -ef|grep testha-1）  
期望结果	只在其中一个节点上存在虚拟机进程，不会存在多个虚拟机进程（亦即不会存在脑裂的情况）  
实际结果	  
测试人员		开发人员		测试日期	  


项目/软件	FitOS	版本	V0.0.1  
作者		功能模块名	sanlock（虚拟机防脑裂）  
用例编号	TEST_HA_002  
功能特性	镜像启动的虚拟机测试sanlock功能生效(同一个节点)  
测试目的	验证从镜像启动的虚拟机，sanlock生效，即可以保证同一个虚拟机盘，第二个虚拟机进程与其关联的时候，关联失败。  
预置条件	 nfs服务正常，sanlock服务正常  
测试数据	N/A  
测试步骤	a)在compute1上从镜像创建虚拟机testha-1  
b)获取到虚拟机的配置文件libvirt.xml  

c)注释掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
d)重启libvirt服务  
e)修改上述的虚拟机配置文件libxirt.xml  
f)Virsh create libvirt.xml  确保虚拟机创建成功，如果失败请重复d,e步骤  
（文末附libvirt.xml示例c-f步骤是为了确保libvirt.xml修改正确）  
g)Virsh destroy 销毁掉f步骤的虚拟机 

h)放开掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
i)重启libvirt服务使h步骤的配置修改生效  
j)修改上文中的libvirt.xml中虚拟机的name字段  
k)执行virsh create libvirt.xml  
l)查看执行结果，是不是报错file exist  
（如果是，说明sanlock生效，不允许第二个进程关联磁盘）  
m)Virsh destroy testha-1  销毁掉一开始创建的虚拟机  
n)Virsh create libvirt.xml 查看没有虚拟机进程绑定的情况下，是不是可以正常启动虚拟机  
期望结果	执行结果如上面步骤的每一步预期相符  
实际结果	  
测试人员		开发人员		测试日期	  

项目/软件	FitOS	版本	V0.0.1  
作者		功能模块名	sanlock（虚拟机防脑裂）  
用例编号	TEST_HA_003  
功能特性	卷启动的虚拟机测试sanlock功能生效(同一个节点)  
测试目的	验证从卷启动的虚拟机，sanlock生效，即可以保证同一个虚拟机盘，第二个虚拟机进程与其关联的时候，关联失败。  
预置条件	 nfs服务正常，sanlock服务正常  
测试数据	N/A  
测试步骤	a.在compute1上从卷创建虚拟机testha-1  
b.获取到虚拟机的配置文件libvirt.xml  

c.注释掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
d.重启libvirt服务  
e.修改上述的虚拟机配置文件libxirt.xml  
f.Virsh create libvirt.xml  确保虚拟机创建成功，如果失败请重复d,e步骤  
（文末附libvirt.xml示例c-f步骤是为了确保libvirt.xml修改正确）  
g.Virsh destroy 销毁掉f步骤的虚拟机  

h.放开掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
i.重启libvirt服务使h步骤的配置修改生效  
j.修改上文中的libvirt.xml中虚拟机的name字段  
k.执行virsh create libvirt.xml  
l.查看执行结果，是不是报错file exist  
（如果是，说明sanlock生效，不允许第二个进程关联磁盘）  
m.Virsh destroy testha-1  销毁掉一开始创建的虚拟机  
n.Virsh create libvirt.xml 查看没有虚拟机进程绑定的情况下，是不是可以正常启动虚拟机  
期望结果	执行结果如上面步骤的每一步预期相符  
实际结果	  
测试人员		开发人员		测试日期	  

项目/软件	FitOS	版本	V0.0.1  
作者		功能模块名	sanlock（虚拟机防脑裂）  
用例编号	TEST_HA_004  
功能特性	镜像启动的虚拟机测试sanlock功能生效(不同节点上)  
测试目的	验证从镜像启动的虚拟机，sanlock生效，即可以保证同一个虚拟机盘，第二个虚拟机进程与其关联的时候，关联失败。  
预置条件	 nfs服务正常，sanlock服务正常  
测试数据	N/A  
测试步骤	a.在compute1上从镜像创建虚拟机testha-1  
b.获取到虚拟机的配置文件libvirt.xml  

c.注释掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
d.重启libvirt服务  
e.修改上述的虚拟机配置文件libxirt.xml  
f.Virsh create libvirt.xml  确保虚拟机创建成功，如果失败请重复d,e步骤  
（文末附libvirt.xml示例c-f步骤是为了确保libvirt.xml修改正确）  
g.Virsh destroy 销毁掉f步骤的虚拟机  
 
h.放开掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
i.重启libvirt服务使h步骤的配置修改生效  
j.修改上文中的libvirt.xml中虚拟机的name字段  
k.将libvirt.xml文件拷贝至compute2  
l.登录compute2，执行virsh create libvirt.xml  
m.查看执行结果，是不是报错file exist 
（如果是，说明sanlock生效，不允许第二个进程关联磁盘）  
n.在compute1上Virsh destroy testha-1  销毁掉一开始创建的虚拟机  
o.在compute12上Virsh create libvirt.xml 查看没有虚拟机进程绑定的情况下，是不是可以正常启动虚拟机  
期望结果	执行结果如上面步骤的每一步预期相符  
实际结果	  
测试人员		开发人员		测试日期	  

项目/软件	FitOS	版本	V0.0.1  
作者		功能模块名	sanlock（虚拟机防脑裂）  
用例编号	TEST_HA_005  
功能特性	卷启动的虚拟机测试sanlock功能生效(不同节点上)  
测试目的	验证从卷启动的虚拟机，sanlock生效，即可以保证同一个虚拟机盘，第二个虚拟机进程与其关联的时候，关联失败。  
预置条件	 nfs服务正常，sanlock服务正常  
测试数据	N/A  
测试步骤	a.在compute1上从卷启动虚拟机testha-1  
b.获取到虚拟机的配置文件libvirt.xml  

c.注释掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
d.重启libvirt服务  
e.修改上述的虚拟机配置文件libxirt.xml  
f.Virsh create libvirt.xml  确保虚拟机创建成功，如果失败请重复d,e步骤  
（文末附libvirt.xml示例c-f步骤是为了确保libvirt.xml修改正确）  
g.Virsh destroy 销毁掉f步骤的虚拟机  

h.放开掉/etc/libvirt/qemu.con中lock_manager = "sanlock"  
i.重启libvirt服务使h步骤的配置修改生效  
j.修改上文中的libvirt.xml中虚拟机的name字段  
k.将libvirt.xml文件拷贝至compute2  
l.登录compute2，执行virsh create libvirt.xml  
m.查看执行结果，是不是报错file exist  
（如果是，说明sanlock生效，不允许第二个进程关联磁盘）  
n.在compute1上Virsh destroy testha-1  销毁掉一开始创建的虚拟机  
o.在compute12上Virsh create libvirt.xml 查看没有虚拟机进程绑定的情况下，是不是可以正常启动虚拟机  
期望结果	执行结果如上面步骤的每一步预期相符  
实际结果	  
测试人员		开发人员		测试日期	  


项目/软件	FitOS	版本	V0.0.1  
作者		功能模块名	sanlock（虚拟机防脑裂）  
用例编号	TEST_HA_006  
功能特性	Sanlock在不同平台上的功能验证  
测试目的	在ubuntu和cenos（redhat）两个平台上，分别进行的打包  
需要验证两个平台上的功能情况  
预置条件	 nfs服务正常，sanlock服务正常  
测试数据	N/A  
测试步骤	a.分别搭建ubuntu和centos的系统  
b.在两套系统上分别验证上述用例  
期望结果	功能正常  
实际结果	  
测试人员		开发人员		测试日期	  


附录：  
    <domain type="kvm">
      <name>demo1</name>
      <memory>4194304</memory>
      <vcpu>2</vcpu>
      <os>
    <type>hvm</type>
    <boot dev="hd"/>
      </os>
      <features>
    <acpi/>
    <apic/>
      </features>
      <cputune>
    <shares>2048</shares>
      </cputune>
      <clock offset="utc">
    <timer name="pit" tickpolicy="delay"/>
    <timer name="rtc" tickpolicy="catchup"/>
    <timer name="hpet" present="no"/>
      </clock>
      <cpu mode="host-model" match="exact">
    <topology sockets="2" cores="1" threads="1"/>
      </cpu>
      <devices>
    <disk type="network" device="disk">
      <driver type="raw" cache="writeback" discard="unmap"/>
      <source protocol="rbd" name="vms/11ab78cd-8720-4b78-ab43-972884313c37_disk">
    <host name="10.10.200.205" port="6789"/>
    <host name="10.10.200.207" port="6789"/>
    <host name="10.10.200.208" port="6789"/>
      </source>
      <auth username="cinder">
    <secret type="ceph" uuid="363185a8-e676-42d9-a9ed-bf50cf58be9d"/>
      </auth>
      <target bus="virtio" dev="vda"/>
    </disk>
    </devices>
    </domain>


devices中只保留disk相关内容即可

