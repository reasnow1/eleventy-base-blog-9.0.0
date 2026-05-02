---
title: Debian代理浏览器 Debian proxy vivaldi
description: This is a post on My Blog about agile frameworks.
date: 2026-05-03
tags: Debian
---
在代理环境下启动浏览器

to run vivaldi under proxy, use this .sh
```sh
#!/bin/bash
export http_proxy=http://127.0.0.1:7897
export https_proxy=http://127.0.0.1:7897
vivaldi
```
