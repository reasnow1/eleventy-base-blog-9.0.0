---
title: 找虐之安装Arch Linux Install Arch Linux
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-29
tags: Arch Linux
---
找虐之安装Arch Linux

玩linux，怎么能不 挑战一下“最困难版本”

文档看似写得很好，实则，有些坑

接下来细说

首先，下载地址，我用南方科技大学的镜像站了：

[https://mirrors.sustech.edu.cn/archlinux/iso/2026.04.01/archlinux-2026.04.01x86_64.iso](https://mirrors.sustech.edu.cn/archlinux/iso/2026.04.01/archlinux2026.04.01-x86_64.iso)

安装完成后用Ventoy或者dd弄到U盘上。

开机，我的BIOS会写有：

- Kingston storage
- UEFI Kingston storage

选择UEFI，这样，后面的启动就是UEFI了。

开幕直接下马威，屏幕没有显示，可能跟显卡（RX 580）有关。

启动之前，按e，光标移到最后，输入

```config
nomodeset
```

进入。

现在可以开始品鉴官方说明

[<https://wiki.archlinux.org/title/Installation_guide#Verify_the_boot_mode>]

(<https://wiki.archlinux.org/title/Installation_guide#Verify_the_boot_mode>)

前面有设置键盘，我们用英文键盘，可以忽略。

```bash
cat /sys/firmware/efi/fw_platform_size
64
```

很好，继续

```bash
ping 223.6.6.6
```

我没有任何设置，插网线到主板上，就可以上网了。

```bash
timedatectl
fdisk -l
```

接下来，开始第一个坑点：压根不告诉你怎么用fdisk。不过，至少是列出了建议的布局。

我们使用deepseek补上。

## Step-by-step `fdisk` for UEFI

1. **Start fdisk** (as root):

   ```bash
   fdisk /dev/sda
   ```
2. **Create a new GPT partition table** (wipes old partitions):
   - Type `g` → press Enter.
3. **Create the EFI partition** (about 512 MB, type EFI):
   - Type `n` → Enter (new partition).
   - Partition number: `1` → Enter.
   - First sector: (just press Enter to accept default).
   - Last sector: `+1GiB` → Enter.
   - 因为我之前是debian所以这里说有之前的，按enter抹掉
   - Now change its type: type `t` → Enter.
   - Partition type: `1` (EFI System) → Enter.
   - 这里我好像没看到要弄类型？

3.5补上swap分区，分区序号2，官方要求最少4GiB，我用了6GiB

1. **Create the root partition** (remaining space, Linux filesystem): 
   - Type `n` → Enter.
   - Partition number: `3` → Enter (or accept default).
   - First sector: (press Enter).
   - Last sector: (press Enter to use all remaining space).
   - 最后一步没印象有没有
   - For type, Linux filesystem is usually default; you can skip `t` or verify

with `l`.

1. **Write the changes**: 
   - Type `w` → Enter. This writes the partition table and exits.

接下来可以按官方的步骤走。

- root_partition就是sda3，
- swap_partition是sda2，
- efi_system_partition是sda1。

......

镜像站部分：

```bash
vim /etc/pacman.d/mirrorlist
```

- 进入后，最上面是worldwide，我采取了注释处理。
- 然后，中间一大段，直接长按d删除。
- 删到china住手。
- 留china，然后光标到下面，
- 直接d然后G全删。
- :wq

现在开始第二个坑点，一堆软件包，看得人眼睛花。

由于不知道是干啥用的，我只运行了官方的最简命令：

```bash
pacstrap -K /mnt base linux linux-firmware
```

现在，我强烈建议此处加一个nano

```bash
pacstrap -K /mnt base linux linux-firmware nano
```

原因后面有。

现在继续，到time部分，补上天朝的时区

```bash
ln -sf /usr/share/zoneinfo/Area/Location /etc/localtime
```

设置locale，主机名，Initramfs（LVM用的）可不鸟，就设个root密码

接下来，最后一个大坑，官方原文：

```text
3.8 Boot loader

Choose a boot loader applicable to your partitioning scheme and install it. See 

the explanations and the comparison table in Arch boot process#Boot loader to 

make your choice, then follow the installation instructions on its dedicated 

page. 
```

让你自己选一个bootloader，然后你点了进去看，是一个对比表格

然后点其中一个，里面是讲怎么用的，

完全没讲在现在（chroot）环境下如何操作

我这里用AI补了。

```bash
mount -t efivarfs efivarfs /sys/firmware/efi/efivars
bootctl install
```

此处出现提示

```warn
monut point /boot which back the random seed file is world accessable ,which is a 

security hole
```

如果电脑是多用户使用的，可以管一下，我一个人用，不鸟了。

```bash
blkid -s PARTUUID -o value /dev/sda3
```

出现输出，看住

接下来，要写conf，如果你没有选nano，现在，无论是nano,vi,vim全部没有

只能

```bash
cat > /boot/loader/entries/arch.conf << EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=你上一步得到的值 rw
EOF
```

现在可以按官方步骤出来了。

```bash
exit
umount -R /mnt
reboot
```

也可以poweroff方便拨U盘。

然后，我启动，发现：黑屏！！

因为没有nomodeset！

然后，又得重新进安装环境。

刚才如果按官网的是会mount，现在得手动

```bash
mount /dev/sda3 /mnt
mount /dev/sda1 /mnt/boot
arch-chroot /mnt
```

然后md因为没有nano导致又挨重新rm然后cat进去一次

```bash
cat > /boot/loader/entries/arch.conf << EOF
title   Arch Linux
linux   /vmlinuz-linux
initrd  /initramfs-linux.img
options root=PARTUUID=重新执行看一次值 rw nomodeset
EOF
```

按deepseek的说法，这里要执行

```bash
mkinitcpio -P
```

然后提示少个啥，直接不鸟

```bash
exit
umount -R /mnt
reboot
```

现在终于进系统了！不过，只有命令行。后面再研究桌面。