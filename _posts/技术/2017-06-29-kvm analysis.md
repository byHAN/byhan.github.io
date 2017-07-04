---
layout: post
title: kvm分析工具
category: 技术
tags: 虚拟化
keywords: 
description: kvm
---


    # echo 1 >/sys/kernel/debug/tracing/events/kvm/enable
    # cat /sys/kernel/debug/tracing/trace_pipe
    [...]
    kvm-5664  [000] 11906.220178: kvm_entry: vcpu 0
    kvm-5664  [000] 11906.220181: kvm_exit: reason apic_access rip 0xc011518c
    kvm-5664  [000] 11906.220183: kvm_mmio: mmio write len 4 gpa 0xfee000b0 val 0x0
    kvm-5664  [000] 11906.220183: kvm_apic: apic_write APIC_EOI = 0x0
    kvm-5664  [000] 11906.220184: kvm_ack_irq: irqchip IOAPIC pin 11
    kvm-5664  [000] 11906.220185: kvm_entry: vcpu 0
    kvm-5664  [000] 11906.220188: kvm_exit: reason io_instruction rip 0xc01e4473
    kvm-5664  [000] 11906.220188: kvm_pio: pio_read at 0xc13e size 2 count 1
    kvm-5664  [000] 11906.220193: kvm_entry: vcpu 0
    ^D
    # echo 0 >/sys/kernel/debug/tracing/events/kvm/enable


### 统计KVM事件 ###

    [root@localhost win7]# perf stat -e 'kvm:*' -a sleep 1h
    ^Csleep: Interrupt

    Performance counter stats for 'sleep 1h':

      1,880 kvm:kvm_entry                                                [100.00%]
          0 kvm:kvm_hypercall                                            [100.00%]
          0 kvm:kvm_hv_hypercall                                      [100.00%]
         20 kvm:kvm_pio                                                     [100.00%]
          0 kvm:kvm_cpuid                                                  [100.00%]
        596 kvm:kvm_apic                                                  [100.00%]
      1,934 kvm:kvm_exit                                                  [100.00%]
        284 kvm:kvm_inj_virq                                             [100.00%]
          3 kvm:kvm_inj_exception                                     [100.00%]
        602 kvm:kvm_page_fault                                       [100.00%]
          0 kvm:kvm_msr                                                      [100.00%]
        102 kvm:kvm_cr                                                       [100.00%]
        260 kvm:kvm_pic_set_irq                                      [100.00%]
        156 kvm:kvm_apic_ipi                                            [100.00%]
        292 kvm:kvm_apic_accept_irq                              [100.00%]
        292 kvm:kvm_eoi                                                     [100.00%]
          0 kvm:kvm_pv_eoi                                                 [100.00%]
          0 kvm:kvm_nested_vmrun                                   [100.00%]
          0 kvm:kvm_nested_intercepts                             [100.00%]
          0 kvm:kvm_nested_vmexit                                    [100.00%]
          0 kvm:kvm_nested_vmexit_inject                        [100.00%]
          0 kvm:kvm_nested_intr_vmexit                            [100.00%]
          0 kvm:kvm_invlpga                                                 [100.00%]
          0 kvm:kvm_skinit                                                    [100.00%]
        979 kvm:kvm_emulate_insn                                  [100.00%]
        618 kvm:vcpu_match_mmio                                  [100.00%]
        635 kvm:kvm_userspace_exit                               [100.00%]
        276 kvm:kvm_set_irq                                              [100.00%]
        276 kvm:kvm_ioapic_set_irq                                 [100.00%]
          4 kvm:kvm_msi_set_irq                                        [100.00%]
        277 kvm:kvm_ack_irq                                              [100.00%]
      1,627 kvm:kvm_mmio                                                [100.00%]
        762 kvm:kvm_fpu                                                      [100.00%]
          0 kvm:kvm_age_page                                            [100.00%]
          0 kvm:kvm_try_async_get_page                          [100.00%]
          0 kvm:kvm_async_pf_doublefault                        [100.00%]
          0 kvm:kvm_async_pf_not_present                      [100.00%]
          0 kvm:kvm_async_pf_ready                                  [100.00%]
          0 kvm:kvm_async_pf_completed                                   

    1.895712367 seconds time elapsed


### KVM性能数据记录与分析 (需要kernel > 3.7) ###

**记录性能数据**


    # ./perf kvm stat record -p 26071 -o perf.data
    ^C[ perf record: Woken up 9 times to write data ]
    [ perf record: Captured and wrote 24.903 MB perf.data.guest (~1088034 samples) ]
    


**分析VMEXIT事件**


    # ./perf kvm stat report --event=vmexit

    Analyze events for all VCPUs:

             VM-EXIT    Samples    Samples%     Time%    Avg time

           APIC_ACCESS      65381    66.58%     5.95%     37.72us ( +-   6.54% )
    EXTERNAL_INTERRUPT      16031    16.32%     3.06%     79.11us ( +-   7.34% )
                 CPUID       5360     5.46%     0.06%      4.50us ( +-  35.07% )
                   HLT       4496     4.58%    90.75%   8360.34us ( +-   5.22% )
         EPT_VIOLATION       2667     2.72%     0.04%      5.49us ( +-   5.05% )
     PENDING_INTERRUPT       2242     2.28%     0.03%      5.25us ( +-   2.96% )
         EXCEPTION_NMI       1332     1.36%     0.02%      6.53us ( +-   6.51% )
        IO_INSTRUCTION        383     0.39%     0.09%     93.39us ( +-  40.92% )
             CR_ACCESS        310     0.32%     0.00%      6.10us ( +-   3.95% )

    Total Samples:98202, Total events handled time:41419293.63us.



**分析MMIO事件**


    # ./perf kvm stat report --event=mmio

    Analyze events for all VCPUs:

         MMIO Access    Samples  Samples%     Time%         Avg time

        0xfee00380:W      58686    90.21%    15.67%      4.95us ( +-   2.96% )
        0xfee00300:R       2124     3.26%     1.48%     12.93us ( +-  14.75% )
        0xfee00310:W       2124     3.26%     0.34%      3.00us ( +-   1.33% )
        0xfee00300:W       2123     3.26%    82.50%    720.68us ( +-  10.24% )

    Total Samples:65057, Total events handled time:1854470.45us.


**分析IOPORT事件**


    # ./perf kvm stat report --event=ioport

    Analyze events for all VCPUs:

      IO Port Access    Samples  Samples%     Time%         Avg time

         0xc090:POUT        383   100.00%   100.00%     89.00us ( +-  42.94% )

    Total Samples:383, Total events handled time:34085.56us.


针对指定的vcpu分析事件，并按照时间来排序输出结果


    # ./perf kvm stat report --event=vmexit --vcpu=0 --key=time

    Analyze events for VCPU 0:

             VM-EXIT       Samples  Samples%    Time%    Avg time

                   HLT        551     5.05%    94.81%   9501.72us ( +-  12.52% )
    EXTERNAL_INTERRUPT       1390    12.74%     2.39%     94.80us ( +-  20.92% )
           APIC_ACCESS       6186    56.68%     2.62%     23.41us ( +-  23.62% )
        IO_INSTRUCTION         17     0.16%     0.01%     20.39us ( +-  22.33% )
         EXCEPTION_NMI         94     0.86%     0.01%      6.07us ( +-   7.13% )
     PENDING_INTERRUPT        199     1.82%     0.02%      5.48us ( +-   4.36% )
             CR_ACCESS         52     0.48%     0.00%      4.89us ( +-   4.09% )
         EPT_VIOLATION       2057    18.85%     0.12%      3.15us ( +-   1.33% )
                 CPUID        368     3.37%     0.02%      2.82us ( +-   2.79% )

    Total Samples:10914, Total events handled time:5521782.02us.
    
本文转自[这里](http://blog.chinaunix.net/uid-26000137-id-3761730.html)