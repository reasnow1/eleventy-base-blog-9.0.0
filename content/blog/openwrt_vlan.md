---
title: OpenWrt 设置单线复用 OpenWrt VLAN Setup
description: This is a post on My Blog about agile frameworks.
date: 2026-04-11
tags: OpenWrt
---

# 家庭网络改造记录

## 一、材料清单
- 网线若干
- 玩客云（OneCloud）
- 绿联 VLAN 交换机
- TP-Link 路由器（作为 AP）
![vlan_switch]({% staticBase %}{% endstaticBase %}/openwrt/vlan_switch.jpg)

---

## 二、玩客云刷 OpenWrt（适配版本 24.10.6）

### 1. 准备编译环境
- Fork 项目：`https://github.com/lxiaya/openwrt-onecloud`
- 下载 `targets` 文件夹
- 安装编译依赖：[OpenWrt 官方文档](https://openwrt.org/docs/guide-developer/toolchain/install-buildsystem)
- 不同系统的安装命令不同
- 设置代理（可选）：
  ```bash
  export http_proxy=http://127.0.0.1:7897
  export https_proxy=http://127.0.0.1:7897
  ```

### 2. 克隆 OpenWrt 并切换版本

目前，lxiaya适配的版本为24.10，所以
```bash
git clone https://github.com/openwrt/openwrt.git
cd openwrt
git pull
git checkout v24.10.6
```

### 3. 合并 targets 文件夹
将之前下载的 `targets` 复制到 `openwrt` 目录中，直接合并。

这样，待会就会出现玩客云的选择。

### 4. 更新 feeds 并配置
```bash
./scripts/feeds update -a
./scripts/feeds install -a
make menuconfig
```

**必须勾选的选项：**
- Target: `amlogic onecloud`（出现后选择）
- `Utilities` → `haveged`（避免开机慢）
- `LuCI` 默认居然没有，补上
- `LuCI` → `Protocols` → `luci-proto-ppp`（选luci后一般会默认选这个）
- `LuCI` → `Apps` → `luci-app-UPnP` 需要UPnP的话选这个
- `Network` → `SSH` → `openssh-sftp-server`（传文件）
- 弄好后，保存，出来，把.config文件内容复制，到刚才fork的项目内


### 5. 修改默认 IP（可选）
编辑 `op2.sh`，找到：
```bash
sed -i "s/192.168.1.1/10.0.0.2/" package/base-files/files/bin/config_generate
```
将 `10.0.0.2` 改为你想要的 IP（本文后续使用 `192.168.18.2`）。

另外，原作者在op1.sh和op3.sh内写有一些东西，我全部注释（不要）了。

### 6. GitHub Actions 编译
- 修改 `.github/workflows/openwrt.yml`，将 Release 上传改为 Artifact 上传（避免 token 问题）：
  原代码：
  ```yaml
  - name: Upload firmware to Releases
    uses: softprops/action-gh-release@v2
    if: steps.tag.outputs.status == 'success' && !cancelled()
    with:
      tag_name: ${{ steps.tag.outputs.release_tag }}
      body_path: release.txt
      files: ${{ env.FIRMWARE }}/*
      token: ${{ secrets.TOKEN }}
  ```
  修改：
  ```yaml
  - name: Upload firmware as artifact
    uses: actions/upload-artifact@v4
    with:
      name: firmware
      path: ${{ env.FIRMWARE }}/*
  ```
- 现在可以点击github的workflow开始编译了。
- 编译结束后，下载，解压：`openwrt-amlogic-meson8b-thunder-onecloud-ext4-emmc.burn.img.xz`
- 注：其实不改这个，也会上传一个很像产物的文件，不过，我没有深入研究

### 7. 刷入玩客云
参考教程：[S805 刷机指南](https://www.ecoo.top/docs/tutorial-basics/s805)（步骤略）

---

## 三、绿联交换机 VLAN 配置

### 1. 初始访问
- 交换机默认管理 IP：`192.168.18.8`
- 电脑网口接交换机 3/4/5 口，设置静态 IP `192.168.18.9`或者其他同网段
- 浏览器访问 `192.168.18.8`

### 2. VLAN 划分
| 端口 | 模式      | PVID | Allowed VLANs | 用途         |
|------|-----------|------|---------------|--------------|
| 1    | Access    | 10   | -             | 接光猫 (WAN) |
| 2    | Trunk     | 1    | 10,20         | 接玩客云     |
| 3/4/5| Access    | 20   | -             | 接 AP/电脑   |

- 默认情况下，会只有一个PVID，就是1。
- 新增 VLAN 10（名称 WAN）和 VLAN 20（名称 LAN）
- 按上表配置后保存，交换机立即失联（正常现象）
- 如果不想失联，可尝试将管理VLAN改为20
- 我没改，所以，失联了，这部分无截图

---

## 四、OpenWrt 网络配置

### 1. 初始登录
- 连接玩客云，电脑
- 电脑保持DHCP可以拿IP
- 访问默认 IP（前面设置的）
- 设置 root 密码

### 2. 修改 `/etc/config/network`
SSH连接后：

```bash
vim /etc/config/network
```

前面的loopback和globals可以不理。

修改内容如下（核心部分）：
```ini
config device
    option name 'br-lan'
    option type 'bridge'
    list ports 'eth0'
    list ports 'eth0.20'

config interface 'lan'
    option device 'br-lan'
    option proto 'static'
    option ipaddr '192.168.18.2'
    option netmask '255.255.255.0'
    option ip6assign '60'
    option ifname 'eth0.20'

config interface 'wan'
    option proto 'pppoe'
    option username '宽带账号'
    option password '宽带密码'
    option ipv6 'auto'
    option device 'eth0.10'
```

### 3. 重启网络
```bash
/etc/init.d/network restart
```
此时OpenWrt会失联。稍等几秒，设置生效后即可进行物理连接。

---

## 五、物理接线

```
光猫 ---- 交换机端口 1 (VLAN 10)
玩客云 -- 交换机端口 2 (Trunk)
其他设备 - 交换机端口 3/4/5 (VLAN 20)
```

- openwrt默认的DHCP范围是100－250
- 所以，路由器关闭DHCP，IP设置为192.168.18.1－99应该都行
- 刚才openwrt是.2，所以不要用.2。
- wan设置可以直接不鸟
- 就可以当AP用了。

---

## 六、使用

### 1. 使用
- 电脑接345口，可以重新访问luci。
- 打开luci，interface，会有个问你转不转化的，点是
- 然后可以看到多出个Protocol: Virtual dynamic interface (DHCPv6 client)
![vlan_interfaces]({% staticBase %}{% endstaticBase %}/openwrt/vlan_interfaces.png)
- 然后就有IPV6了，虽然我也不知道怎么有的。
- 在DHCP列表内，还可以看到刚才的VLAN交换机CM933，还有IP地址
- 但无法访问。
![vlan_switch_ip]({% staticBase %}{% endstaticBase %}/openwrt/vlan_switch_ip.png)

### 2. 原因分析（Deepseek写的）
- **ipv6**：`wan` 接口设置了 `option ipv6 'auto'`，PPPoE 拨号后运营商通过 DHCPv6-PD 委派前缀，OpenWrt 自动为 LAN 侧分配 IPv6 地址并启用 SLAAC/DHCPv6，内网设备获得 IPv6 地址。
- **交换机管理 IP**：交换机管理接口仍属于 **VLAN 1**，而 Trunk 端口只允许 VLAN 10 和 20，导致管理流量无法到达 OpenWrt。
- **影响**：上网、IPv6 均正常。如需恢复管理，可将交换机的管理 VLAN 改为 20（需重新直连配置），或忽略不管。

---

