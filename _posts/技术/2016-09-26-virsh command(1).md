---
layout: post
title: virsh命令（1）之domain部分
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

### 简介 ###

本系列整理virsh命令，这篇是第一篇，整理virsh help domain相关内容  

    virsh # help domain
    Domain Management (help keyword 'domain'):
    attach-device                  attach device from an XML file(通过XML配置给虚拟机添加设备)
    attach-disk                    attach disk device(添加磁盘设备）
    attach-interface               attach network interface（添加网络设备）
    autostart                      autostart a domain（给虚拟机添加自动启动配置，当libvirtd服务被拉起的时候，自动启动）
    blkdeviotune                   Set or query a block device I/O tuning parameters.（设置或查询磁盘I/ O参数）
    blkiotune                      Get or set blkio parameters（获取或设置blkio参数）
    blockcommit                    Start a block commit operation.（将磁盘文件的变化保存到备份文件中）
    blockcopy                      Start a block copy operation.（拷贝磁盘备份镜像到目的地）
    blockjob                       Manage active block operations（管理active状态的磁盘任务）
    blockpull                      Populate a disk from its backing image.（从备份镜像中拷贝数据到磁盘）
    blockresize                    Resize block device of domain.（给虚拟机的块设备扩容）
    change-media                   Change media of CD or floppy drive（更新cdrom或floppy设备）
    console                        connect to the guest console（通过控制连接虚拟机）
    cpu-baseline                   compute baseline CPU（计算基准CPU）
    cpu-compare                    compare host CPU with a CPU described by an XML file（将XML中配置的CPU与HostCPU进行对比）
    cpu-stats                      show domain cpu statistics（显示CPU统计信息）
    create                         create a domain from an XML file（根据XML配置创建并启动一个非持久化虚拟机）
    define                         define (but don't start) a domain from an XML file（根据XML配置创建一个虚拟机）
    desc                           show or set domain's description or title（显示或设置虚拟机的描述或者标题）
    destroy                        destroy (stop) a domain（强制删除一个虚拟机）
    detach-device                  detach device from an XML file（根据XML定义删除虚拟机的设备）
    detach-disk                    detach disk device（删除磁盘设备）
    detach-interface               detach network interface（删除网络设备）
    domdisplay                     domain display connection URI（显示虚拟机的连接URI）
    domfsfreeze                    Freeze domain's mounted filesystems.(冻结虚拟机的挂载文件系统)
    domfsthaw                      Thaw domain's mounted filesystems.(解冻虚拟机的挂载文件系统)
    domfsinfo                      Get information of domain's mounted filesystems.
    domfstrim                      Invoke fstrim on domain's mounted filesystems.(在虚拟机挂载的文件系统上执行fstrim命令)
    domhostname                    print the domain's hostname(打印虚拟机的hostname)
    domid                          convert a domain name or UUID to domain id(把虚拟机的名字或UUID转化为虚拟机ID)
    domif-setlink                  set link state of a virtual interface(虚拟机设备接口链路状态设置)
    domiftune                      get/set parameters of a virtual interface(获取或者设置虚拟机设备接口得参数)
    domjobabort                    abort active domain job(终止虚拟机的一个处于active的任务)
    domjobinfo                     domain job information(获取虚拟机的任务信息)
    domname                        convert a domain id or UUID to domain name(通过虚拟机的ID或者UUID获取虚拟机的名字)
    dompmsuspend                   suspend a domain gracefully using power management functions(使用电源管理功能将虚拟机优雅挂起)
    dompmwakeup                    wakeup a domain from pmsuspended state(唤醒使用dompmsuspend挂起的虚拟机)
    domuuid                        convert a domain name or id to domain UUID(通过虚拟机的name或ID获取虚拟机的UUID)
    domxml-from-native             Convert native config to domain XML(将已存在的一组QEMU参数转成可以被libvirt使用Domain XML文件)
    domxml-to-native               Convert domain XML to native config(将已存在的libvirt使用的Domain XML文件转化为QEMU参数)
    dump                           dump the core of a domain to a file for analysis(dump出虚拟机的内存文件)
    dumpxml                        domain information in XML(获取虚拟机的XML配置信息)
    edit                           edit XML configuration for a domain(编辑虚拟机的XML配置文件)
    event                          Domain Events(获取虚拟机事件)
    inject-nmi                     Inject NMI to the guest(注入NMI中断到虚拟机内部)
    iothreadinfo                   view domain IOThreads
    iothreadpin                    control domain IOThread affinity
    iothreadadd                    add an IOThread to the guest domain
    iothreaddel                    delete an IOThread from the guest domain
    send-key                       Send keycodes to the guest(往虚拟机发送键盘按键指令)
    send-process-signal            Send signals to processes(给进程发送信号)
    lxc-enter-namespace            LXC Guest Enter Namespace(进入LXC虚拟机的命名空间)
    managedsave                    managed save of a domain state(管理保存虚拟机的状态,将虚拟机保存并销毁)
    managedsave-remove             Remove managed save of a domain（删除虚拟机状态管理设置）
    memtune                        Get or set memory parameters（获取或者设置内存参数）
    metadata                       show or set domain's custom XML metadata
    migrate                        migrate domain to another host（将虚拟机迁移到另一个节点）
    migrate-setmaxdowntime         set maximum tolerable downtime（设置停机迁移时间）
    migrate-compcache              get/set compression cache size（获取或者设置内存压缩的cache大小）
    migrate-setspeed               Set the maximum migration bandwidth（设置迁移的最大带宽）
    migrate-getspeed               Get the maximum migration bandwidth（获取迁移的最大带宽）
    numatune                       Get or set numa parameters（获取或者设置numa参数）
    qemu-attach                    QEMU Attach（QEMU Attach）
    qemu-monitor-command           QEMU Monitor Command（给qemu monitor发送命令）
    qemu-monitor-event             QEMU Monitor Events（监听qemu monitor事件）
    qemu-agent-command             QEMU Guest Agent Command（给qemu agent发送命令）
    reboot                         reboot a domain（优雅重启虚拟机）
    reset                          reset a domain（强制重启虚拟机）
    restore                        restore a domain from a saved state in a file（通过保存的内存文件恢复虚拟机）
    resume                         resume a domain（唤醒处于pause状态的虚拟机）
    save                           save a domain state to a file（将虚拟机状态保存到一个文件）
    save-image-define              redefine the XML for a domain's saved state file（根据save的虚拟机状态文件重定义XML）
    save-image-dumpxml             saved state domain information in XML(将虚拟机的状态保存到XML文件中)
    save-image-edit                edit XML for a domain's saved state file(修改保存虚拟机状态的XML文件)
    schedinfo                      show/set scheduler parameters(显示或设置scheduler参数)
    screenshot                     take a screenshot of a current domain console and store it into a file(截取当前虚拟机的屏幕，并将其保存到一个文件中)
    set-user-password              set the user password inside the domain
    setmaxmem                      change maximum memory limit(该变最大内存限制)
    setmem                         change memory allocation(设置内存大小)
    setvcpus                       change number of virtual CPUs(设置vcpu个数)
    shutdown                       gracefully shutdown a domain(优雅关闭虚拟机)
    start                          start a (previously defined) inactive domain(启动一个处于关闭状态的虚拟机)
    suspend                        suspend a domain(暂停一个虚拟机)
    ttyconsole                     tty console(tty console显示)
    undefine                       undefine a domain(删除虚拟机的定义)
    update-device                  update device from an XML file(根据XML配置更新虚拟机的设备)
    vcpucount                      domain vcpu counts(获取虚拟机vcpu个数)
    vcpuinfo                       detailed domain vcpu information(显示虚拟机vcpu的详细信息)
    vcpupin                        control or query domain vcpu affinity(控制或者查询虚拟机的vcpu亲和性)
    emulatorpin                    control or query domain emulator affinity(控制或者查询模拟器的亲和性)
    vncdisplay                     vnc display(获取vnc端口号)



#### 1 虚拟机操作相关 ####

**create**：   create create a domain from an XML file  
**define**：   define (but don't start) a domain from an XML file  
**start**：    start a (previously defined) inactive domain  
**destroy**：  destroy (stop) a domain  
**shutdown**： gracefully shutdown a domain  
**undefine**:  undefine a domain  

**reboot**： reboot a domain  
**reset**: reset a domain（Reset the target domain as if by power button）  

**suspend**： suspend a domain  
**resume**： resume a domain  

**save**：                           save a domain state to a file  
**restore**：                        restore a domain from a saved state in a file  
**screenshot**:                     take a screenshot of a current domain console and store it into a file  

**autostart**：                     autostart a domain(Configure a domain to be automatically started at boot)  

**managedsave**:                    managed save of a domain state(Save and destroy a running domain, so it can be restarted from the same state at a later time.  When the virsh 'start' command is next run for the domain, it will automatically be started from this saved state.)
**managedsave-remove**:             Remove managed save of a domain

**desc**：                           show or set domain's description or title  
**domhostname**:                    print the domain's hostname  
![](http://i.imgur.com/ptDCQMw.png)  

**domname**:                        convert a domain id or UUID to domain name(输出domain name)  
**metadata**：                       show or set domain's custom XML metadata

**domuuid**：                        convert a domain name or id to domain UUID  
**domid**：                          convert a domain name or UUID to domain id 
**numatune**：                       Get or set numa parameters



**emulatorpin**:                    control or query domain emulator affinity(Pin domain emulator threads to host physical CPUs)  
![](http://i.imgur.com/enpAzU4.png)  

#### 2.xml相关 ####

**domxml-from-native**:             Convert native config to domain XML  
**domxml-to-native**:               Convert domain XML to native config  
 
**dumpxml**:                        domain information in XML  
**edit**:                           edit XML configuration for a domain  

#### 3.cpu相关 ####

**vcpucount**:                      domain vcpu counts  
**vcpuinfo**:                       detailed domain vcpu information  
**vcpupin**:                        control or query domain vcpu affinity  
**setvcpus**:                       change number of virtual CPUs  
**cpu-baseline**:                   compute baseline CPU  
**cpu-compare**:                    compare host CPU with a CPU described by an XML file 
**cpu-stats**:                      show domain cpu statistics  

#### 4.mem相关 ####

**setmaxmem**:                      change maximum memory limit  
**setmem**:                         change memory allocation  
**memtune**:                        Get or set memory parameters  
![](http://i.imgur.com/qu88vJB.png)  

#### 5.blk ####

**blkdeviotune**:                   Set or query a block device I/O tuning parameters.  
![](http://i.imgur.com/aSHgubn.png)  

**blkiotune**:                      Get or set blkio parameters  
![](http://i.imgur.com/aACtYHL.png)  
**blockcommit**:                    Start a block commit operation.（Commit changes from a snapshot down to its backing image合并快照文件）  
**blockcopy**:                      Start a block copy operation.  
blockjob:                       Manage active block operations  
**blockpull**: Populate a disk from its backing image. （将backing file数据合并至overlay中） 
blockresize:                    Resize block device of domain.  

#### 6.attch/detach ####

**attach-device**:                  attach device from an XML file  
**attach-device**:                    attach disk device  
**attach-interface**:               attach network interface  

**detach-device**:                  detach device from an XML file  
**detach-disk**:                    detach disk device  
**detach-interface**:               detach network interface  

**update-device**：                  update device from an XML file  

#### 7.迁移相关 ####

**migrate**:                        migrate domain to another host  
**migrate-setmaxdowntime**:         set maximum tolerable downtime  
**migrate-compcache**:              get/set compression cache size  
**migrate-setspeed**:               Set the maximum migration bandwidth  
**migrate-getspeed**:               Get the maximum migration bandwidth  


#### 8.guest agent ####

QEMU guest agent 是一个运行在虚拟机内部的普通应用程序，其目的是实现一种宿主机和虚拟机进行交互的方式这种方式不依赖于网络，而是依赖于VirtIO serial。  
QEMU提供了串口设备的模拟及数据交换的通道，最终呈现出来的是一个串口设备（虚拟机内部）和一个unix socket文件（宿主机上）。  
通过QEMU guest agent, 宿主机可以控制虚拟机实现冻结/恢复/整理文件系统（freeze and thaw filesystems)、列出IP地址等功能。  

**set-user-password**： set the user password inside the domain  
**domfsfreeze**:        Freeze domain's mounted filesystems.  
**domfsthaw**:          Thaw domain's mounted filesystems.  
**domfsinfo**:          Get information of domain's mounted filesystems.  
**domfstrim**:          Invoke fstrim on domain's mounted filesystems.  
 
![](http://i.imgur.com/pfMm2Ud.png)  

**dompmsuspend**:                    suspend a domain gracefully using power management functions  
**dompmwakeup**:                     wakeup a domain from pmsuspended state  

#### 9.io thread ####

当前不知道是配置，还是什么原因，没有相关的信息

**iothreadinfo**：                   view domain IOThreads  
![](http://i.imgur.com/WrZSDlu.png)  
**iothreadpin**：                    control domain IOThread affinity  
**iothreadadd**：                    add an IOThread to the guest domain  
**iothreaddel**：                    delete an IOThread from the guest domain  


#### 10 操纵save生成的文件 ####

这里主要是处理save生成的文件，包括以xml形式展现等  

**save-image-define**：             redefine the XML for a domain's saved state file(Replace the domain XML associated with a saved state file)  
**save-image-dumpxml**：            saved state domain information in XML（Extract the domain XML that was in effect at the time the saved state file file was created with the save command）  
**save-image-edit**:                edit XML for a domain's saved state file  

#### 11.Qemu-specific Commands ####

Use of the following commands is strongly discouraged.   
They can cause libvirt to become confused and do the wrong thing on subsequent operations.  
Once you have used this command, please do not report problems to the libvirt developers; the reports will be ignored.  

qemu-attach                    QEMU Attach  
qemu-monitor-command           QEMU Monitor Command  
qemu-monitor-event             QEMU Monitor Events  
qemu-agent-command             QEMU Guest Agent Command  

#### 12.网络相关 ####

**domif-setlink**:                  set link state of a virtual interface  

    virsh domif-setlink domain vnet0 down
    virsh domif-setlink domain vnet0 up

**domiftune**:                      get/set parameters of a virtual interface 

#### 13.other ####

domjobabort:                        abort active domain job  
**domjobinfo**:                     domain job information  

**dump**:                           dump the core of a domain to a file for analysis  
**domdisplay**:                     domain display connection URI
**inject-nmi**：                    Inject NMI(不可屏蔽中断)  to the guest（ This is used when response time is critical, such as non-recoverable hardware errors）  
**vncdisplay**:                     vnc display（Output the IP address and port number for the VNC display）  
**ttyconsole**:                     tty console（Output the device for the TTY console）  
**schedinfo**:                      show/set scheduler parameters  
![](http://i.imgur.com/uBLVi2a.png)  
**lxc-enter-namespace**:            LXC Guest Enter Namespace(The virsh lxc-enter-namespace command can be used to enter the namespaces and security context of a container and then execute an arbitrary command.)  

**send-key**：                       Send keycodes to the guest  
![](http://i.imgur.com/5VAQwJR.png)  
在虚拟机内部,每执行一条上述的send-key都会回显一个key字符串  
![](http://i.imgur.com/TzbbzOh.png)
**send-process-signal**：            Send signals to processes

**console**：                        connect to the guest console
**event**：                          Domain Events