---
title: 找虐之安装Arch Linux 第2篇 Install Arch Linux chapter 2
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-30
tags: Arch Linux
---

注：本文写的时候无中文输入法，因为操作多怕忘了所以先写，中文是后面补的

right now i successfully installed desktop enviroment but i don't have a chinese type method

but i really want to write things down otherwise i might forget!

# 进入命令行后after install

用root登录login as root

首先添加用户first i add a user

```bash
useradd -m -G wheel -s /bin/bash *username*
passwd *username*
```

加到轮子组里，后面可以用 su 。add to wheel so we can su later.

现在now

```bash
exit
```

然后用普通用户登录and login common user.

sudo并没有安装，不过也不需要，只要sudo is not installed by default but we don't actually need just

```bash
su
```

弄完 exit 即可。and exit after done.

现在你可以ping一下，会发现网络不通right now you could try to ping and you will find that network is not aviliable

对的，刚才安装的时候是通的yes, when we install Arch linux network is aviliable

现在又不通了now it just gone!

use

```bash
ip link
ip link set enp5s0 up
```

查看网络接口，记得用真实的接口名字to check interfaces and set it up, be sure to use actual name

then

```bash
cat > /etc/systemd/network/20-wired.network << EOF
[Watch]
Name=enp5s0
[Network]
DHCP=yes
EOF
```

这个配置文件，官方还要多一句话，不过不鸟了on official website there is another line but i ignored.

```bash
systemctl start systemd-networkd
systemctl enable systemd-networkd
```

好像是要systemd-networkd.service才对，不记得了or systemd-networkd.service? i don't remember

现在可以now you can

```bash
ping 223.6.6.6
```

但不能but you can't

```bash
pacman -Syu
```

因为没有域名解析because there is no DNS

```bash
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf
systemctl start systemd-resolved
systemctl enable systemd-resolved
```

现在有网了you can

```bash
pacman -Syy
```

now

```bash
pacman -S nano
```

终于有文本编辑器了，感动得我眼泪都要掉下来 T_T  oh God i finally have a text editer i am almost moved to tears! T_T

now

```bash
nano /boot/loader/loader.conf
```

```conf
timeout 3
```

取消这个注释，这样启动的时候有机会改启动选项this gives us time to config the boot option.

```bash
nano /boot/loader/entries/arch.conf
```

直接删掉nomodeset just removed nomodeset

但是现在还不能不用nomodeset 启动，所以每次都要加nomodeset 。but now we can't boot without it so i need to press e on boot and type nomodeset everytime.

安装显卡驱动now install driver:

https://wiki.archlinux.org/title/Graphics_processing_unit#Installation

实质上我是看不懂的，是

```bash
lspci -vnnd ::03xx
```

之后发给deepseek，他说我是 GCN 4 代
i am GCN gen 4 so

```bash
pacman -S mesa vulkan-radeon xf86-video-amdgpu
```

then

```bash
mkinitcpio -P
reboot
```

此时仍要nomodeset。unfortunately it doesn't work...

安装桌面install desktop:

```bash
pacman -Sy mate mate-extra
```

全部按回车press Enter on everything

```bash
pacman -S lightdm lightdm-gtk-greeter
systemctl enable lightdm
systemctl start lightdm
reboot
```

现在有桌面了，但分辨率不对。now we have desktop but the solution is wrong

继续研究the research continues......