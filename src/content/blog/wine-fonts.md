---
title: Wine 下的字体缺失、不好看
publishDate: 2025-12-02
description: 安装字体解决Wine下字体缺失、不好看的问题
tags:
  - Wine
  - Problem
language: 'Chinese'
---


## 手动安装字体
1. 从网上找Windows的所有字体文件

参考链接：
[](https://www.bilibili.com/opus/602881200681961500)

2. 将其复制入wine的字体目录下 `~/.wine/drive_c/windows/Fonts/`


## 使用 Winetricks 安装全字体包

使用 `winetricks allfonts` 安装尽可能全面的字体，如果在安装某一字体时遇到终端卡死，可以在其他终端使用 `wineserver -k` 来强制结束 Wine 服务，Wine 会跳过该字体的安装

### 一些其他可用的字体
全字体可能太大，可以选择一些比较重要的字体安装
``` bash
winetricks corefonts
winetricks cjkfonts
winetricks arial tahoma simsun
```

## 启用字体平滑

使用 `winetricks fontsmooth=rgb` 开启字体平滑

## 参考
[Wine - Arch Wiki](https://wiki.archlinuxcn.org/wiki/Wine)