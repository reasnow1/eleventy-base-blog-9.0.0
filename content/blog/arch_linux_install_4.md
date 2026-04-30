---
title: 找虐之安装Arch Linux 第4篇 Install Arch Linux chapter 4
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-05-02
tags: Arch Linux
---
现在开始进入软件阶段

sudo apt update
```bash
pacman -Syu
```
浏览器：
```bash
pacman -S firefox
```
VS code
```bash
pacman -S code
```
FUSE库，用于运行AppImage：
```bash
pacman -S fuse2
```
添加运行权限命令
```bash
chmod +x 文件名.AppImage
```
魔法使用：

[https://github.com/libnyanpasu/clash-nyanpasu](https://github.com/libnyanpasu/clash-nyanpasu)

Arduino权限不足解决：
```bash
ls -l /dev/ttyACM*
crw-rw---- 1 root uucp 166, 0 Apr 29 09:12 /dev/ttyACM0
usermod -a -G uucp 用户名
reboot
```
参考文献：[https://forum.arduino.cc/t/permission-denied-on-dev-ttyacm0/475568](https://forum.arduino.cc/t/permission-denied-on-dev-ttyacm0/475568)

node.js和npm
```bash
pacman -S nodejs npm
```
验证
```bash
node -v
npm -v
```
构建博客
```bash
npm install
npx @11ty/eleventy
```
文件传输
```bash
pacman -S filezilla
```
搜索包
```bash
pacman -Ss 包名
```
删除包
```bash
pacman -R 包名
```
无声音修复：
```bash
cat /proc/asound/cards
 0 [SmartMouse     ]: USB-Audio - SmartMouse
                      ZY.Ltd SmartMouse at usb-0000:00:14.0-7, full speed
 1 [PCH            ]: HDA-Intel - HDA Intel PCH
                      HDA Intel PCH at 0xf7f20000 irq 136
 2 [HDMI           ]: HDA-Intel - HDA ATI HDMI
                      HDA ATI HDMI at 0xf7e60000 irq 137
```
```bash
sudo usermod -aG audio xinya
# then log out & in
echo 'defaults.pcm.card 1' > ~/.asoundrc
echo 'defaults.ctl.card 1' >> ~/.asoundrc
alsamixer   # F6 to choose HDA Intel PCH, unmute Master & PCM
speaker-test -t wav -c 2   # success!
```
之前显卡问题：

经测试，似乎不是USB问题。

通过
```config
amdgpu.dc=0
```
启动后，实质上系统已经启动。

等待启动完毕后，拨掉HMDI线，再插上

可以按
- ctrl+alt+f2
- root
- 回车
- 密码
- 回车
- reboot

或者重启

之后一切正常。