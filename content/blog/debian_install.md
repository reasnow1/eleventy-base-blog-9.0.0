---
title: Debian安装要点 About installing Debian
description: This is a post on My Blog about agile frameworks.
date: 2026-04-04
tags: Debian
---

## 下载地址

```text
https://mirror.sysu.edu.cn/debian-cd/current/amd64/iso-cd/debian-13.4.0-amd64-netinst.iso
```

## 安装过程关键点

### 设置 root 密码
- **选择留空**，之后使用普通用户并通过 `sudo` 提权。

### 硬盘分区
- **不使用 LVM**，选择 `use entire disk` 更简单。
- 注意看清要安装的硬盘，在 **GRUB 安装位置** 的选择界面，选择与 Debian 同一块硬盘。

### 镜像源选择
- 地理位置最近的镜像：`https://mirror.sysu.edu.cn/`
- 如果有代理，可以暂时选择 `debian.org`
- **一定要打开代理**，因为需要安装 GNOME 桌面。

## 显卡问题处理（RX 580 可能黑屏）

第一次启动会黑屏，在 GRUB 界面：

1. 按 `e` 编辑启动项
2. 找到 `linux` 开头的行，在末尾加上 `amdgpu.dc=0`（注意前面有空格）

进入系统后，永久修复：

```bash
sudo nano /etc/default/grub
```

找到 `GRUB_CMDLINE_LINUX_DEFAULT="quiet"`，改为：

```bash
GRUB_CMDLINE_LINUX_DEFAULT="quiet amdgpu.dc=0"
```

更新 GRUB：

```bash
sudo update-grub
```

## 换源（使用国内镜像）

编辑 `/etc/apt/sources.list`，替换为以下内容：

```bash
#deb cdrom:[Debian GNU/Linux 13.4.0 _Trixie_ - Official amd64 NETINST with firmware 20260314-11:53]/ trixie contrib main non-free-firmware

deb https://mirror.sysu.edu.cn/debian/ trixie main contrib non-free non-free-firmware
deb-src https://mirror.sysu.edu.cn/debian/ trixie main contrib non-free non-free-firmware

deb https://mirror.sysu.edu.cn/debian-security trixie-security main contrib non-free non-free-firmware
deb-src https://mirror.sysu.edu.cn/debian-security trixie-security main contrib non-free non-free-firmware

# trixie-updates
deb https://mirror.sysu.edu.cn/debian/ trixie-updates main contrib non-free non-free-firmware
deb-src https://mirror.sysu.edu.cn/debian/ trixie-updates main contrib non-free non-free-firmware
```

更新软件源：

```bash
sudo apt update
```

## 安装输入法（fcitx5）

```bash
sudo apt install fcitx5 fcitx5-chinese-addons
```

## 代理设置（清理）

如果安装时设置了代理，在 `/etc/apt/apt.conf` 中可能留有配置，例如：

```bash
Acquire::http::Proxy "http://192.168.1.187:7897";
```

需要删除该文件或注释掉相关内容，避免后续 apt 使用代理。

## 自动安全更新（可选）

若安装时未选择自动更新，可手动开启：

```bash
sudo apt install unattended-upgrades
sudo dpkg-reconfigure unattended-upgrades
# 选择 Yes
```

## 快速打开终端

```bash
gnome-terminal
```
![debian-terminal]({% staticBase %}{% endstaticBase %}/debian/debian.png)

## 显示系统托盘图标

```bash
sudo apt install gnome-shell-extension-appindicator
```

重启后，启用扩展：

```bash
gnome-extensions list
gnome-extensions enable ubuntu-appindicators@ubuntu.com
```

## 安装 FileZilla（FTP 客户端）依赖

```bash
sudo apt install libgtk2.0-0
```


注：文本转markdown:[deepseek](www.deepseek.com)
