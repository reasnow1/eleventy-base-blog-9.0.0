---
title: 搭建出租房网络 Mobile network setup
description: This is a post on My Blog about agile frameworks.
date: 2026-04-14
tags: OpenWrt
---

材料：
- 中兴F50
- 旧笔记本电脑（型号：万利达 酷奔 K1）
- 电信光猫
- 网线若干

工具：
- U盘
- USB键盘（笔记本键盘无用）
- SD卡，读卡器
- 因为这电脑键盘失灵才这样
- 如果不失灵，则2个U盘

# 开始：固件准备

下载新版OpenWrt：

[https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/targets/x86/generic/openwrt-imagebuilder-25.12.2-x86-generic.Linux-x86_64.tar.zst](https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/targets/x86/generic/openwrt-imagebuilder-25.12.2-x86-generic.Linux-x86_64.tar.zst)

注：这台笔记本只能32位，所以是generic，64位用x64

解压后修改其中的repositories：

```config
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/targets/x86/generic/packages/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/packages/i386_pentium4/base/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/targets/x86/generic/kmods/6.12.74-1-a55ea23bfb68f7c61df57b238ec32836/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/packages/i386_pentium4/luci/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/packages/i386_pentium4/packages/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/packages/i386_pentium4/routing/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/packages/i386_pentium4/telephony/packages.adb
https://mirror.nyist.edu.cn/openwrt/releases/25.12.2/packages/i386_pentium4/video/packages.adb
```
为了让固件出来，默认地址也是这些，可以
```bash
mkdir ./etc
mkdir ./etc/apk
nano ./etc/apk/distfeeds.conf
```
然后复制刚才的内容。

如果没弄，可以后面用vim进去改，按d然后大写G可以删除全部原内容，再补上镜像地址。

然后，让体积大一些，改：
```bash
nano .config
```
找到
```config
#
# Image Options
#
CONFIG_TARGET_KERNEL_PARTSIZE=16
CONFIG_TARGET_ROOTFS_PARTSIZE=104
CONFIG_TARGET_ROOTFS_PARTNAME=""
# CONFIG_TARGET_ROOTFS_PERSIST_VAR is not set
# end of Target Images
```
修改，我这边是改成
```config
#
# Image Options
#
CONFIG_TARGET_KERNEL_PARTSIZE=128
CONFIG_TARGET_ROOTFS_PARTSIZE=512
CONFIG_TARGET_ROOTFS_PARTNAME=""
# CONFIG_TARGET_ROOTFS_PERSIST_VAR is not set
# end of Target Images
```
官方的安装命令：
```bash
@debian:~/Downloads/openwrt-imagebuilder-25.12.2-x86-generic.Linux-x86_64$ sudo apt install build-essential file libncurses-dev zlib1g-dev gawk git \
gettext libssl-dev xsltproc rsync wget unzip python3 python3-distutils
Package python3-distutils is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or
is only available from another source

Error: Package 'python3-distutils' has no installation candidate
```
修改最后一项
```bash
sudo apt install build-essential file libncurses-dev zlib1g-dev gawk git gettext libssl-dev xsltproc rsync wget unzip python3 python3-setuptools
```
成功运行。

构建命令：
```bash
make image \
PACKAGES="luci openssh-sftp-server kmod-usb-net kmod-usb-net-cdc-ether" \
FILES="files"
```
注：
- luci：官方巨坑，默认不带，直接无
- openssh-sftp-server：传文件使用
- kmod-usb-net kmod-usb-net-cdc-ether用于中兴F50驱动
- FILES="files"用于将刚才的源修改文件注入

中途，会出现：
```bash
warning: Your BIOS Boot Partition is under 1 MiB, please increase its size..
```
不过无实际影响。

产物：
/openwrt-imagebuilder-25.12.2-x86-generic.Linux-x86_64/bin/targets/x86/generic/openwrt-25.12.2-x86-generic-generic-ext4-combined-efi.img.gz

按官方推荐的用ext4-combined-efi.img.gz

解压后，放到SD卡内（如果普通电脑可使用第2个U盘）

名字可改短一点，如openwrt.img

# 准备启动U盘

一般的电脑，随便什么linux发行版都行

这台只能i386，所以

[https://zenlayer.dl.sourceforge.net/project/gparted/gparted-live-stable/1.6.0-10/gparted-live-1.6.0-10-i686.iso](https://zenlayer.dl.sourceforge.net/project/gparted/gparted-live-stable/1.6.0-10/gparted-live-1.6.0-10-i686.iso)

一开始用的debian，无法挂载，ubuntu-16.04.6-server-i386.iso也不行，因此换这个

下载完成后dd写入。
```bash
lsblk
```
确认硬盘正确，不要写错了（我毁了自己系统1次）
```bash
sudo dd if=./gparted-live-1.6.0-10-i686.iso of=/dev/sdd
```
U盘灯不闪后拨

# 刷机

用刚才的u盘启动

![mobile_net_setup_dd]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_dd.jpg)

用dd写入。

设置openwrt：

![mobile_net_setup_luci1]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_luci1.png)
![mobile_net_setup_luci2]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_luci2.png)
![mobile_net_setup_luci3]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_luci3.png)

# 连接

- 中兴F50 - 尽可能短的USB线 - 笔记本
- 笔记本 - 网线 - 电信光猫
- 电脑光猫 - 其他上网设备 

![mobile_net_setup_connect]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_connect.jpg)

光猫关闭dhcp，IP地址设置在192.168.1.2-99之间即可。

![mobile_net_setup_modem]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_modem.png)

# 扩展

openwrt扩容命令（官方两条合一）
```bash
opkg update
opkg install parted
opkg install losetup resize2fs
parted -l -s
parted -f -s /dev/sda resizepart 2 100%
reboot
```
parted -l -s是用来看硬盘情况的，可以不打。重启后：
```bash
losetup /dev/loop0 /dev/sda2 2> /dev/null
resize2fs -f /dev/loop0
reboot
```
这样就有200G空间了。

# 关于中兴F50供电玄学

即使线是短的，仍有机率触发无法开机。因此：
- 尽量在开机之前插好线
- 开机后尽量不乱动
- 如果什么操作导致了不开机，建议重刷
![mobile_net_setup_flash]({% staticBase %}{% endstaticBase %}/openwrt/mobile_net_setup_flash.png)
- 重刷的系统一般可以正常开机
- 后面操作多了，可能就不开机了。
