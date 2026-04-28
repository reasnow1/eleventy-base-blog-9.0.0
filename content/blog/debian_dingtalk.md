---
title: Debian安装钉钉 Dingtalk on Debian
description: This is a post on My Blog about agile frameworks.
date: 2026-04-10
tags: Debian
---

# Debian 安装使用钉钉

- **系统版本**：Debian 13.4  
- **钉钉版本**：8.1.0.6021101

## 1. 安装

从官网下载 `.deb` 包后，使用 `apt` 安装：

```bash
sudo apt install ./com.alibabainc.dingtalk_8.1.0.6021101_amd64.deb
```

安装完成后会出现图标，但**无法直接打开**。

## 2. 查找可执行文件路径

使用 `dpkg -L` 查看包安装的文件列表：

```bash
dpkg -L com.alibabainc.dingtalk
```

输出示例（节选）：

```
/.
/opt
/opt/apps
/opt/apps/com.alibabainc.dingtalk
/opt/apps/com.alibabainc.dingtalk/entries
/opt/apps/com.alibabainc.dingtalk/entries/applications
/opt/apps/com.alibabainc.dingtalk/entries/applications/com.alibabainc.dingtalk.desktop
/opt/apps/com.alibabainc.dingtalk/entries/applications/com.alibabainc.dingtalk_std_int.desktop
/opt/apps/com.alibabainc.dingtalk/entries/autostart
/opt/apps/com.alibabainc.dingtalk/entries/autostart/com.alibabainc.dingtalk.desktop
/opt/apps/com.alibabainc.dingtalk/files
/opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101
...
```

从输出可知，可执行文件位于：

```
/opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101/com.alibabainc.dingtalk
```

## 3. 尝试运行 – 缺少共享库

```bash
/opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101/com.alibabainc.dingtalk
```

错误信息：

```
error while loading shared libraries: libdtfbase.so: cannot open shared object file: No such file or directory
```

**原因**：缺少共享库 `libdtfbase.so`。

## 4. 指定库路径并运行 – 符号冲突

进入钉钉目录并设置 `LD_LIBRARY_PATH`：

```bash
cd /opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101
LD_LIBRARY_PATH=. ./com.alibabainc.dingtalk
```

错误信息：

```
./com.alibabainc.dingtalk: symbol lookup error: /lib/x86_64-linux-gnu/libpango-1.0.so.0: undefined symbol: hb_ot_metrics_get_position
```

**原因**：钉钉自带的 harfbuzz 库与系统库冲突。

## 5. 禁用钉钉自带的 harfbuzz 库

将相关库文件重命名为 `.bak` 以屏蔽：

```bash
sudo mv libharfbuzz.so.0.20301.0 libharfbuzz.so.0.20301.0.bak
sudo mv libharfbuzz.so.0 libharfbuzz.so.0.bak
sudo mv libharfbuzz.so libharfbuzz.so.bak
```

## 6. 再次运行 – 可执行栈错误

```bash
LD_LIBRARY_PATH=. ./com.alibabainc.dingtalk --no-sandbox
```

错误信息：

```
Load /opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101//dingtalk_dll.so failed! Err=/opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101//dingtalk_dll.so: cannot enable executable stack as shared object requires: Invalid argument
```

**原因**：钉钉自带的 `dingtalk_dll.so` 被标记为需要可执行栈，而 Debian 的内核/动态链接器默认禁止该操作（安全原因）。

## 7. 使用 `patchelf` 清除可执行栈标记

安装 `patchelf` 并修复：

```bash
sudo apt install patchelf
sudo patchelf --clear-execstack /opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101/dingtalk_dll.so
```

## 8. 成功运行

再次执行：

```bash
cd /opt/apps/com.alibabainc.dingtalk/files/8.1.0-Release.6021101
LD_LIBRARY_PATH=. ./com.alibabainc.dingtalk --no-sandbox
```

此时钉钉可以正常打开。关闭命令行后，也可以通过系统菜单栏中的图标启动。

## 9. 感想

> 大家都笑 QQ 和微信是浏览器（因为用 Electron 来开发），但人家跨平台是真能用。钉钉这 Qt/C++，性能是高了，依赖是真地狱，官方还不提供 AppImage。

## 10. 快速修复命令（一键解决）

若在新安装的 Debian 系统上按上述步骤安装钉钉后无法打开，可直接执行以下命令完成所有修复（假设已通过 `sudo apt install ./com.alibabainc.dingtalk_*.deb` 完成安装）：

```bash
# 1. 进入钉钉实际安装目录（版本号可能不同，使用通配符）
cd /opt/apps/com.alibabainc.dingtalk/files/*-Release.*/

# 2. 禁用钉钉自带的 harfbuzz 库（避免与系统库冲突）
sudo mv libharfbuzz.so.0.20301.0 libharfbuzz.so.0.20301.0.bak 2>/dev/null
sudo mv libharfbuzz.so.0 libharfbuzz.so.0.bak 2>/dev/null
sudo mv libharfbuzz.so libharfbuzz.so.bak 2>/dev/null

# 3. 安装 patchelf 并清除 dingtalk_dll.so 的可执行栈标记
sudo apt update && sudo apt install -y patchelf
sudo patchelf --clear-execstack dingtalk_dll.so

# 可选 - 4. 修改系统桌面快捷方式，添加环境变量和 --no-sandbox 参数
#DESKTOP_FILE="/opt/apps/com.alibabainc.dingtalk/entries/applications/com.alibabainc.dingtalk.desktop"
#CURRENT_DIR=$(pwd)

#sudo sed -i "s|^Exec=.*|Exec=env LD_LIBRARY_PATH=$CURRENT_DIR $CURRENT_DIR/com.alibabainc.dingtalk --no-sandbox|" $DESKTOP_FILE

# 可选 - 5. 更新桌面数据库（可选，使修改立即生效）
#sudo update-desktop-database

echo "修复完成！现在可以通过系统菜单或桌面图标正常启动钉钉了。"
```

> **说明**：  
> - 上述命令会自动定位钉钉的安装目录（版本号部分使用通配符 `*`）。  
> - 本人没有执行第4和第5步，即可直接点击图标即可运行。4和5是AI补上的，请自行决定是否使用。
> - （AI补充）如果系统中存在多个钉钉 `.desktop` 文件，可根据需要重复第 4 步修改其他文件（如 `com.alibabainc.dingtalk_std_int.desktop`）。  
