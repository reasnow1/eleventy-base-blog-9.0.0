---
title: 安装Fedora服务器 Fedora server setup
description: This is a post on My Blog about agile frameworks.
date: 2026-04-19
tags: Fedora
---
因为电脑用的debian，所以，为了多样性，服务器用Fedora。

依旧：

[https://mirrors.nyist.edu.cn/fedora/releases/43/Server/x86_64/iso/Fedora-Server-dvd-x86_64-43-1.6.iso](https://mirrors.nyist.edu.cn/fedora/releases/43/Server/x86_64/iso/Fedora-Server-dvd-x86_64-43-1.6.iso)

下载镜像。

完成后，安装，有图形界面。弄好用户（不要ROOT），硬盘，软件

- Container Management 容器用
- Domain Membership 多人使用
- Guest Agents 虚拟机使用
- Hardware Support for Server Systems 物理机使用
- Headless Management WEB管理页面

我是只要了Hardware Support for Server Systems

安装完成后，开机，可以看到提示

https://IP地址:9090/有web console

不过，懒得用了。

SSH上去后：

```bash
sudo vim /etc/yum.repos.d/fedora.repo
```
直接用镜像站
```config
[fedora]
name=Fedora $releasever - $basearch
failovermethod=priority
baseurl=https://mirror.nyist.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
metadata_expire=28d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-debuginfo]
name=Fedora $releasever - $basearch
failovermethod=priority
baseurl=https://mirror.nyist.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
metadata_expire=28d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-source]
name=Fedora $releasever - $basearch
failovermethod=priority
baseurl=https://mirror.nyist.edu.cn/fedora/releases/$releasever/Everything/$basearch/os/
metadata_expire=28d
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```
```bash
sudo vim /etc/yum.repos.d/fedora-updates.repo
```
```config
[updates]
name=Fedora $releasever - $basearch - Updates
failovermethod=priority
baseurl=https://mirror.nyist.edu.cn/fedora/updates/$releasever/Everything/$basearch/
enabled=1
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[updates-debuginfo]
name=Fedora $releasever - $basearch - Updates
failovermethod=priority
baseurl=https://mirror.nyist.edu.cn/fedora/updates/$releasever/Everything/$basearch/
enabled=1
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[updates-source]
name=Fedora $releasever - $basearch - Updates
failovermethod=priority
baseurl=https://mirror.nyist.edu.cn/fedora/updates/$releasever/Everything/$basearch/
enabled=1
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```
现在可以更新了。
```bash
sudo yum update
```
查看笔记本电量：
```bash
$ ls /sys/class/power_supply/
ACAD  BAT1
$ ls /sys/class/power_supply/BAT1
alarm           energy_full_design  power          technology
capacity        energy_now          power_now      type
capacity_level  extensions          present        uevent
cycle_count     hwmon2              serial_number  voltage_min_design
device          manufacturer        status         voltage_now
energy_full     model_name          subsystem
$ cat /sys/class/power_supply/BAT1/capacity   
98
$ cat /sys/class/power_supply/BAT1/status
Discharging
```
```bash
cat /sys/class/power_supply/BAT1/capacity
```
关闭屏幕背光：
```bash
$ ls /sys/class/backlight/
radeon_bl0
$ ls /sys/class/backlight/radeon_bl0
actual_brightness  brightness      power  subsystem  uevent
bl_power           max_brightness  scale  type
$ echo 0 | sudo tee /sys/class/backlight/radeon_bl0/brightness
0
```
```bash
echo 0 | sudo tee /sys/class/backlight/radeon_bl0/brightness
```
查看硬件：（有兴趣可看）
```bash
lscpu
free -h
lsblk
lspci
```


硬盘空间：之前不注意看，结果：
```bash
lsblk
NAME            MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda               8:0    0 223.6G  0 disk 
├─sda1            8:1    0   600M  0 part /boot/efi
├─sda2            8:2    0     2G  0 part /boot
└─sda3            8:3    0   221G  0 part 
  └─fedora-root 252:0    0    15G  0 lvm  /
zram0           251:0    0   5.3G  0 disk [SWAP]
```
用以下命令扩容：
```bash
sudo lvextend -l +100%FREE /dev/mapper/fedora-root
sudo xfs_growfs /    # because Fedora uses xfs by default
```

END
<br/>
<br/>

---
注：repo原配置参考如下：
/etc/yum.repos.d/fedora.repo
```config
[fedora]
name=Fedora $releasever - $basearch
#baseurl=http://download.example/pub/fedora/linux/releases/$releasever/Everything/$basearch/os/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-$releasever&arch=$basearch
enabled=1
countme=1
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-debuginfo]
name=Fedora $releasever - $basearch - Debug
#baseurl=http://download.example/pub/fedora/linux/releases/$releasever/Everything/$basearch/debug/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-debug-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[fedora-source]
name=Fedora $releasever - Source
#baseurl=http://download.example/pub/fedora/linux/releases/$releasever/Everything/source/tree/
metalink=https://mirrors.fedoraproject.org/metalink?repo=fedora-source-$releasever&arch=$basearch
enabled=0
metadata_expire=7d
repo_gpgcheck=0
type=rpm
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```
/etc/yum.repos.d/fedora-updates.repo
```config
[updates]
name=Fedora $releasever - $basearch - Updates
#baseurl=http://download.example/pub/fedora/linux/updates/$releasever/Everything/$basearch/
metalink=https://mirrors.fedoraproject.org/metalink?repo=updates-released-f$releasever&arch=$basearch
enabled=1
countme=1
repo_gpgcheck=0
type=rpm
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[updates-debuginfo]
name=Fedora $releasever - $basearch - Updates - Debug
#baseurl=http://download.example/pub/fedora/linux/updates/$releasever/Everything/$basearch/debug/
metalink=https://mirrors.fedoraproject.org/metalink?repo=updates-released-debug-f$releasever&arch=$basearch
enabled=0
repo_gpgcheck=0
type=rpm
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False

[updates-source]
name=Fedora $releasever - Updates Source
#baseurl=http://download.example/pub/fedora/linux/updates/$releasever/Everything/SRPMS/
metalink=https://mirrors.fedoraproject.org/metalink?repo=updates-released-source-f$releasever&arch=$basearch
enabled=0
repo_gpgcheck=0
type=rpm
gpgcheck=1
metadata_expire=6h
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-fedora-$releasever-$basearch
skip_if_unavailable=False
```