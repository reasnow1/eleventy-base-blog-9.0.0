---
title: 找虐之安装Arch Linux 第3篇 Install Arch Linux chapter 3
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-05-01
tags: Arch Linux
---
我复活了，经过前面的奋战之后，终于解决了nomodeset的问题。

而且，还不知道是怎么解决的。

首先，如果什么都不加，就会直接黑屏。

如果加上
```config
amdgpu.dc=0
```
那么启动到一半就会卡住。

我首先尝试：
```bash
pacman -S linux-firmware-amdgpu
mkinitcpio -P
```
没有效果。

然后又手动改mkinitcpio的配置，无效，又改回来。

然后尝试用这个启动：
```config
radeon.cik_support=0 amdgpu.cik_support=1
```
无效。

然后查论坛，说是先不加参数启动一次，然后按机箱按键重启，重启后：
```bash
journalctl -b -1
```
来查看日志。此时deepseek建议用：
```config
amdgpu.dc=0 video=HDMI-A-1:e
```
启动。然后失败。

又建议用
```config
amdgpu.dc=0 amdgpu.gpu_recovery=1 amdgpu.aspm=0
```
启动。失败。

然后此时deepseek提出要拨掉所有USB设备？what？但是之前启动，确实是最后卡在usb加载那里，所以...

我把除了键盘之外的所有东西拨了，然后用
```config
amdgpu.dc=0
```
启动。然后失败。

然后我是输入参数后，按回车，按的一瞬间拨掉键盘。也不得。

然后重启，我忘记接键盘，所以，直接没有参数启动了！

然后，就成功进桌面了！

然后，把USN东西全部插回去，启动，成功

所以，最后莫名其妙地成功了......

现在安装中文字体和输入法...

直接放弃思考了，deepseek给什么执行什么。
```bash
pacman -S noto-fonts-cjk wqy-microhei wqy-zenhei
```
deepseek建议把
```bash
nano /etc/locale.gen
```
```config
#zh_CN.UTF-8 UTF-8
```
之类的全部取消注释，但其实不注释也行。
```bash
locale-gen
```
输入法：
```bash
pacman -S fcitx5 fcitx5-chinese-addons fcitx5-gtk fcitx5-qt fcitx5-config-qt
```
然后加五笔，调字体之类的略过了，自己调一下

这个Mate桌面环境，风格我喜欢，但是不见有“黑夜模式”

要自己把主题换黑的，然后，再把浏览器的主题也换黑。
