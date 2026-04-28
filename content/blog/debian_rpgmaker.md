---
title: Debian修复RPGmaker启动问题 Debian RPG Maker MV fix
description: This is a post on My Blog about agile frameworks.
date: 2026-04-15
tags: Debian
---
参考文献：
[https://steamcommunity.com/sharedfiles/filedetails/?id=3484375774](https://steamcommunity.com/sharedfiles/filedetails/?id=3484375774)

[https://steamcommunity.com/app/363890/discussions/1/596288191849591818/](https://steamcommunity.com/app/363890/discussions/1/596288191849591818/)

首先到这里下载：
[https://nwjs.io/](https://nwjs.io/)

![debian_rpgmaker_nwjs]({% staticBase %}{% endstaticBase %}/debian/debian_rpgmaker_nwjs.png)

下载后解压。

打开RPG Maker MV文件夹：

/home/用户名/.local/share/Steam/steamapps/common/RPG Maker MV

![debian_rpgmaker_merge1]({% staticBase %}{% endstaticBase %}/debian/debian_rpgmaker_merge1.png)
![debian_rpgmaker_merge2]({% staticBase %}{% endstaticBase %}/debian/debian_rpgmaker_merge2.png)

然后改一下游戏名：在~/Documents/Games/游戏名

package.json
```json
{
    "name": "这里是空的，补上游戏名",
    "main": "index.html",
    "js-flags": "--expose-gc",
    "window": {
        "title": "",
        "toolbar": false,
        "width": 816,
        "height": 624,
        "icon": "icon/icon.png"
    }
}
```

即可。

注：启动steam命令：
```bash
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
steam
```
