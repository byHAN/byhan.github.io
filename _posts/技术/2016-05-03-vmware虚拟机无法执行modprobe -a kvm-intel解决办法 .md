---
layout: post
title:vmware虚拟机无法执行modprobe -a kvm-intel解决办法
category: 技术
tags: openstack
keywords: 
description: 
---

本人在服务上安装了vmware ESX，然后其中创建了几台虚拟机，安装了openstack的Liberty版本。
但是无法使用kvm，按照官方指导设置kvm的时候报错。

![](http://i.imgur.com/8J76eO2.png)

从字面意思可以查看是无法加载kvm_intel模块，由于服务器本人确认没有问题。
所以问题只可能出现在vmware虚拟机上。

#### 打开SSH ####

默认ESX的ssh 是关闭的，可以在troubleshooting里面打开，具体操作谷歌一把。
修改VMware虚拟机

主要参考这里，为避免链接失效，现摘录如下：（需要在vmware的虚拟机磁盘文件中追加vhv.enable = "TRUE"）

    Enable vmx/svm flags for the Guest/Nested hypervisor in Vmware ESXi5.5 from CLI
        The official guide - https://communities.vmware.com/docs/DOC-8970 - for enabling hardware assisted virtualization (VMX/SVM CPU flags) for the guest hypervisor is to use the web client. For those of us using this setup just for testing and using the free vsphere client, this method is not compatible.
        So how can vmware ESXi 5.5 be instructed to provide the CPU virtualization vmx flag to the guest hypervisor without web client ?
        There are two workarounds: one from the regular vsphere client and one from the ESXi 5.5 shell. Steps:
             Shutdown the nested/guest hypervisor (Ubuntu KVM in my case)
             Locate the guest hypervisor virtual machine configuration file (<VM-name.vmx>), edit and add the following line at the end:
            vhv.enable = "TRUE"
            Save and close the file.
             Identify the nested hypervisor vm ID and reload it's configuration with the [b]vim-cmd esxi command:[/b]
            ~ # vim-cmd vmsvc/getallvms | grep -i ubun
            44     VM6-Ubuntu-KVM        [datastore1] VM6-Ubuntu-KVM/VM6-Ubuntu-KVM.vmx             ubuntu64Guest      vmx-08
            ~ # vim-cmd vmsvc/reload 44
             Start the nested VM
            Verify that the nested hypervisor correctly detects the vmx flag
            Once the virtual machine started, login and check the /proc/cpuinfo file:
            $sudo grep -c vmx /proc/cpuinfo
            4
至此执行modprobe -a kvm-intel可以无误了。


