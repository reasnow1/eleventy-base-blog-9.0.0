---
title: 安装FreeBSD作网关 FreeBSD install and router setup
description: This is a post on My Blog about leveraging agile frameworks.
date: 2026-04-28
tags: FreeBSD
---
下载地址：
[https://mirrors.nju.edu.cn/freebsd/releases/ISO-IMAGES/15.0/FreeBSD-15.0-RELEASE-amd64-memstick.img.xz](https://mirrors.nju.edu.cn/freebsd/releases/ISO-IMAGES/15.0/FreeBSD-15.0-RELEASE-amd64-memstick.img.xz)

解压后dd到u盘：(官方命令)
```bash
dd if=FreeBSD-15.0-RELEASE-amd64-memstick.img of=/dev/sdd bs=1M conv=sync
```
大概要写6到7分钟。

完成后开始安装，官方的文档写得非常的详细：

[https://docs.freebsd.org/en/books/handbook/bsdinstall/](https://docs.freebsd.org/en/books/handbook/bsdinstall/)

这里提点关键：

- Selecting Components to Install 选择 base-dbg
好像只是提供debug？我怕没东西用，选了）
- Partitioning Choices 选择 Auto (UFS)
不用 Auto (ZFS) 更简单
- Choosing a Mirror 用 https://mirrors.ustc.edu.cn/freebsd/
建议这个，南京大学的，我用了，有问题,会无法下载isc-dhcp
- Selecting Additional Services to Enable 开 ntpd_sync_on_start 
sshd默认开了。开这个同步时间。
- Selecting Hardening Security Options 直接不要。
- Add User Accounts 增加普通用户，加入用户组whell
这样后面可以su。

安装完成

我登录的时候，普通用户不知道为啥没了，要root先登录然后
```sh
adduser
```
才得
# 配置网关
```sh
ifconfig
```
```sh
re0: flags=1008843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST,LOWER_UP> metric 0 mtu 1500
	options=8209b<RXCSUM,TXCSUM,VLAN_MTU,VLAN_HWTAGGING,VLAN_HWCSUM,WOL_MAGIC,LINKSTATE>
	ether 08:9e:01:a2:14:bd
	inet 192.168.1.1 netmask 0xffffff00 broadcast 192.168.1.255
	media: Ethernet autoselect (100baseTX <full-duplex>)
	status: active
	nd6 options=29<PERFORMNUD,IFDISABLED,AUTO_LINKLOCAL>
lo0: flags=1008049<UP,LOOPBACK,RUNNING,MULTICAST,LOWER_UP> metric 0 mtu 16384
	options=680003<RXCSUM,TXCSUM,LINKSTATE,RXCSUM_IPV6,TXCSUM_IPV6>
	inet 127.0.0.1 netmask 0xff000000
	inet6 ::1 prefixlen 128
	inet6 fe80::1%lo0 prefixlen 64 scopeid 0x2
	groups: lo
	nd6 options=21<PERFORMNUD,AUTO_LINKLOCAL>
ue0: flags=1008843<UP,BROADCAST,RUNNING,SIMPLEX,MULTICAST,LOWER_UP> metric 0 mtu 1500
	options=80000<LINKSTATE>
	ether 5c:4d:bf:88:7f:47
	inet 192.168.0.182 netmask 0xffffff00 broadcast 192.168.0.255
	inet6 fe80::5e4d:bfff:fe88:7f47%ue0 prefixlen 64 scopeid 0x3
	inet6 ipv6::::::::: prefixlen 64 autoconf pltime 3600 vltime 3600
	media: Ethernet autoselect
	status: active
	nd6 options=23<PERFORMNUD,ACCEPT_RTADV,AUTO_LINKLOCAL>
```
目前需要手打命令：
```sh
su
echo 'ifconfig_re0="inet 192.168.1.1 netmask 255.255.255.0"' >> /etc/rc.conf
service netif restart re0
```
现在可以电脑网线连笔记本了。

手动设置
- IP 192.168.1.182
- 子关掩码 255.255.255.0
- 网关 192.168.1.1

使用ssh登录。

后面要su的自己su一下，我不记得哪里要su了
```sh
echo 'gateway_enable="YES"' >> /etc/rc.conf
sysctl net.inet.ip.forwarding=1
cat > /etc/pf.conf <<EOF
ext_if = "ue0"      # 外网：USB 共享网络
int_if = "re0"      # 内网：接路由器

set skip on lo0

nat on \$ext_if from \$int_if:network to any -> (\$ext_if)

pass all
EOF
echo 'pf_enable="YES"' >> /etc/rc.conf
service pf start
```
现在开始配置镜像站，注意

即使只用pkg，也要配置port，因为你执行update的时候他会同时更新port的。

- [https://mirrors.ustc.edu.cn/help/freebsd-pkg.html](https://mirrors.ustc.edu.cn/help/freebsd-pkg.html)
- [https://mirrors.ustc.edu.cn/help/freebsd-ports.html](https://mirrors.ustc.edu.cn/help/freebsd-ports.html)

```sh
mkdir /usr/local/etc/
mkdir /usr/local/etc/pkg
mkdir /usr/local/etc/pkg/repos
vi /usr/local/etc/pkg/repos/ustc.conf
```
他这里vi然后按i不显示 - INSERT - 的，但可以输入
```conf
ustc-ports: { 
    url: "https://mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/quarterly",
    mirror_type: "none",
    signature_type: "fingerprints",
    fingerprints: "/usr/share/keys/pkg",
    enabled: yes
}

ustc-ports-kmods: {
    url: "https://mirrors.ustc.edu.cn/freebsd-pkg/${ABI}/kmods_quarterly_${VERSION_MINOR}",
    mirror_type: "none",
    signature_type: "fingerprints",
    fingerprints: "/usr/share/keys/pkg",
    enabled: yes
}

FreeBSD-ports: { 
    enabled: no 
}

FreeBSD-ports-kmods: { 
    enabled: no 
}

```
```sh
vi /etc/make.conf
```
```conf
MASTER_SITE_OVERRIDE?=https://mirrors.ustc.edu.cn/freebsd-ports/distfiles/${DIST_SUBDIR}/
```
现在输入
```sh
pkg update -f
```
即使弄了前面，第一波，也还是会从官网下，慢得一彼。

完成后
```sh
pkg install isc-dhcp44-server
cat > /usr/local/etc/dhcpd.conf <<EOF
subnet 192.168.1.0 netmask 255.255.255.0 {
  range 192.168.1.50 192.168.1.200;
  option routers 192.168.1.1;
  option domain-name-servers 192.168.0.1;
}
EOF
echo 'dhcpd_enable="YES"' >> /etc/rc.conf
echo 'dhcpd_ifaces="re0"' >> /etc/rc.conf
service isc-dhcpd start
```
用中兴F50来提供DNS。

现在可以接交换机使用了。

# 关闭背光

目前没有特别好的方法。

首先加载模块：
```sh
sudo kldload acpi_video
```
以下命令使开机自动加载。
```sh
echo "acpi_video_load=\"YES\"" >> /boot/loader.conf
```
输入
```sh
sysctl hw.acpi.video
```
看到
```sh
hw.acpi.video.tv0.active: 1
hw.acpi.video.crt0.active: 1
hw.acpi.video.lcd0.levels: 100 100 0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88 89 90 91 92 93 94 95 96 97 98 99 100
hw.acpi.video.lcd0.economy: 100
hw.acpi.video.lcd0.fullpower: 100
hw.acpi.video.lcd0.brightness: 100
hw.acpi.video.lcd0.active: 1
```
输入
```sh
sysctl hw.acpi.video.lcd0.brightness=0
```
看到
```sh
hw.acpi.video.lcd0.brightness: 100 -> 0
```
然后还得重启才生效。

最后，并不是100%黑，还是亮一丢丢。

网上找了些方法，不生效。

除非安装个桌面Xrog，但太折腾，不弄了。

我们的deepseek写了自动化的方法：

创建标准的 rc 脚本（更规范）

1. 创建脚本 `/usr/local/etc/rc.d/blank_screen`：
   ```sh
   #!/bin/sh
   # PROVIDE: blank_screen
   # REQUIRE: NETWORKING
   # BEFORE: LOGIN
   # KEYWORD: shutdown

   . /etc/rc.subr

   name=blank_screen
   rcvar=blank_screen_enable
   start_cmd="blank_screen_start"
   stop_cmd=":"

   blank_screen_start()
   {
       /sbin/sysctl hw.acpi.video.lcd0.brightness=0
   }

   load_rc_config $name
   run_rc_command "$1"
   ```

2. 赋予执行权限：
   ```sh
   chmod +x /usr/local/etc/rc.d/blank_screen
   ```

3. 在 `/etc/rc.conf` 中启用：
   ```sh
   echo 'blank_screen_enable="YES"' >> /etc/rc.conf
   ```

注：如果后面犯病没网，就把网线拨了，重启，完全启动后，再插网线。