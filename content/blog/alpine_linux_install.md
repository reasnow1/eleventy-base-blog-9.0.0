---
title: 安装Alpine linux 安装Alpine linux install
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-25
tags: Alpine linux
---

安装Alpine linux
下载地址（i386架构）

https://mirrors.nyist.edu.cn/alpine/latest-stable/releases/x86/alpine-extended-3.23.4-x86.iso

选择extended就可以不用网络安装。

启动后，输入root，密码留空登录，输入

```bash
setup-alpine
```

进行安装。安装要点：

网络设置：

```bash
Available interfaces are: eth0 wlan0.
Which one do you want to initialize? (or '?' or 'done') [eth0] 
Do you want to do any manual network configuration? (y/n) [n] 
Available interfaces are: wlan0.
Which one do you want to initialize? (or '?' or 'done') [wlan0] done 
```

第一个输入eth0然后使用dhcp，第二个输入done跳过wifi设置。

硬盘设置：

```bash
Which disk(s) would you like to use? (or '?' for help or 'none') sda
Confirmation for the chosen disk(s) appears. The following disk is selected: sda (128.0 GB JMicron Tech ).
How would you like to use it? ('sys', 'data', 'lvm' or '?' for help) sys
```

选择硬盘后使用sys。

- 主机名：最好设一个，方便后面路由器查看IP
- 镜像站：由于我们用了extended所以暂时不弄。
- SSH：打开，我选择了OpenSSH。
- 普通用户：可以创造一个，用于后面rtorrent

重启进入系统。

查看是否有IP：

```bash
ip addr show
```

如果没有IP可以输入以下命令开网口，拿IP

```bash
ip link set eth0 up
udhcpc -i eth0
```

此时仍然无法SSH登录，需要在笔记本上操作。

输入root账号密码登录后：

修改Edit /etc/ssh/sshd_config:

```bash
vi /etc/ssh/sshd_config
```

找到这行Find the line:

```
#PermitRootLogin prohibit-password
```

改为Change it to:

```
PermitRootLogin yes
```

重启SSH服务器Restart OpenSSH:

```bash
rc-service sshd restart
```

即可登录。

接下来在主机上操作：

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
Generating public/private ed25519 key pair.
Enter file in which to save the key (/home/用户名/.ssh/id_ed25519): 
Enter passphrase for "/home/用户名/.ssh/id_ed25519" (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/用户名/.ssh/id_ed25519
Your public key has been saved in /home/用户名/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:Nwm...... your_email@example.com
The key's randomart image is:
+--[ED25519 256]--+
（图案省略）
+----[SHA256]-----+
```

```bash
cat /home/用户名/.ssh/id_ed25519.pub
ssh-ed25519 AAAA...... your_email@example.com
```

复制

```text
ssh-ed25519 AAAA...... your_email@example.com
```

SHA256:Nwm...... your_email@example.com
这一段

在Alpine上操作：

```bash
localhost:~# mkdir -p /root/.ssh
localhost:~# chmod 700 /root/.ssh
localhost:~# echo "ssh-ed25519 AAAA...... your_email@example.com" >> /root/.ssh/authorized_keys
localhost:~# chmod 600 /root/.ssh/authorized_keys
```

此时可以尝试退出再登录，应该可以无密码登录。

可以再把刚才改为Change it to:

```config
PermitRootLogin prohibit-password
```

```bash
rc-service sshd restart
```

现在增加镜像源：

```bash
vi /etc/apk/repositories
```

```config
https://mirror.nyist.edu.cn/alpine/v3.24/main/
https://mirror.nyist.edu.cn/alpine/latest-stable/community/
```

```bash
apk update
```

关闭屏幕背光：

```bash
localhost:~# ls /sys/class/backlight/
intel_backlight
localhost:~# ls /sys/class/backlight/intel_backlight
actual_brightness  device             scale              uevent
bl_power           max_brightness     subsystem
brightness         power              type
localhost:~# cat /sys/class/backlight/intel_backlight/brightness
2000000
localhost:~# echo 0 > /sys/class/backlight/intel_backlight/brightness
```

```bash
echo 0 > /sys/class/backlight/intel_backlight/brightness
```