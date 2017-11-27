---
layout: post
title: 虚拟内存情况dommemstat分析
category: 技术
tags: 技术
keywords:
description: 
---

有同事问dommemstat详细含义，读了下代码,记录如下  
之前分析过，dommemstat获取到详细的情况需要virtio-ballon的支持  
详细见本人[这篇博文](www.hanbaoying.com/2017/03/20/Virtio-Balloon.html) 


----------

![](https://i.imgur.com/HnxilOk.png)


最难懂的是如下这几个字段：  
unused  
available  
usable  


通读libvirt代码会发现，是调用的qemu的guest-stat方法

    if (!(cmd = qemuMonitorJSONMakeCommand("qom-get",
                                           "s:path", balloonpath,
                                           "s:property", "guest-stats",
                                           NULL)))
                                           
然后获取到值后封装返回


    GET_BALLOON_STATS(statsdata, "stat-swap-in",
                      VIR_DOMAIN_MEMORY_STAT_SWAP_IN, 1024);
    GET_BALLOON_STATS(statsdata, "stat-swap-out",
                      VIR_DOMAIN_MEMORY_STAT_SWAP_OUT, 1024);
    GET_BALLOON_STATS(statsdata, "stat-major-faults",
                      VIR_DOMAIN_MEMORY_STAT_MAJOR_FAULT, 1);
    GET_BALLOON_STATS(statsdata, "stat-minor-faults",
                      VIR_DOMAIN_MEMORY_STAT_MINOR_FAULT, 1);
    GET_BALLOON_STATS(statsdata, "stat-free-memory",
                      VIR_DOMAIN_MEMORY_STAT_UNUSED, 1024);
    GET_BALLOON_STATS(statsdata, "stat-total-memory",
                      VIR_DOMAIN_MEMORY_STAT_AVAILABLE, 1024);
    GET_BALLOON_STATS(statsdata, "stat-available-memory",
                      VIR_DOMAIN_MEMORY_STAT_USABLE, 1024);
    GET_BALLOON_STATS(data, "last-update",
                      VIR_DOMAIN_MEMORY_STAT_LAST_UPDATE, 1);
                      
其中libvirt中各个字段含义如下：

    /*
     * The amount of memory left completely unused by the system.  Memory that
     * is available but used for reclaimable caches should NOT be reported as
     * free.  This value is expressed in kB.
     */
    VIR_DOMAIN_MEMORY_STAT_UNUSED          = 4,

    /*
     * The total amount of usable memory as seen by the domain.  This value
     * may be less than the amount of memory assigned to the domain if a
     * balloon driver is in use or if the guest OS does not initialize all
     * assigned pages.  This value is expressed in kB.
     */
    VIR_DOMAIN_MEMORY_STAT_AVAILABLE       = 5,
    /*
     * How much the balloon can be inflated without pushing the guest system
     * to swap, corresponds to 'Available' in /proc/meminfo
     */
    VIR_DOMAIN_MEMORY_STAT_USABLE          = 8,


----------

![](https://i.imgur.com/IEF6WXv.png)


通过[这篇博文](www.hanbaoying.com/2017/03/20/Virtio-Balloon.html)我们知道  
数据是从virtio-balloon的前端获取的，guest的kernel中

	unsigned long events[NR_VM_EVENT_ITEMS];
	struct sysinfo i;
	int idx = 0;
	long available;

	all_vm_events(events);
	si_meminfo(&i);

	available = si_mem_available();

	update_stat(vb, idx++, VIRTIO_BALLOON_S_SWAP_IN,
				pages_to_bytes(events[PSWPIN]));
	update_stat(vb, idx++, VIRTIO_BALLOON_S_SWAP_OUT,
				pages_to_bytes(events[PSWPOUT]));
	update_stat(vb, idx++, VIRTIO_BALLOON_S_MAJFLT, events[PGMAJFAULT]);
	update_stat(vb, idx++, VIRTIO_BALLOON_S_MINFLT, events[PGFAULT]);
	update_stat(vb, idx++, VIRTIO_BALLOON_S_MEMFREE,
				pages_to_bytes(i.freeram));
	update_stat(vb, idx++, VIRTIO_BALLOON_S_MEMTOT,
				pages_to_bytes(i.totalram));
	update_stat(vb, idx++, VIRTIO_BALLOON_S_AVAIL,
				pages_to_bytes(available));


各个字段含义如下(注意qemu2.3中没有VIRTIO_BALLOON_S_AVAIL)：  

	#define VIRTIO_BALLOON_S_SWAP_IN  0   /* Amount of memory swapped in */
	#define VIRTIO_BALLOON_S_SWAP_OUT 1   /* Amount of memory swapped out */
	#define VIRTIO_BALLOON_S_MAJFLT   2   /* Number of major faults */
	#define VIRTIO_BALLOON_S_MINFLT   3   /* Number of minor faults */
	#define VIRTIO_BALLOON_S_MEMFREE  4   /* Total amount of free memory */
	#define VIRTIO_BALLOON_S_MEMTOT   5   /* Total amount of memory */
	#define VIRTIO_BALLOON_S_AVAIL    6   /* Available memory as in /proc */
	#define VIRTIO_BALLOON_S_NR       7

![](https://i.imgur.com/qQiAijP.png)

----------

**总结**（virsh命令中三个字段对应guestos的/proc/meminfo字段）：  

unused——>Memfree  
available->MemTotal  
usable->MemAvailable  


其中内核中各字段含义如下：  

**MemTotal**

系统从加电开始到引导完成，firmware/BIOS要保留一些内存，kernel本身要占用一些内存，最后剩下可供kernel支配的内存就是MemTotal。  
这个值在系统运行期间一般是固定不变的。  
可参阅解读DMESG中的内存初始化信息。

**MemFree**

表示系统尚未使用的内存。  
[MemTotal-MemFree]就是已被用掉的内存。

**MemAvailable**

有些应用程序会根据系统的可用内存大小自动调整内存申请的多少，所以需要一个记录当前可用内存数量的统计值。  
MemFree并不适用，因为MemFree不能代表全部可用的内存，系统中有些内存虽然已被使用但是可以回收的。  
比如cache/buffer、slab都有一部分可以回收，所以这部分可回收的内存加上MemFree才是系统可用的内存，即MemAvailable。  
/proc/meminfo中的MemAvailable是内核使用特定的算法估算出来的，要注意这是一个估计值，并不精确。



参考文献：

http://linuxperf.com/?p=142