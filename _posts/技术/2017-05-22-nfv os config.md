---
layout: post
title: 虚拟化实时性提升（零）之配置步骤
category: 技术
tags: 虚拟化层
keywords: 
description: RT-Preempt,linux realtime,rtmutexes
---

## 背景 ##

NFV场景下，电信级背景中对系统，或者说对虚拟机中的实时性有要求  
本文使用相应调优手段，减少虚拟机的时延  

## BIOS设置 ##

    CPU Power and Performance <Performance>
    CPU C-State <Disabled> 
    C1E Autopromote <Disabled>
    Processor C3 <Disabled>
    Processor C6 <Disabled>
    Select Memory RAS <Maximum Performance>
    NUMA Optimized <Enabled>
    Cluster-on-Die <Disabled>
    Patrol Scrub <Disabled>
    Demand Scrub <Disabled>
    Correctable Error <10>
    Intel(R) Hyper-Threading <Disabled>
    Active Processor Cores <All>
    Execute Disable Bit <Enabled>
    Intel(R) Virtualization Technology <Enabled>
    Intel(R) TXT <Disabled>
    Enhanced Error Containment Mode <Disabled>
    USB Controller <Enabled>
    USB 3.0 Controller <Auto>
    Legacy USB Support <Disabled>
    Port 60/64 Emulation <Disabled>

对应到不同的宿主机类型，可能选配项不同  
修改对应的项目  

## HOST ##

### 切rt内核 ###

将内核替换为实时内核  
这里采用kernel-rt-3.10.0-514.6.1.rt56.429.el7.x86_64.rpm  
（注意，会有一系列依赖包要装）

本文通过[srpm](http://vault.centos.org/7.3.1611/rt/)本地编译，安装  
更多rt相关内容参看本人这篇博文  

### 优化kernel启动参数 ###

在内核引导配置/boot/grub2/grub.cfg中，对应内核后追加如下参数  
更全面的内核参数，参看本人这篇博文

    isolcpus=20,21,22,23 nohz_full=20-23 rcu_nocbs=20-23 iommu=pt intel_iommu=on default_hugepagesz=1G
    hugepagesz=1G mce=off idle=poll intel_pstate=disable processor.max_cstate=1 pcie_asmp=off tsc=reliable

** 1. isolcpus **

用户可以使用 isolcpus 开机参数来从调度器隔离一个或多个 CPU，以此防止调度器在此 CPU 上调度任何用户空间的线程。  
一旦 CPU 被隔离，用户须手动分配进程至被隔离的 CPU，或使用 CPU 关联系统呼叫或 numactl 命令。  
我们这里隔离出一定数目的cpu给虚拟机的vcpu线程使用，在虚拟机配置文件中使用cpupin绑定，详见后文。  

**验证：**  
通过如下方法查询对应cpu上是否有用户态线程  

    ps -eo psr,cmd |grep "  3"

** 2. nohz_full **

提供一种动态的无时钟设置,在内核"CONFIG_NO_HZ_FULL=y"的前提下，指定哪些CPU核心可以进入完全无滴答状态  
配置了CONFIG_NO_HZ_FULL=y后，当cpu上只有一个任务在跑的时候，不发送调度时钟中断到此cpu  
也就是减少调度时钟中断，不中断空闲CPU，从而可以减少耗电量和减少系统抖动  
默认情况下所有的cpu都不会进入这种模式，需要通过nohz_full参数指定那些cpu进入这种模式  

当这一设置在一个内核中启用时，所有的计时活动将会被移动至无延迟敏感性的内核。  

"CPU列表"是一个逗号分隔的CPU编号(从0开始计数)，也可以使用"-"界定一个范围。例如"0,2,4-7"等价于"0,2,4,5,6,7"  
[注意]  
(1)"boot CPU"(通常都是"0"号CPU)会无条件的从列表中剔除。  
(2)这里列出的CPU编号必须也要同时列进"rcu_nocbs=..."选项中。  
(3)系统启动的时候，必须手动将 rcu 线程移动至对延迟不敏感的内核，比如为 0 内核  

    for i in `pgrep rcu` ; do taskset -pc 0 $i ; done

验证方法：  
[git://git.kernel.org/pub/scm/linux/kernel/git/frederic/dynticks-testing.git](git://git.kernel.org/pub/scm/linux/kernel/git/frederic/dynticks-testing.git)

** 3.rcu_nocbs **

当cpu有RCU callbacks pending的时候,nohz_full设置可能不会生效  
rcu_nocbs来指定cpu 进行卸载RCU callback processing  

**4. iommu=pt和intel_iommu**

使用Intel® VT-d，在使用igb_uio驱动时必须设置内核参数iommu=pt  
使用vfio-pci驱动必须设置iommu=pt and iommu=on  
网卡是intel的，如果内核选项没有默认设置INTEL_IOMMU_DEFAULT_ON，可以设置内核参数intel_iommu=on

**5. default_hugepagesz**

默认的HugeTLB页大小。  
若未指定，那么其默认值就是CPU自身的默认值。  
大多数现代计算机体系结构提供对多页面大小的支持，比如X86_64支持4K和2M(要求CPU带有"pse"标记)以及1G(要求CPU带有"pdpe1gb"标记)。  
因此Linux将物理内存划分成许多固定大小的页面(默认为4K)，每个页对应一个page结构，这些结构组成一个mem_map[]数组。  
TLB(Translation Lookaside Buffer)是虚拟地址到物理地址的翻译缓冲区，这种缓冲区在处理器上是很宝贵的，操作系统总是尝试将有限的TLB资源发挥到极致。  
而HugeTLB特性则允许将某些页的尺寸增大到2MB或1GB，从而大大减小TLB的尺寸，提高缓冲区的命中率，进而提升内存性能。

**6. hugepagesz**

指定HugeTLB页的大小，通常与"hugepages="联合使用(可使用多次)，为不同尺寸的大页分别预留不同的数量。  
例如：hugepagesz=2M hugepages=128 hugepagesz=1G hugepages=8  
注意：1GB的大页只能在命令行上使用"hugepages="预先分配，且分配之后不可在运行时释放。  

**7. mce=off**

彻底禁用MCE  
MCE(Machine Check Exception)是用来报告主机硬件相关问题的一种日志机制.

**8. idle=poll**

从根本上禁用休眠功能(也就是禁止进入C-states状态)  

C-state表示处理器处于空闲状态（只有C0表示处理器在正常运行）。  
像C1、C1E、C3以及C6这样的状态表示处理器核心空闲时能够进入的电源保护模式。  
当处理器核心处于C6状态时，核心基本上就完全关闭了。  
因为核心处于空闲状态，用户从不会感觉到C-state带来的影响。  

**9. intel_pstate=disable**

禁用 Intel CPU 的 P-state 驱动(CONFIG_X86_INTEL_PSTATE)，也就是Intel CPU专用的频率调节器驱动  
P-state表明处理器处于省电模式但仍旧在执行一些工作。  
处理器处于P态的一个常见的例子就是在系统的电源配置文件中“低功率”配置文件将会降低处理器的电压以及时钟频率。  
较高的P-state能够显著节省电力，但是肯定会影响工作负载的性能  

**10. processor.max_cstate**

无视ACPI表报告的值，强制指定CPU的最大C-state值(必须是一个有效值)：  
C0为正常状态，其他则为不同的省电模式(数字越大表示CPU休眠的程度越深/越省电)。  
"9"表示无视所有的DMI黑名单限制。

**11. pcie_asmp=off**

主​​​​​​​动​​​​​​​式​​​​​​​电​​​​​​​源​​​​​​​管​​​​​​​理​​​​​​​（ASPM）在​​​​​​​ PCI Express 或​​​​​​​者​​​​​​​ PCIe（Peripheral Component Interconnect Express）子​​​​​​​系​​​​​​​统​​​​​​​中​​​​​​​的​​​​​​​节​​​​​​​电​​​​​​​  
其​​​​​​​原​​​​​​​理​​​​​​​为​​​​​​​当​​​​​​​设​​​​​​​备​​​​​​​连​​​​​​​接​​​​​​​的​​​​​​​ PCI 连​​​​​​​接​​​​​​​没​​​​​​​有​​​​​​​处​​​​​​​于​​​​​​​使​​​​​​​用​​​​​​​状​​​​​​​态​​​​​​​时​​​​​​​将​​​​​​​其​​​​​​​设​​​​​​​定​​​​​​​为​​​​​​​低​​​​​​​功​​​​​​​率​​​​​​​状​​​​​​​态​​​​​​​。  
​​​​​​​ASPM 可​​​​​​​同​​​​​​​时​​​​​​​在​​​​​​​终​​​​​​​端​​​​​​​和​​​​​​​连​​​​​​​接​​​​​​​中​​​​​​​控​​​​​​​制​​​​​​​电​​​​​​​源​​​​​​​状​​​​​​​态​​​​​​​，并​​​​​​​在​​​​​​​连​​​​​​​接​​​​​​​终​​​​​​​端​​​​​​​的​​​​​​​设​​​​​​​备​​​​​​​处​​​​​​​于​​​​​​​满​​​​​​​电​​​​​​​状​​​​​​​态​​​​​​​时​​​​​​​仍​​​​​​​可​​​​​​​在​​​​​​​连​​​​​​​接​​​​​​​中​​​​​​​节​​​​​​​电​​​​​​​。​​​​​​​

**12. tsc=reliable**

tsc=reliable 表示TSC时钟源是绝对稳定的，关闭启动时和运行时的稳定性检查。  
用于在某些老旧硬件/虚拟化环境使用TSC时钟源。

### 其他优化手段 ###

1. 关闭swap  

    sudo swapoff -a

2. 关闭ksm  

     echo 0 > /sys/kernel/mm/ksm/merge_across_nodes && echo 0 > /sys/kernel/mm/ksm/run

如果当前有copy-on-write的页，通过如下方法回合  
然后在执行上一句

    echo 2 > /sys/kernel/mm/ksm/run && sleep 300 && cat /sys/kernel/mm/ksm/pages_shared

3. 关闭看门狗

linux下每个CPU都有一个看门狗(watchdog)进程  
该进程每秒获取其CPU的当前时间戳并保存于per-CPU，而timer interrupt()会调用softlock_tick()  
该函数比较CPU当前时间与per-CPU保存的时间，若差值大于softlockup_thresh则系统产生一条告警信息。  

NMI watchdog(non maskable interrupt)又称硬件watchdog，用于检测OS是否挂死，系统硬件定期产生一个NMI  
而每个NMI调用内核查看其中断数量，如果一段时间(10秒)后其数量没有显著增长，则判定系统已经挂死  
接下来启用panic机制即重启OS，如果开启了Kdump还会产生crash dump文件  

避免造成实时毛刺，通过如下方式关闭  

    echo 0 > /proc/sys/kernel/watchdog
    echo 0 > /proc/sys/kernel/nmi_watchdog

4. 调整隔离cpu上的ksoftirqd和rcuc的优先级

查询出隔离cpu上的ksoftirqd和rcuc(RCU callbacks)进程  
由于在前文中我们已经设置隔离cpu的rcu_nocbs来卸载RCU callbacks（见前文）  
通过chrt设置调度策略为SCHED_FIFO，并调低优先级  

    host_isolcpus="20-23"
    startVal=$(echo ${host_isolcpus} | cut -f1 -d-)
    endVal=$(echo ${host_isolcpus} | cut -f2 -d-)
    i=0
    while [ ${startVal} -le ${endVal} ]; do
        tid=`pgrep -a ksoftirq | grep "ksoftirqd/${startVal}$" | cut -d ' ' -f 1`
        chrt -fp 2 ${tid}
    
        tid=`pgrep -a rcuc | grep "rcuc/${startVal}$" | cut -d ' ' -f 1`
        chrt -fp 3 ${tid}
    
        cpu[$i]=${startVal}
        i=`expr $i + 1`
        startVal=`expr $startVal + 1`
    done

查询出rcub进程  
rcub(RCU boosting)主要是为了解决中断逆置问题  
前文已经设置了cpu隔离和nohz_full,除了ipi中断或者vcpu的 irq delivery，被隔离的cpu不会被其他任务使用  
通过chrt设置调度策略为SCHED_FIFO，并调低优先级  

    for tid in `pgrep -a rcub | cut -d ' ' -f 1` ; do
        chrt -fp 3 ${tid}
    done


SCHED_FIFO （也叫做静态优先级调度）是一项实时策略，定义了每个线程的固定优先级  
在使用 SCHED_FIFO 时，调度器会按优先级顺序扫描所有的 SCHED_FIFO 线程，并对准备运行的最高优先级线程进行调度。  
一个 SCHED_FIFO 线程的优先级级别可以是 1 至 99 之间的任何整数，99 是最高优先级。

5. 禁止带宽限制

    echo -1 > /proc/sys/kernel/sched_rt_period_us
    echo -1 > /proc/sys/kernel/sched_rt_runtime_us

管理员可以限制 SCHED_FIFO 的带宽以防止实时应用程序的程序员启用独占处理器的实时任务。
⁠该参数以微秒为单位来定义时间，是百分之百的处理器带宽。默认值为 1000000 μs, 或1秒。
⁠该参数以微秒为单位来定义时间，用来运行实时线程。默认值为 950000 μs, 或0.95秒。

这里设为-1禁止生效 

6. 设置中断亲和性  

将绑定到隔离cpu上的中断路由导到cpu0上  

    for irq in /proc/irq/* ; do
       echo 0 > ${irq}/smp_affinity_list
    done

7.禁用trace  

    set -o xtrace
    curpwd=`pwd`
    TRACE_FILE=trace.txt
    TRACEDIR=/sys/kernel/debug/tracing
    
    bash -c "echo 0 > $TRACEDIR/tracing_on"
    sleep 1
    bash -c "cat $TRACEDIR/trace > /tmp/$TRACE_FILE"
    
    bash -c "echo > $TRACEDIR/set_event"
    bash -c "echo > $TRACEDIR/trace"
    sysctl kernel.ftrace_enabled=0
    bash -c "echo nop > $TRACEDIR/current_tracer"
    
    set +o xtrace
    cd $curpwd

### 虚拟机设置 ###

本节针对虚拟机配置做相关规划  
也就是说在创建虚拟机的时候，虚拟机的配置文件追加如下配置  

1. cpu绑定  

将vcpu绑定都内核隔离出来的物理cpu中  
在本例中将虚拟机的4个vcpu绑定到物理cpu 20-23中  

    <cputune>
        <vcpupin vcpu="0" cpuset="20"/>
        <vcpupin vcpu="1" cpuset="21"/>
        <vcpupin vcpu="2" cpuset="22"/>
        <vcpupin vcpu="3" cpuset="23"/>
    </cputune>

2. 开启大页  

通过以下命令申请16个大页（本例中虚拟机为16G）  
（dpdk中需要使用大页提高传输效率，这里开启大页后实测实时性可以有1us的提升）  

    sysctl vm.nr_hugepages=16

在虚拟机的配置文件中追加以下配置  
设置虚拟机使用大页  

    <memoryBacking>
      <hugepages/>
    </memoryBacking>

3. 禁用kvmclock  

虚拟机里面的中断不是真正的中断，是qemu或者准确说kvm注入到虚拟机中的  
因此中断不能即时的被处理，只有进入非根（no-root）模式，也就是vm-entry的时候中断才能够被注入到物理cpu中去  
具体参考本人这篇博文《中断虚拟化（pic）到底是咋整的》  
这就导致通过时钟中断计数，再换算为时间这种方式是有偏移的，是不准确的  

除此之外，虚拟机主动vm-exit去读取当前的时间，这种方式会导致虚拟机频繁的退出，影响性能。  

Kvm为了解决这个问题，引入了kvmclock  
虚拟机开辟一块内存，由host(VMM)根据需要将hostos的时间更新到内存中  
当虚拟机需要时间的时候，到这块内存中去读取  

![](http://i.imgur.com/oMI0Dbu.png)

      <clock offset='utc'>
          <timer name='kvmclock' present='no'/>
      </clock>

使用上述配置后，虚拟机里面的时钟源被退化为tsc  
tsc是借用的物理cpu上的时钟频率  

**验证：**  

cat /sys/devices/system/clocksource/clocksource0/available_clocksource
cat /sys/devices/system/clocksource/clocksource0/current_clocksource

## GUEST ##

此节是针对guestOS的设置  
为获得较好的结果，kenel采用与host一致的rt内核  

### 优化kernel启动参数 ###

    isolcpus=3 nohz_full=3 rcu_nocbs=3  mce=off idle=poll default_hugepagesz=1G hugepagesz=1G

### 其他优化手段 ###

1. 关闭swap  

    sudo swapoff -a

2. 关闭ksm  

     echo 0 > /sys/kernel/mm/ksm/merge_across_nodes && echo 0 > /sys/kernel/mm/ksm/run
     echo 2 > /sys/kernel/mm/ksm/run && sleep 300 && cat /sys/kernel/mm/ksm/pages_shared

3. 关闭看门狗

    echo 0 > /proc/sys/kernel/watchdog
    echo 0 > /proc/sys/kernel/nmi_watchdog

5. 禁止带宽限制

    echo -1 > /proc/sys/kernel/sched_rt_period_us
    echo -1 > /proc/sys/kernel/sched_rt_runtime_us

6. 禁止时钟迁移

    echo 0 > /proc/sys/kernel/timer_migration

## 测试 ##

期间测试了内核社区（3.10，4.4）  
测试了centos7.3下的rt包  

其中centos7.3下的rt包，同时叠加上述调优得到的结果最好  

### 测试命令 ###

使用cyclictest测试实时性  
这里将cyclictest绑定到vcpu3上  
测试持续时间10分钟，共测试五组

    taskset -c 3 cyclictest -m -n -p95 -h60 -i200 -D 10m

### 测试结果 ###

#### 优化前 ####

    [root@localhost ~]# taskset -c 3 cyclictest -m -n -p95 -h60 -i200 -D 10m
    # /dev/cpu_dma_latency set to 0us
    policy: fifo: loadavg: 0.00 0.01 0.03 1/129 2407          
    
    T: 0 ( 2404) P:95 I:200 C:2999375 Min:      5 Act:   22 Avg:   16 Max:    1475
    # Histogram
    000000 000000
    000001 000000
    000002 000000
    000003 000000
    000004 000000
    000005 000003
    000006 000349
    000007 003581
    000008 060662
    000009 047736
    000010 030176
    000011 021747
    000012 016957
    000013 023105
    000014 033531
    000015 098533
    000016 154254
    000017 1248460
    000018 1017814
    000019 150627
    000020 060286
    000021 024083
    000022 005110
    000023 000937
    000024 000181
    000025 000112
    000026 000057
    000027 000052
    000028 000072
    000029 000053
    000030 000032
    000031 000020
    000032 000016
    000033 000011
    000034 000011
    000035 000010
    000036 000020
    000037 000019
    000038 000020
    000039 000015
    000040 000011
    000041 000020
    000042 000026
    000043 000021
    000044 000027
    000045 000013
    000046 000010
    000047 000005
    000048 000002
    000049 000001
    000050 000001
    000051 000007
    000052 000005
    000053 000006
    000054 000002
    000055 000002
    000056 000002
    000057 000001
    000058 000005
    000059 000002
    # Total: 002998821
    # Min Latencies: 00005
    # Avg Latencies: 00016
    # Max Latencies: 01475
    # Histogram Overflows: 00604
    # Histogram Overflow at cycle number:
    # Thread 0: 04405 08373 14643 18371 24881 28369 35119 38367 45357 48365 55595 58363 65833 68361 76071 78359 86309 88357 93843 93844 93845 96546 96547 98348 106778 106779 106962 106964 108346 117016 118344 127254 128342 137492 138340 147730 148338 157968 158336 168206 168334 178332 188330 188680 198328 198918 208326 209156 218324 219394 228322 229632 238320 239870 248318 250108 250109 255556 258310 260340 # 00544 others

#### 优化后 ####

    [root@localhost home]# taskset -c 3 cyclictest -m -n -p95 -h60 -i200 -D 10m
    # /dev/cpu_dma_latency set to 0us
    policy: fifo: loadavg: 0.00 0.01 0.01 1/152 2441
    
    T: 0 ( 2440) P:95 I:200 C:2999967 Min:  6 Act:7 Avg:7 Max:  12
    # Histogram
    000000 000000
    000001 000000
    000002 000000
    000003 000000
    000004 000000
    000005 000000
    000006 068170
    000007 056487
    000008 2790777
    000009 006079
    000010 072488
    000011 005997
    000012 000002
    000013 000000
    ...
    # Total: 003000000
    # Min Latencies: 00006
    # Avg Latencies: 00007
    # Max Latencies: 00012
    # Histogram Overflows: 00000
    # Histogram Overflow at cycle number:
    # Thread 0:


### 参考文献 ###

[https://wiki.opnfv.org/display/kvm/Nfv-kvm-tuning](https://wiki.opnfv.org/display/kvm/Nfv-kvm-tuning)  

[https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Host_Configuration_and_Guest_Installation_Guide/chap-Virtualization_Host_Configuration_and_Guest_Installation_Guide-KVM_guest_timing_management.html](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/6/html/Virtualization_Host_Configuration_and_Guest_Installation_Guide/chap-Virtualization_Host_Configuration_and_Guest_Installation_Guide-KVM_guest_timing_management.html)  

[https://rwmj.wordpress.com/tag/kvmclock/](https://rwmj.wordpress.com/tag/kvmclock/)

[https://wiki.opnfv.org/display/kvm/KVM4NFV+Test++Environment](https://wiki.opnfv.org/display/kvm/KVM4NFV+Test++Environment)

[https://gerrit.opnfv.org/gerrit/gitweb?p=kvmfornfv.git;a=tree;h=d806a3a9443fcf29245e37aa0efc5f4a4de5230c;hb=ebcdab040619575a47ad19b18e7ed5c38d287336
https://github.com/opnfv/kvmfornfv](https://gerrit.opnfv.org/gerrit/gitweb?p=kvmfornfv.git;a=tree;h=d806a3a9443fcf29245e37aa0efc5f4a4de5230c;hb=ebcdab040619575a47ad19b18e7ed5c38d287336)