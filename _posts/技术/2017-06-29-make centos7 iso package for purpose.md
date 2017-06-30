---
layout: post
title: CentOS7定制封装发行版
category: 技术
tags: 虚拟化
keywords: iso,centos7
description: 
---

实际工作中，CentOS的安装需要设置的语言、键盘模式、时区等信息都存在很大程度上的雷同型。  
安装完成后的一些设置工作也都是一样的。  
这些工作都可以在安装操作系统的时候自动完成。
最终实现，安装完成即可得到一个可以使用的CentOS系统。

打包环境
----

我们的打包环境是在一个纯净的CentOS7-Minimal上进行的，安装完成CentOS7之后我们进行以下操作安装打包环境：

    yum install wget anaconda repodata createrepo mkisofs yum-plugin-downloadonly -y

将CentOS7的minimal版本下载下来或scp存储至/root/目录下，下载命令：

    wget http://mirrors.ustc.edu.cn/centos/7.0.1406/isos/x86_64/CentOS-7.0-1406-x86_64-Minimal.iso

我们将开发环境设置在/opt目录下，首先创建/opt/centosori目录用于挂载原始的ISO：

    mkdir /opt/centosori
    mount /root/CentOS-7.0-1406-x86_64-Minimal.iso /opt/centosori

复制原始ISO至要更改的环境中：

    cp -r /opt/centosori /opt/centosnew

此时/opt/centosnew的目录如下所示：

    [root@localhost centosnew]# ll -h 
    总用量 68K 
    -rw-r--r--. 1 root root 14 12月 18 10:59 CentOS_BuildTag drwxr-xr-x. 3 root root 33 12月 18 10:59 EFI 
    -rw-r--r--. 1 root root 214 12月 18 10:59 EULA -rw-r--r--. 1 root root 18K 12月 18 10:59 GPL
    drwxr-xr-x. 3 root root 54 12月 18 10:59 images
    drwxr-xr-x. 2 root root 4.0K 12月 18 11:01 isolinux
    drwxr-xr-x. 2 root root 41 12月 18 10:59 LiveOS 
    drwxr-xr-x. 2 root root 16K 12月 18 10:59 Packages
    drwxr-xr-x. 2 root root 4.0K 12月 18 10:59 repodata 
    -rw-r--r--. 1 root root 1.7K 12月 18 10:59 RPM-GPG-KEY-CentOS-7 
    -rw-r--r--. 1 root root 1.7K 12月 18 10:59 RPM-GPG-KEY-CentOS-Testing-7 
    -r--r--r--. 1 root root 2.9K 12月 18 10:59 TRANS.TBL


定义自动应答脚本
--------

我们在此只做一个增加base组的yum安装包，增加到minimal的ISO中供大家参考  
首先下载base的yum包（此处制作ISO的系统一定是要新安装的，否则会有包下载不全的情况出现）：  

    yum -y --downloadonly --downloaddir=/opt/centosnew/Packages groupinstall base

然后将系统安装时的自动应答文件拷贝至/opt/centosnew/isolinux/文件夹下：

    cp /root/anaconda-ks.cfg /opt/centosnew/isolinux/base-ks.cfg

编辑base-ks.cfg文件：

#version=RHEL7 
# System authorization information auth --enableshadow --passalgo=sha512 
# Use CDROM installation media cdrom 
# Run the Setup Agent on first boot firstboot 
--enable ignoredisk --only-use=sda 
# Keyboard layouts 
keyboard --vckeymap=us --xlayouts='us' 
# System language 
lang en_US.UTF-8 
# Network information 
network --bootproto=dhcp --device=eth0 --ipv6=auto --activate
network --hostname=localhost.localdomain 
# Root password 
rootpw --iscrypted $6$evKW0uI7XvLXV0dG$ELbcO4R6c9qq.Qd5m0iJPDJsZM9wOWq9jlbL7ztsgZxaly8FcWX5lAEQULr8EF1uevenUAmHrhQbOYsEVjiE90 
# System timezone 
timezone Asia/Shanghai --isUtc --nontp
user --groups=wheel --name=cloud --password=$6$aOxLlKnnDm4ol9yv$xQmkbHctQDkvtkQMubZRkJVAIifoYThaxvO1.fn1zkDZnqHaIYtm8ypfshkUBmT6VEYpO/5kqM73JUGMtkYFo0 --iscrypted --gecos="cloud" 
# System bootloader configuration 
bootloader --location=mbr --boot-drive=sda
autopart --type=lvm 
# Partition clearing information 
clearpart --none --initlabel 
%packages @core @base ＃添加base group包 
%end

此时已经完成base包的添加，在package的部分增加@base的部分,用户如果想添加其他的rpm包可以将rpm包放到Package中，然后在ks文件中添加对应的包名即可，非group安装的时候不用在前边加@修饰符，下面是％package的一个例子：

%packages @base @core @console-internet 
tree 
policycoreutils-python %end

在ks文件中使用的默认密码是zaq12wsx，修改方式如下：

openssl passwd -1

输入并验证自己的密码，替换到ks文件中即可

# Root password rootpw --iscrypted $6$evKW0uI7XvLXV0dG$ELbcO4R6c9qq.Qd5m0iJPDJsZM9wOWq9jlbL7ztsgZxaly8FcWX5lAEQULr8EF1uevenUAmHrhQbOYsEVjiE90

增加base的group list到资源库的XML中

由于minimal的安装中没有base的组安装信息，我们在repo的xml中增加base的组信息

修改/opt/centosnew/repodata/中以xml结尾的文件，在core的group结束标签</group>后增加如下内容：

  <group> <id>base</id> <name>Base</name> <default>false</default> <uservisible>false</uservisible> <packagelist> <packagereq requires="ruby" type="conditional">rubygem-abrt</packagereq> <packagereq type="default">abrt-addon-ccpp</packagereq> <packagereq type="default">abrt-addon-python</packagereq> <packagereq type="default">abrt-cli</packagereq> <packagereq type="default">abrt-console-notification</packagereq> <packagereq type="default">bash-completion</packagereq> <packagereq type="default">blktrace</packagereq> <packagereq type="default">bridge-utils</packagereq> <packagereq type="default">bzip2</packagereq> <packagereq type="default">chrony</packagereq> <packagereq type="default">cryptsetup</packagereq> <packagereq type="default">dmraid</packagereq> <packagereq type="default">dosfstools</packagereq> <packagereq type="default">ethtool</packagereq> <packagereq type="default">fprintd-pam</packagereq> <packagereq type="default">gnupg2</packagereq> <packagereq type="default">hunspell-en</packagereq> <packagereq type="default">hunspell</packagereq> <packagereq type="default">kpatch</packagereq> <packagereq type="default">ledmon</packagereq> <packagereq type="default">libaio</packagereq> <packagereq type="default">libreport-plugin-mailx</packagereq> <packagereq type="default">libstoragemgmt</packagereq> <packagereq type="default">lvm2</packagereq> <packagereq type="default">man-pages-overrides</packagereq> <packagereq type="default">man-pages</packagereq> <packagereq type="default">mdadm</packagereq> <packagereq type="default">mlocate</packagereq> <packagereq type="default">mtr</packagereq> <packagereq type="default">nano</packagereq> <packagereq type="default">ntpdate</packagereq> <packagereq type="default">pinfo</packagereq> <packagereq type="default">plymouth</packagereq> <packagereq type="default">pm-utils</packagereq> <packagereq type="default">rdate</packagereq> <packagereq type="default">rfkill</packagereq> <packagereq type="default">rng-tools</packagereq> <packagereq type="default">rsync</packagereq> <packagereq type="default">scl-utils</packagereq> <packagereq type="default">setuptool</packagereq> <packagereq type="default">smartmontools</packagereq> <packagereq type="default">sos</packagereq> <packagereq type="default">sssd-client</packagereq> <packagereq type="default">strace</packagereq> <packagereq type="default">sysstat</packagereq> <packagereq type="default">systemtap-runtime</packagereq> <packagereq type="default">tcpdump</packagereq> <packagereq type="default">tcsh</packagereq> <packagereq type="default">teamd</packagereq> <packagereq type="default">time</packagereq> <packagereq type="default">unzip</packagereq> <packagereq type="default">usbutils</packagereq> <packagereq type="default">vim-enhanced</packagereq> <packagereq type="default">virt-what</packagereq> <packagereq type="default">wget</packagereq> <packagereq type="default">which</packagereq> <packagereq type="default">words</packagereq> <packagereq type="default">xfsdump</packagereq> <packagereq type="default">xz</packagereq> <packagereq type="default">yum-langpacks</packagereq> <packagereq type="default">yum-plugin-security</packagereq> <packagereq type="default">yum-utils</packagereq> <packagereq type="default">zip</packagereq> <packagereq type="mandatory">acl</packagereq> <packagereq type="mandatory">at</packagereq> <packagereq type="mandatory">attr</packagereq> <packagereq type="mandatory">authconfig</packagereq> <packagereq type="mandatory">bc</packagereq> <packagereq type="mandatory">bind-utils</packagereq> <packagereq type="mandatory">cpio</packagereq> <packagereq type="mandatory">crda</packagereq> <packagereq type="mandatory">crontabs</packagereq> <packagereq type="mandatory">cyrus-sasl-plain</packagereq> <packagereq type="mandatory">dbus</packagereq> <packagereq type="mandatory">ed</packagereq> <packagereq type="mandatory">file</packagereq> <packagereq type="mandatory">firewalld</packagereq> <packagereq type="mandatory">logrotate</packagereq> <packagereq type="mandatory">lsof</packagereq> <packagereq type="mandatory">man-db</packagereq> <packagereq type="mandatory">net-tools</packagereq> <packagereq type="mandatory">ntsysv</packagereq> <packagereq type="mandatory">pciutils</packagereq> <packagereq type="mandatory">psacct</packagereq> <packagereq type="mandatory">quota</packagereq> <packagereq type="mandatory">centos-indexhtml</packagereq> <packagereq type="mandatory">setserial</packagereq> <packagereq type="mandatory">traceroute</packagereq> <packagereq type="mandatory">usb_modeswitch</packagereq> <packagereq type="optional">acpid</packagereq> <packagereq type="optional">audispd-plugins</packagereq> <packagereq type="optional">brltty</packagereq> <packagereq type="optional">cryptsetup-reencrypt</packagereq> <packagereq type="optional">device-mapper-persistent-data</packagereq> <packagereq type="optional">dos2unix</packagereq> <packagereq type="optional">dumpet</packagereq> <packagereq type="optional">genisoimage</packagereq> <packagereq type="optional">gpm</packagereq> <packagereq type="optional">i2c-tools</packagereq> <packagereq type="optional">kabi-yum-plugins</packagereq> <packagereq type="optional">libatomic</packagereq> <packagereq type="optional">libcgroup</packagereq> <packagereq type="optional">libcgroup-tools</packagereq> <packagereq type="optional">libitm</packagereq> <packagereq type="optional">libstoragemgmt-netapp-plugin</packagereq> <packagereq type="optional">libstoragemgmt-nstor-plugin</packagereq> <packagereq type="optional">libstoragemgmt-smis-plugin</packagereq> <packagereq type="optional">libstoragemgmt-targetd-plugin</packagereq> <packagereq type="optional">libstoragemgmt-udev</packagereq> <packagereq type="optional">linuxptp</packagereq> <packagereq type="optional">logwatch</packagereq> <packagereq type="optional">mkbootdisk</packagereq> <packagereq type="optional">mtools</packagereq> <packagereq type="optional">ncurses-term</packagereq> <packagereq type="optional">ntp</packagereq> <packagereq type="optional">oddjob</packagereq> <packagereq type="optional">pax</packagereq> <packagereq type="optional">prelink</packagereq> <packagereq type="optional">PyPAM</packagereq> <packagereq type="optional">python-volume_key</packagereq> <packagereq type="optional">redhat-lsb-core</packagereq> <packagereq type="optional">redhat-upgrade-dracut</packagereq> <packagereq type="optional">redhat-upgrade-tool</packagereq> <packagereq type="optional">rsyslog-gnutls</packagereq> <packagereq type="optional">rsyslog-gssapi</packagereq> <packagereq type="optional">rsyslog-relp</packagereq> <packagereq type="optional">sgpio</packagereq> <packagereq type="optional">sox</packagereq> <packagereq type="optional">squashfs-tools</packagereq> <packagereq type="optional">star</packagereq> <packagereq type="optional">tmpwatch</packagereq> <packagereq type="optional">udftools</packagereq> <packagereq type="optional">uuidd</packagereq> <packagereq type="optional">volume_key</packagereq> <packagereq type="optional">wodim</packagereq> <packagereq type="optional">x86info</packagereq> <packagereq type="optional">yum-plugin-aliases</packagereq> <packagereq type="optional">yum-plugin-changelog</packagereq> <packagereq type="optional">yum-plugin-tmprepo</packagereq> <packagereq type="optional">yum-plugin-verify</packagereq> <packagereq type="optional">yum-plugin-versionlock</packagereq> <packagereq type="optional">zsh</packagereq> </packagelist> </group>

并在grouplist中添加base：

 <grouplist> <groupid>core</groupid> <groupid>base</groupid> </grouplist>

修改ISOLINUX启动文件

修改/opt/centosnew/isolinux中的isolinux.cfg文件，找到label linux部分，并修改如下所示：

label linux
 menu label ^Install CentOS 7 kernel vmlinuz
 append initrd=initrd.img ks=cdrom:/isolinux/base-ks.cfg

label check
 menu label Test this ^media &amp; install CentOS 7 menu default kernel vmlinuz
 append initrd=initrd.img

修改包的资源库并打包ISO

我使用了一个脚本来自动修改包的资源库并打包为ISO，在/opt下创建buildiso.sh脚本并赋予可执行权力：

touch /opt/buildiso.sh
chmod +x /opt/buildiso.sh

修改buildiso.sh 的内容：

#!/bin/bash 
cd /opt/centosnew/repodata
mv *-comps.xml comps.xml
ls .|grep -v "comps.xml"|xargs -i rm -f {} 
cd ../ createrepo -g repodata/comps.xml ./ declare -x discinfo=`head -1 .discinfo` mkisofs -R -J -T -r -l -d -joliet-long -allow-multidot -allow-leading-dots -no