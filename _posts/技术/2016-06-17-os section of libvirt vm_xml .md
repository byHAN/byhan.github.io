---
layout: post
title: os section of libvirt vm_xml 
category: nova
tags: 虚拟化层
keywords: 
description: 
---

## 背景知识  ##


## 一句话总结 ##

## libvirt ##

今天在看书的时候看到libvirt的虚拟机XML中有os这一项  
让我们一探究竟  
![](http://i.imgur.com/EHt3nOb.png)  
这部分内容主要由两部分组成  

### 系统类型-type  ###  

type规定了guestOS的类型，这里是hvm(hardware virtual machine)  
代表是可以直接运行在物理设备上的os  
这种os不用针对虚拟化做修改，也就是原生的os，不感知自己是否是虚拟化的  
底层的虚拟化需要时全虚拟化（full virtualization）
与之相对的xen上支持的os可能需要对os做一定的修改  

arch指定了当前虚拟机的cpu系统结构，这里是x86_64，代表模拟的是64位的x86架构  
mainchine指定了

    <guest>
      <os_type>hvm</os_type>
      <arch name='x86_64'>
        <wordsize>64</wordsize>
        <emulator>/usr/bin/qemu-system-x86_64</emulator>
        <machine canonical='pc-i440fx-utopic' maxCpus='255'>ubuntu</machine>
        <machine maxCpus='255'>pc-1.3</machine>
        <machine maxCpus='255'>pc-0.12</machine>
        <machine maxCpus='255'>pc-q35-1.6</machine>
        <machine maxCpus='255'>pc-q35-1.5</machine>
        <machine maxCpus='255'>pc-i440fx-1.6</machine>
        <machine canonical='pc-q35-2.2' maxCpus='255'>q35</machine>
        <machine maxCpus='255'>pc-i440fx-1.7</machine>
        <machine maxCpus='1'>xenpv</machine>
        <machine maxCpus='255'>pc-q35-2.1</machine>
        <machine maxCpus='255'>pc-0.11</machine>
        <machine maxCpus='255'>pc-0.10</machine>
        <machine canonical='pc-i440fx-2.2' maxCpus='255'>pc</machine>
        <machine maxCpus='255'>pc-1.2</machine>
        <machine maxCpus='1'>isapc</machine>
        <machine maxCpus='255'>pc-i440fx-trusty</machine>
        <machine maxCpus='255'>pc-q35-1.4</machine>
        <machine maxCpus='128'>xenfv</machine>
        <machine maxCpus='255'>pc-0.15</machine>
        <machine maxCpus='255'>pc-i440fx-1.5</machine>
        <machine maxCpus='255'>pc-i440fx-1.4</machine>
        <machine maxCpus='255'>pc-q35-2.0</machine>
        <machine maxCpus='255'>pc-0.14</machine>
        <machine maxCpus='255'>pc-1.1</machine>
        <machine maxCpus='255'>pc-q35-1.7</machine>
        <machine maxCpus='255'>pc-i440fx-2.1</machine>
        <machine maxCpus='255'>pc-1.0</machine>
        <machine maxCpus='255'>pc-i440fx-2.0</machine>
        <machine maxCpus='255'>pc-0.13</machine>
        <domain type='qemu'/>
        <domain type='kvm'>
          <emulator>/usr/bin/qemu-system-x86_64</emulator>
        </domain>
      </arch>
      <features>
        <cpuselection/>
        <deviceboot/>
        <disksnapshot default='on' toggle='no'/>
        <acpi default='on' toggle='yes'/>
        <apic default='on' toggle='no'/>
      </features>
    </guest>


### 启动方式-boot  ###

## qemu ##

![](http://i.imgur.com/OpDkoC6.png)
QEMU 模拟的是 INTEL 的 i440fx 硬件芯片组
QEMU在初始化硬件的时候，最开始的函数就是pc_init1。  
在这个函数里面会相继的初始化CPU，中断控制器，ISA总线，然后就要判断是否需要支持PCI  
如果支持则调用i440fx_init初始化我们伟大的PCI总线。  

i440fx_init函数主要参数就是之前初始化好的ISA总线以及中断控制器，返回值就是PCI总线  
之后我们就可以将我们自己喜欢的设备统统挂载在这个上面

### 参考文献 ###

http://libvirt.org/formatdomain.html
http://www.ibm.com/developerworks/cn/linux/1410_qiaoly_qemubios/
http://blog.csdn.net/yearn520/article/details/6576875