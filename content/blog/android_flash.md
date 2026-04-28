---
title: 魅族note16解bl锁（简陋记录） Meizu Note 16 unlock
description: This is a post on My Blog about agile frameworks.
date: 2026-04-13
tags: Android
---

因为魅族note16限制太多，话不多说直接开干。

首先，要windows环境。

下载是在[http://qutick102.ysepan.com/](http://qutick102.ysepan.com/)

需要： 紫光驱动_R4.21.3201.zip20.4MB

 ums9621_Meizu_Note16.zip1.2MB

---

下载后，用windows，

先解压，紫光驱动，DriversForWin10

不知道用不用先点DriverUninstall64.exe

反正我是卸了又删很多次。

DriverSetup.exe，一闪而过

然后DPInst64.exe我又点了一下。

---

开ums9621_Meizu_Note16

现在，连接手机

是音量下＋电源

酷安有人说是上，害我惨了。

然后我当时是运行spd_dump.exe

它是等待30秒

后面显示手机连上了。我就ctrl+C，然后

运行unlock_autopatch_9621.bat

中间几次要按键，无脑按，有等待就等待。

成功后，手机重启，是出厂状态。

---

此时bl已解。

到魅族官网下包，大，最好提前下

[https://www.flyme.com/firmwarelist-203.html](https://www.flyme.com/firmwarelist-203.html)

下载后，是卡刷的，然后

[https://github.com/tobyxdd/android-ota-payload-extractor](https://github.com/tobyxdd/android-ota-payload-extractor)

用这个来解。

直接命令./android-ota-extractor payload.bin

出几个，出了boot.img就可以停了。

---

然后复制这个上去手机。

[https://github.com/topjohnwu/Magisk](https://github.com/topjohnwu/Magisk)

下载magisk的apk

```shell
adb devices
```

看手机，开USB调试。
```shell
adb install Magisk-v30.7.apk
```
进去后点install，选img，修补后，拿到电脑上。
```shell
adb reboot bootloader
```
然后，这里不能刷，还得来
```shell
fastboot reboot fastboot
```
之后
```shell
fastboot getvar current-slot
```
我这里是
```shell
current-slot: b
Finished. Total time: 0.003s
```
所以，刷b
```shell
fastboot flash boot_b magisk_patched-30700_Ey4YH.img
fastboot reboot
```
然后fastboot会等待，可直接掐。

这里，已经得ROOT了。

（我以为没得，点install to inactive slot，然后A分区直接寄，后面把官方
update.zip放进去，打开文件管理，点安装，才修复得的。）

注：换区
```shell
fastboot set_active a
```

---

进入后，备份：
```shell
adb shell
su
```
这里，授权时间，点forever，不要到一半直接寄
```shell
for part in $(ls /dev/block/by-name/); do
    if [ "$part" != "sda" ] && [ "$part" != "userdata" ]; then
        echo "Backing up $part ..."
        dd if=/dev/block/by-name/$part of=/sdcard/backup2/$part.img bs=1M
    else
        echo "Skipping $part"
    fi
done
```

```shell
Backing up vendor_boot_b ...
100+0 records in
100+0 records out
104857600 bytes (100 M) copied, 0.269 s, 372 M/s
```
最后一行

---

后注：
tmux 换源
```shell
termux-change-repo
```
开ssh：
```shell
pkg install openssh
sshd
```

如果用
```shell
pm install *.apk
```
会提示不给安装，然后有个要你到什么什么地方安装的提示，要把安装包挪到那里才能安装