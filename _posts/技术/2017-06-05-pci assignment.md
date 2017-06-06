---
layout: post
title: 设备直通（PCI Assignment）到底是咋整的
category: 技术
tags: 虚拟化
keywords: 
description: virtio
---

## 背景 ##

IO虚拟化的三板斧：  
1.全模拟；2.virtio驱动半虚拟化；3.设备直通  

本文主要聚焦设备直通，重点阐述原理  
至于上述三种的优缺点，如何直通等入门级资料请自行谷歌  

本人[这篇博文](http://www.hanbaoying.com/2017/02/21/pci-passthrough-in-centos.html)中讲述了如何进行pci网卡的直通  
[这篇博文](http://www.hanbaoying.com/2017/03/01/pci-ASSIGNMENT-problem-solve.html)讲述了直通过程中遇到的问题，及其解决思路  

## 一句话总结 ##

将设备从主机的总线上拿下来，挂到模拟出来的虚拟机总线上去。  
这其中需要解决两个问题：
1.数据传输中，即dma时，设备如何直接访问到guestos内的地址  
2.中断过程中，设备的中断如何直达guestos  

以上是VT_到解决的主要两个问题，它通过北桥（现在叫MCH）中内置的硬件借助于iommu，dmar及其重构msi结构实现了DMA虚拟化和中断虚拟化。  

（VT-d interrupt-remapping重新定义MSI的格式，它依旧是一个DMA写请求的形式，不过其中不是目标内存地址，而是一个消息ID，通过维护一个表结构，硬件可以通过消息ID区分不同的虚拟机）  

## 原理 ##

设备直通可以通过kvm和vfio两种方式  
kvm方式是qemu调用kvm实现设备的直通，是一种传统方式  
vfio是将内核态的iommu通过接口暴露给用户态，实现的设备直通，qemu调用vfio，vfio再调用iommu等实现直通，具体vfio[参考这里](https://www.linux-kvm.org/images/b/b4/2012-forum-VFIO.pdf)  



### kvm ### 

#### 创建 ####

**1. 用户态**  

assign_class_init  
-->assigned_realize  
-->get_real_device(根据指定的设备信息，从/sys/bus/pci/devices获取到所需的信息)  
-->assigned_dev_register_regions(EPt建立好后，guest os访问gpa时就直接访问真实设备了不会有vm-exit发生)
-->assign_device-->kvm_device_pci_assign-->kvm_vm_ioctl(s, KVM_ASSIGN_PCI_DEVICE, &dev_data)  
-->assign_intx-->kvm_device_msix_deassign(以msix为例)-->kvm_deassign_irq_internal-->kvm_vm_ioctl(s, KVM_DEASSIGN_DEV_IRQ, &assigned_irq);

**2.内核态**

kvm_vm_ioctl_assigned_device  
-->kvm_vm_ioctl_assign_device  
-->kvm_iommu_map_guest-->iommu_domain_alloc(如果没有分配，为设备分配iommu_domain)  
-->kvm_iommu_map_guest-->kvm_iommu_map_memslots-->kvm_iommu_map_pages(检查是否建立映射，用iommu_map建立映射)
-->kvm_assign_device-->iommu_attach_device(将设备关联到iommu)  

----------

kvm_vm_ioctl_assigned_device  
-->kvm_vm_ioctl_assign_irq  

----------

-->assign_host_irq  
-->assigned_device_enable_host_msix(这里只分享msix)  
-->注册了中断处理函数kvm_assigned_dev_msix和中断处理线程kvm_assigned_dev_thread_msix

----------

-->assign_guest_irq  
-->assigned_device_enable_guest_msix  


**中断管理**  

kvm_assigned_dev_raise_guest_irq  
-->kvm_set_irq  
-->kvm_irq_map_gsi  

kvm_assigned_dev_msix  
-->kvm_set_irq_inatomic  
-->kvm_arch_set_irq_inatomic  
-->kvm_irq_delivery_to_apic_fast  

所以当中断发生的时候，发中断请求到apic  

### vfio ###

![](http://i.imgur.com/sCbGGxn.png)  


    -device vfio-pci,host=01:10.0,id=net0

#### qemu ####

vfio_initfn  
-->
1. 验证主机上设备是否存在于/sys/bus/pci/devices/  
2. 获取到设备的groupid  
3. vfio_get_group根据groupid获取到group  
   假设已经设置过此group下的设备，则直接返回缓存的group信息  否则，创建/dev/vfio/groupid并且通过VFIO_GROUP_GET_STATUS验证当前这个group是否可行  
   vfio_connect_container-->VFIO_GROUP_SET_CONTAINER和VFIO_SET_IOMMU设置group和iommu的关联关系  
   KVM_CREATE_DEVICE创建一个设备  
   KVM_SET_DEVICE_ATTR设置设备的group属性为group->fd  
4. 检查设备是否已经设置过直通  
5. vfio_get_device  
   通过VFIO_GROUP_GET_DEVICE_FD获取到设备的fd  
   通过VFIO_DEVICE_GET_INFO获取到设备信息  
6. vfio_populate_device补齐设备相关信息  
7. 设置pci的配置空间(config)  
8. vfio_pci_size_rom设置rom相关   
9. vfio_map_bars设置内存相关   
10. vfio_add_capabilities-->vfio_add_std_cap-->msix_init假设这里是msix，初始化msix内容  
11. 假如PCI_INTERRUPT_PIN则注册vfio_update_irq作为回调函数  
12. vfio_enable_intx-->VFIO_DEVICE_SET_IRQS
13. vfio_enable_intx-->vfio_enable_intx_kvm-->VFIO_DEVICE_SET_IRQS

#### 内核 ####

1. VFIO_GROUP_SET_CONTAINER  

vfio_group_set_container  
-->attach_group(假设我们这里是vfio_iommu_type1)  
-->vfio_iommu_type1_attach_group-->vfio_bus_type  
-->vfio_iommu_type1_attach_group-->iommu_domain_alloc  
-->vfio_iommu_type1_attach_group-->iommu_attach_group-->__iommu_attach_group-->__iommu_group_for_each_dev-->iommu_group_do_attach_device-->__iommu_attach_device-->intel_iommu_attach_device

2. VFIO_SET_IOMMU  
vfio_ioctl_set_iommu  
-->__vfio_container_attach_groups  
-->attach_group  

3. VFIO_GROUP_GET_DEVICE_FD  
vfio_group_get_device_fd  


4.VFIO_GROUP_GET_STATUS  
vfio_group_viable  
-->vfio_dev_viable  

5. KVM_CREATE_DEVICE  
kvm_ioctl_create_device  

6. KVM_SET_DEVICE_ATTR  
kvm_device_ioctl_attr  
-->kvm_vfio_set_attr  
-->kvm_vfio_set_group

7. VFIO_DEVICE_GET_INFO  

8. VFIO_DEVICE_SET_IRQS
vfio_pci_set_irqs_ioctl