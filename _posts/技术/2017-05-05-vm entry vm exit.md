---
layout: post
title: qemu-kvm中vcpu虚拟化到底是咋整的
category: 技术
tags: 虚拟化层
keywords: vmentry,vmexit,kvm
description: 
---

### 一句话总结 ###

实例化一个vcpu就是在hostOS中创建了一个线程  
线程里有个while循环，循环里不停的调用kvm_cpu_exec方法  
kvm_cpu_exec方法调用通过kvm_vcpu_ioctl(cpu, KVM_RUN, 0)使得kvm切换为no-root模式  
在no-root模式下处理特权指令的时候，会退回root模式  
然后一步步返回到kvm_cpu_exec中根据不同原因，处理返回异常  

如此一个轮回结束  
周而复始，vcpu


再多说一点  
内存中申请一块内存  
根模式和非根模式切换的时候  
先把当前寄存器值放到这块内存中，然后设置物理cpu使得进入对应模式  
这块内存叫vmcs


(这个一句话总结有点长，一口气说完估计能憋死，凑合着看吧)

### 背景 ###

vcpu初始化的时候（qemu_init_vcpu）是启动了一个线程，也就是说vcpu其实就是一个线程.  
线程运行方法是qemu_kvm_cpu_thread_fn  
![](http://i.imgur.com/MXynnpc.png)  
kvm_init_vcpu调用KVM_CREATE_VCPU创建了vcpu返回vm_fd  
vcpu的运行是在kvm_cpu_exec里面的，这里调用如下命令进入kvm  

    run_ret = kvm_vcpu_ioctl(cpu, KVM_RUN, 0);

进入KVM后，KVM会切入Guest OS，假如Guest OS运行运行，需要访问IO等  
也就是说要访问physical device，那么Qemu与KVM就要进行emulate。  
如果是KVM emulate的则由KVM emulate，然后切回Guest OS。  
如果是Qemu emulate的，则从KVM中进入Qemu，等Qemu中的device model执行完emulate之后，再次在Qemu中调用kvm_vcpu_ioctl(vcpu_fd, KVM_RUN, xxx)进入KVM运行，然后再切回Guest OS

### vm-entry ###

kvm_vcpu_ioctl（kvm_main.c）-->  
kvm_arch_vcpu_ioctl_run(kvm/x86.c)-->  
vcpu_run(kvm/x86.c)-->  
vcpu_enter_guest(kvm/x86.c)  
在qemu中kvm_vcpu_ioctl(cpu, KVM_RUN, 0)调用kvm后  
代码层层调用最终核心实现的方法是vcpu_enter_guest  

#### vcpu->requests 处理 ####

上次VM-Exit时可能调用kvm_make_request设置不同的request  
下次准备VM-Entry时需要处理这些request.  
![](http://i.imgur.com/6x4ZA5p.png)  
![](http://i.imgur.com/Jf0tSZ8.png)  

#### prepare_guest_switch ####  

     (vmx.c)
    .prepare_guest_switch = vmx_save_host_state,

1.保存host的fs和gs的断选择子（segment selector）到vmcs  

2. kvm_set_shared_msr设置host对应的msr寄存器  
   MSR 总体来是为了设置CPU 的工作环境和标示CPU 的工作状态，包括温度控制，性能监控等  
   guest的msr在handle_wrmsr  最终是在vmx_set_msr中更新  

3. 判断当前是否满足vm-entry  
   vcpu的mode不对，有requests请求，需要重新调度，有pending的信号  
   有异常任何情况，不进入vm_entry  
   开中断，开抢占（在此之前已经关抢占，关中断）  
![](http://i.imgur.com/dQcGaJ9.png)

4.kvm_x86_ops->run(vcpu)  
  -->vmx_vcpu_run(vmx.c)  
   更新vmcs中的GUEST_RSP和 GUEST_RIP  
   刷新vmcs中的HOST_CR4字段  
   （其他寄存器在 kvm_arch_vcpu_ioctl_set_regs时设置）  
   （段寄存器在vmx_vcpu_reset时设置）  
   下面调用汇编代码，保存host相关内容，然后加载vmcs中的guest的寄存器值，跳转至guest中代码  
   ![](http://i.imgur.com/KOXdBHQ.png)

### vm-exit ###

前半部分我们知道了如何vm-entry  
此时进入no-root非根模式执行guest的指令  
当指令访问特权指令如访问io访问设备的时候会vm-exit  

1.vmx_vcpu_run后半段  

    vmx->idt_vectoring_info = vmcs_read32(IDT_VECTORING_INFO_FIELD);
    vmx->loaded_vmcs->launched = 1;
    vmx->exit_reason = vmcs_read32(VM_EXIT_REASON);

2.加入vm-exit是由于EXIT_REASON_MCE_DURING_VMENTRY或者EXIT_REASON_EXCEPTION_NMI导致的  
  在vmx_complete_atomic_exit方法中需要进行特殊处理  
  （kvm_machine_check）（kvm_before_handle_nmi和kvm_after_handle_nmi） 
3. 如果有事件模拟的virtual nmi中断，则用vmx_recover_nmi_blocking处理  
4. 获取与预处理导致的中断由vmx_complete_interrupts-->__vmx_complete_interrupts处理

----------
（至此退出vmx_vcpu_run重返vmx_vcpu_run）

kvm_x86_ops->handle_exit
-->vmx_handle_exit
根据不同情况处理异常

至此从kvm中返回到用户态qemu中kvm_cpu_exec方法  
![](http://i.imgur.com/l7ZC2wr.png)  
根据不同退出原因，处理异常  
然后退出到线程方法qemu_kvm_cpu_thread_fn  
继续执行下一次循环  
![](http://i.imgur.com/FVFomPl.png)
