---
title: 虐心之网卡问题电脑装机 Install Debian on a external pci net card
description: This is a post on My Blog about agile frameworks.
date: 2026-04-18
tags: Debian
---
![debian-r8168-pcbox]({% staticBase %}{% endstaticBase %}/debian/debian-r8168-pcbox.jpg)

如图所示，这台电脑的主板自带网卡坏了，所以，用了额外的PCI网卡。

但是，这个网卡，刚开始你进系统是有网络的，过了一会儿，就没网

纯折磨人

解决：首先不能用CD镜像，就是不用netinst.iso，用

[https://mirror.nyist.edu.cn/debian-cd/current/amd64/iso-dvd/debian-13.4.0-amd64-DVD-1.iso](https://mirror.nyist.edu.cn/debian-cd/current/amd64/iso-dvd/debian-13.4.0-amd64-DVD-1.iso)

使用DVD镜像。如果下载慢，可以用磁力链

[https://mirror.nyist.edu.cn/debian-cd/current/amd64/bt-dvd/debian-13.4.0-amd64-DVD-1.iso.torrent](https://mirror.nyist.edu.cn/debian-cd/current/amd64/bt-dvd/debian-13.4.0-amd64-DVD-1.iso.torrent)

下载完成后，装机，暂时跳过网络镜像环节。

装好后，开机是有网的，一旦用了一下，立马无。

查日志：
```bash
sudo dmesg | grep -E "r8169|enp3s0|rtl_counters"
```
```log
[ 1751.678064] r8169 0000:04:00.0 enp4s0: NETDEV WATCHDOG: CPU: 2: transmit queue 0 timed out 5804 ms
[ 1751.816075] r8169 0000:04:00.0 enp4s0: rtl_chipcmd_cond == 1 (loop: 100, delay: 100).
```
关键问题：被watchdog给弄了。

找USB网线转接，或者手机USB网络共享，先上网。

网络测试：（阿里云服务器）
```bash
ping 223.6.6.6
```
默认没有源，加
```bash
sudo nano /etc/apt/sources.list
```
```config
#deb cdrom:[Debian GNU/Linux 13.4.0 _Trixie_ - Official amd64 NETINST with firmware 20260314-11:53]/ trixie contrib main non-free-firmware

deb https://mirror.nyist.edu.cn/debian/ trixie main contrib non-free non-free-firmware
deb-src https://mirror.nyist.edu.cn/debian/ trixie main contrib non-free non-free-firmware

deb https://mirror.nyist.edu.cn/debian-security trixie-security main contrib non-free non-free-firmware
deb-src https://mirror.nyist.edu.cn/debian-security trixie-security main contrib non-free non-free-firmware

# trixie-updates
deb https://mirror.nyist.edu.cn/debian/ trixie-updates main contrib non-free non-free-firmware
deb-src https://mirror.nyist.edu.cn/debian/ trixie-updates main contrib non-free non-free-firmware
```
```bash
sudo apt update
sudo apt install r8168-dkms
echo "blacklist r8169" | sudo tee /etc/modprobe.d/blacklist-r8169.conf
sudo update-initramfs -u
sudo reboot
```
这样就禁用了原来的r8169。

重启之后，并没有网卡。于是：
```bash
$ lspci -nnk | grep -A3 -i ethernet
03:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8211/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 07)
	Subsystem: Realtek Semiconductor Co., Ltd. Device [10ec:0123]
	Kernel modules: r8169
04:00.0 Ethernet controller [0200]: Realtek Semiconductor Co., Ltd. RTL8111/8168/8211/8411 PCI Express Gigabit Ethernet Controller [10ec:8168] (rev 02)
	Subsystem: Realtek Semiconductor Co., Ltd. RTL8111/8168 PCI Express Gigabit Ethernet controller [10ec:8168]
	Kernel modules: r8169

$ sudo modprobe -r r8169

$ sudo dkms install r8168/8.055.00

Error! Your kernel headers for kernel 6.12.73+deb13-amd64 cannot be found at /lib/modules/6.12.73+deb13-amd64/build or /lib/modules/6.12.73+deb13-amd64/source.
Please install the linux-headers-6.12.73+deb13-amd64 package or use the --kernelsourcedir option to tell DKMS where it's located.
```
由此可见，是这个什么响应头的问题。
```bash
sudo apt install linux-headers-$(uname -r)
```
这个命令其实安装的就是linux-headers-6.12.73+deb13-amd64。

装的时候，可以看到有编译内核，然后重启，启动的时候，可以看到r8168加载，最后终于上网！

注意：建议关闭软件商店的更新，就是那个重启的时候要求更新的。如果不关，更新之后，会掉驱动，需要重新用手机联网后执行
```bash
sudo apt install linux-headers-$(uname -r)
```

另外：刚才 有执行以下操作：
```bash
sudo nano /etc/default/grub
```
原来：
```config
GRUB_CMDLINE_LINUX_DEFAULT="quiet"
```
改：
```config
GRUB_CMDLINE_LINUX_DEFAULT="quiet pcie_aspm=off"
```
```bash
sudo update-grub
```
这招并没有奏效，后面成功之后，也没改回来，不知道是否发挥效果

注：显卡：
```bash
$ lspci | grep -iE "3d|display|vga" | grep -i nvidia
01:00.0 VGA compatible controller: NVIDIA Corporation GK208B [GeForce GT 730] (rev a1)

sudo apt install nvidia-kernel-dkms nvidia-driver
```
装一半，提示：
```info
This system has a graphics card which is no longer handled by the NVIDIA driver (package nvidia-driver). You may wish to keep the package  installed - for instance to drive some other card - but the card with the following chipset won't be usable:                                                                      
01:00.0 VGA compatible controller [0300]: NVIDIA Corporation GK208B         
[GeForce GT 730] [10de:1287] (rev a1)                                       
The above card requires either the non-free legacy NVIDIA driver (package nvidia-tesla-470-driver) or the free Nouveau driver (package xserver-xorg-video-nouveau). 
Use the update-glx command to switch between different installed drivers.                                                                    
 Before the Nouveau driver can be used you must remove NVIDIA configuration from xorg.conf (and xorg.conf.d/).  
 
  Install NVIDIA driver despite unsupported graphics card? 
  <Yes>                  <No> 
 ```
 因此，选择了no，然后nvidia-tesla-470-driver只在debian12里面有，所以，只装了个捞版解决。
 ```bash
 sudo apt install xserver-xorg-video-nouveau
 ```
修改windows为默认启动项：
```bash
sudo grep -i "^menuentry" /boot/grub/grub.cfg

menuentry 'Debian GNU/Linux' --class debian --class gnu-linux --class gnu --class os $menuentry_id_option 'gnulinux-simple-583031e6-4ea4-4343-b382-eb8f44489c53' {
menuentry 'Windows 10 (on /dev/sdb1)' --class windows --class os $menuentry_id_option 'osprober-chain-7EBC955FBC95132F' {
```
```bash
sudo nano /etc/default/grub
```
将
```config
GRUB_DEFAULT=0
```
改为
```config
GRUB_DEFAULT="Windows 10 (on /dev/sdb1)"
```
生效：
```bash
sudo update-grub
```