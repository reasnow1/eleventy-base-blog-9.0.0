---
title: Fedora安装Deluge并配置UPnP Fedora server install Deluge and UPnP setup
description: This is a post on My Blog about agile frameworks.
date: 2026-04-20
tags: Fedora
---
安装Deluge：
```bash
sudo dnf list | grep deluge
```
可以看到有deluge、deluge-common、deluge-console、deluge-gtk等，我们选择：
```bash
sudo dnf install deluge-daemon deluge-web
```
修改配置，锁定imcoming端口：
```bash
sudo vim /var/lib/deluge/.config/deluge/core.conf
```
```config
    "listen_interface": "",
    "listen_ports": [
        6881,
        6891
    ],
```
改为
```config
    "listen_interface": "0.0.0.0:6881, [::]:6881",
    "listen_ports": [
        6881,
        6881
    ],
```
```bash
sudo systemctl status deluge-daemon
```
目前是dead状态。
```bash
sudo systemctl enable deluge-daemon
sudo systemctl start deluge-daemon
sudo systemctl start deluge-web
```
最后一条：启动Web服务。

检查防火墙：
```bash
sudo firewall-cmd --state
sudo firewall-cmd --list-ports
```
打开防火墙：
```bash
sudo firewall-cmd --add-port=8112/tcp --permanent
sudo firewall-cmd --reload
```
现在可以访问http://服务器IP:8112/来打开了。默认密码deluge。

可选：不知是否有用：打开这个防火墙端口：
```bash
sudo firewall-cmd --remove-port=6881/tcp --permanent
sudo firewall-cmd --remove-port=6881/udp --permanent
sudo firewall-cmd --reload
```
检查端口状态：
```bash
sudo ss -tuln | grep 6881
```
应该看到有IPV4，IPV6的，说明应该都打开了。

此时upnp仍无效，被fedora自身防火墙拦截。添加富规则：
```bash
sudo firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.18.2" accept' --permanent
sudo firewall-cmd --reload
sudo firewall-cmd --list-rich-rules
```
source是openwrt的地址。最后一条显示防火墙配置。

检查：
```bash
sudo dnf install miniupnpc
upnpc -l
```
这样应该是可以upnp到了。不过，仍失败failed with code 501 (Action Failed)，要配置openwrt（编译时选择或安装luci-app-upnp）。

首先ipv6：
```bash
vim /etc/config/firewall
```
在后面加，或者luci里面按一样的加也得：
```config
config rule
	option src 'wan'
	option dest 'lan'
	option name 'allow-deluge'
	option dest_port '6881'
	option target 'ACCEPT'
	list dest_ip 'ipv6地址（短）'
	list dest_ip 'ipv6地址（长）'
	list dest_ip 'ipv4地址'
```
直接双管齐下。

此时ipv4仍然不正常，查看日志：
```bash
logread -e miniupnpd
```
提示：
```info
Sun Apr 19 08:10:09 2026 daemon.warn miniupnpd[3583]: IGNORED : private/reserved address 10.102.100.136 is not suitable for external IP
```
参考文献：

[https://www.bilibili.com/read/cv40623028](https://www.bilibili.com/read/cv40623028)

修改：
```bash
vim /etc/config/upnpd
```
```config
config upnpd 'config'
	...
        option external_ip '公网IP'
	option secure_mode '1' 
```
加使用公网的IP这一行
```bash
/etc/init.d/miniupnpd restart
```
此时fedora重启一下deluge
```bash
sudo systemctl restart deluge-daemon
```
现在应该正常了。但是，终究只是个安慰，canyouseeme.org不能检测到。

注：之前用TP-LINK路由器的时候，居然可以打洞到公网IP上，现在无论如何设置均无法达到那样的效果， STUN也不行，因此放弃