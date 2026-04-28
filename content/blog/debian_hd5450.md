---
title: Debian 下 HD5450 显卡开启 B 站硬解 Debian hardware video encoding
description: This is a post on My Blog about agile frameworks.
date: 2026-04-26
tags: Debian
---
# Debian 下 HD5450 显卡开启 B 站硬解

## 0. 缘起

之前用NVIDIA卡太捞，一点硬解都没有，看视频极卡，还不如集呈，所以另找一张。

最后挑了一张 AMD 的 HD5450，因为 Linux 下 A 卡的开源驱动 radeon 集成得最好，不用折腾闭源驱动。

这是一张 2010 年的入门卡。

## 1. 驱动安装：其实啥都不用装

把 HD5450 插上去，Debian 13 启动后就能进桌面。但是只有640\*480。

内核自带 radeon 驱动，但默认缺少非自由固件，需要手动安装：

```bash
sudo apt install firmware-amd-graphics
sudo reboot
```

查看驱动信息：

```bash
glxinfo | grep "OpenGL renderer"
# 输出：AMD CEDAR (DRM 2.50.0, LLVM 15.0.6)
```

重启后，vainfo 应该能识别 VA-API 支持：

```bash
vainfo
# 输出一堆 VAProfileH264... 说明 H.264 硬解可用
```

## 2. 播放器硬解：mpv 一把过

先用本地文件测试硬解：

```bash
mpv --hwdec=auto 某个视频.mp4
```

终端出现 Using hardware decoding (vaapi).，成功！

## 3. 浏览器卡成 PPT？Firefox 故意黑名单

本以为万事大吉，打开 B 站随便点个视频——卡！掉帧！radeontop 里 Graphics pipe 冲到 100%。

在about:config里把这个改成enbaled，仍然不硬解。

```
media.hardware-video-decoding.force-enabled
```

一查 about:support，看到一行：

```
FEATURE_HARDWARE_VIDEO_DECODING_NO_R600  blocklisted
```

Firefox 把 R600 驱动（也就是 HD5450 用的驱动）直接拉黑了，即使你在 about:config 里强行打开 media.ffmpeg.vaapi.enabled，底层也会否决。这是 Firefox 为了稳定性故意为之——老卡硬解可能导致 GPU 重置。

于是尝试 Chromium（Chrome），：

```bash
google-chrome-stable
```

视频流畅了！ 但新问题来了：窗口模式下，鼠标一动就疯狂闪烁，全屏却正常。这是 Wayland 下老显卡与 Chrome 合成器的兼容问题。

## 4. 决战闪烁：X11 后端拯救一切

查了一圈，发现强制 Chrome 使用 X11 后端可以根治闪烁：

```bash
google-chrome-stable --ozone-platform=x11
```

完美！不闪了，硬解也正常。

但总不能每次都在终端输命令吧？我需要一个一劳永逸的启动快捷方式。

## 5. 创建带参数的 Chrome 启动器（不覆盖系统更新）

Debian 的 GNOME 桌面下，不能直接在桌面上放图标（默认没开桌面图标）。我决定在应用程序菜单里新增一个启动项。

### 步骤

警告：以下方式我没有尝试过，我用的是

```bash
@debian:~$ mkdir -p ~/.local/share/applications
cp /usr/share/applications/google-chrome.desktop ~/.local/share/applications/
@debian:~$ nano ~/.local/share/applications/google-chrome.desktop

@debian:~$ update-desktop-database ~/.local/share/applications/
```

来改，这样会我怕chrome更新之后有问题，所以最后放弃。

```bash
rm ~/.local/share/applications/google-chrome.desktop
```

只写了个脚本：

```sh
#!/bin/bash
/usr/bin/google-chrome-stable --ozone-platform=x11
```

也可以alt+f2来输入命令。或者设置快捷键。

GNOME也可以设置为用Xrog，登录的时候点右下角设置使用Xrog即可。

下面是Deepseek写的，请自行判断

1. 在用户目录创建自定义 .desktop 文件：

   ```bash
   nano ~/.local/share/applications/chrome-x11.desktop
   ```
2. 填入以下内容：

   ```desktop
   [Desktop Entry]
   Type=Application
   Name=Chrome (X11)
   Comment=使用 X11 后端启动 Chrome，避免老显卡闪烁
   Exec=/usr/bin/google-chrome-stable --ozone-platform=x11 %U
   Icon=google-chrome
   Terminal=false
   Categories=Network;WebBrowser;
   StartupWMClass=Google-chrome-stable
   MimeType=text/html;text/xml;application/xhtml+xml;
   ```
3. 刷新桌面数据库：

   ```bash
   update-desktop-database ~/.local/share/applications/
   ```
4. 按 Super 键（Windows 徽标），搜索 “Chrome (X11)”，右键 → 添加到收藏夹，它就会出现在左侧任务栏（如果安装了 Dash to Dock 扩展的话）。
5. 也可以设置一个全局快捷键：系统设置 → 键盘 → 自定义快捷键，命令填 /usr/bin/google-chrome-stable --ozone-platform=x11，绑定比如 Super+C。

> 为什么不用直接修改系统的 google-chrome.desktop？  
> 因为系统文件会被 Chrome 更新覆盖。新增一个独立命名的启动项，既保留原版，又不会被自动更新干掉。

## 6. 总结：最终成果

- 显卡：AMD HD5450，开源驱动 radeon + firmware-amd-graphics
- 系统：Debian 13 (trixie) + GNOME (Wayland)
- 硬件解码：mpv 和 Chrome 均可用
- 浏览器：Chrome 通过 --ozone-platform=x11 强制 X11 后端，彻底解决闪烁
- 启动方式：在应用程序菜单里增加了专用启动器，一键运行，不依赖终端

### 附：几个实用命令

```bash
# 检查 VA-API 状态
vainfo

# 实时查看显卡各模块负载（不包括 UVD）
sudo radeontop

# 用 mpv 硬解 H.264 视频
mpv --hwdec=auto 视频文件.mp4

# 用 mpv 直接播放 B 站视频（需安装 yt-dlp 并导出 cookie）
mpv --hwdec=auto --cookies --cookies-file=cookies.txt "https://www.bilibili.com/video/xxx"
```

## 7. 心路小结

一张十几年前的亮机卡，在 Linux 下依然能发光发热，前提是愿意折腾那么几个小时。最大的坑不在老卡本身，而在于现代浏览器对旧驱动的“过度保护”——Firefox 的稳定优先让人又爱又恨，Chrome 的硬解开关又藏得很深。

如果你也有一张类似的老 A 卡（HD 5000/6000 系列），希望这篇记录能帮你少绕几个弯。最后补一句：铭瑄的卡，芯片还是 AMD 的，所以驱动不看铭瑄看 AMD。 铭瑄只是品牌名。

---

EOF