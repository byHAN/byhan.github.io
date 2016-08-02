---
layout: post
title: USB 安装VMWARE ESX
category: 技术
tags: 虚拟化层
keywords: 
description: 
---

今天用U盘在服务器上安装了ESX，将相关情况记录如下：

首先用UltraISO将对应的ISO刻录到U盘，但是由于么有引导程序报Initial menu has no LABEL entries
最后按照官网指导安装成功。


详细参见[官方文档](http://pubs.vmware.com/vsphere-60/index.jsp#com.vmware.vsphere.install.doc/GUID-33C3E7D5-20D0-4F84-B2E3-5CD33D32EAA8.html)，本节主要摘取自官网

在操作系统可检测到 USB 闪存驱动器的 Linux 计算机上执行该过程。在示例中，操作系统将 USB 闪存驱动器检测为 /dev/sdb。

注：包含安装脚本的 ks 文件不能位于引导安装或升级所使用的同一个 USB 闪存驱动器上。

# 先决条件 #

- ESXi ISO 映像 VMware-VMvisor-Installer-6.x.x-XXXXXX.x86_64.iso，包括 isolinux.cfg 文件，其中 6.x.x 表示要安装的 ESXi 的版本，XXXXXX 表示安装程序 ISO 映像的内部版本号
- 可访问 Syslinux 版本 3.86 或 4.03 的 Linux 计算机 ，这里使用virtualBox虚拟出来的一台Ubuntu14

# 步骤 #

1. 如果您的 USB 闪存驱动器未检测为 /dev/sdb，或者您不确定 USB 闪存驱动器是如何检测到的，请确定该闪存驱动器的检测方式。
	1. 在命令行中，运行以下命令。  
       tail -f /var/log/messages  
       该命令将显示当前日志消息。ubuntu中默认关闭，需要打开，参考这里
	2. 插入 USB 闪存驱动器。  
       可以提到以类似如下消息的格式显示标识 USB 闪存驱动器的若干条消息。
       Oct 25 13:25:23 ubuntu kernel:[ 712.447080] sd 3:0:0:0:[sdb] Attached SCSI removable disk  
       在此示例中，sdb 用于标识 USB 设备。如果您设备的标识方式与此不同，请使用该标识替换 sdb。

2. 在 USB 闪存驱动器上创建分区表。
  /sbin/fdisk /dev/sdb
- 键入 d 删除分区，直至将其全部删除。
- 键入 n 创建遍及整个磁盘的主分区 1。
- 键入 t 将 FAT32 文件系统的类型设置为适当的设置，如 c。
- 键入 a 在分区 1 上设置活动标记。
- 键入 p 打印分区表。
- 结果应类似于以下文本：
- Disk /dev/sdb:2004 MB, 2004877312 bytes
- 255 heads, 63 sectors/track, 243 cylinders
- Units = cylinders of 16065 * 512 = 8225280 bytes
- Device Boot Start End Blocks Id System
- /dev/sdb1 1 243 1951866 c W95 FAT32 (LBA)
- 键入 w 写入分区表并退出程序。
- 本人在虚拟机中，这里修改的分区表不被内核感知，重启系统解决

3. 使用 Fat32 文件系统格式化 USB 闪存驱动器。   `/sbin/mkfs.vfat -F 32 -n USB /dev/sdb1`
4. 运行下列命令。`/path_to_syslinux-version_directory/syslinux-version/bin/syslinux /dev/sdb1 cat /path_to_syslinux-version_directory/syslinux-version/usr/share/syslinux/mbr.bin > /dev/sdb`
5. 挂载 USB 闪存驱动器。    `mount /dev/sdb1 /usbdisk`
6. 挂载 ESXi 安装程序 ISO 映像。    `mount -o loop VMware-VMvisor-Installer-6.x.x-XXXXXX.x86_64.iso /esxi_cdrom`
7. 将 ISO 映像的内容复制到 /usbdisk。 `cp -r /esxi_cdrom/* /usbdisk`
8. 将 isolinux.cfg 文件重命名为 syslinux.cfg。    `mv /usbdisk/isolinux.cfg /usbdisk/syslinux.cfg`
9. 在 /usbdisk/syslinux.cfg 文件中，将 APPEND -c boot.cfg 一行更改为 APPEND -c boot.cfg -p 1。
10. 如果使用 Syslinux 版本 4.03，请替换 menu.c32。`cp / path_to_syslinux directory/syslinux-4.03/usr/share/syslinux/menu.c32 /usbdisk/`
11. 卸载 USB 闪存驱动器。    `umount /usbdisk`
12. 卸载安装程序 ISO 映像。    `umount /esxi_cdrom`
13. SB 闪存驱动器可以引导 ESXi 安装程序。