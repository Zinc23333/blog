---
title: Wine调用Nvida显卡玩游戏 
publishDate: 2025-10-18
description: '使用一些环境变量使得Wine调用Nvida显卡'
tags:
  - Wine
  - Problem
language: 'Chinese'
---

## 问题描述

在 ArchLinux 下使用系统 Wine 运行游戏时，游戏调用Intel核显，卡顿严重。
使用 `prime-run` 命令，第一次蓝屏，后续该命令无反应。

## 解决方案

增加环境变量 `__NV_PRIME_RENDER_OFFLOAD=1` 和 `__GLX_VENDOR_LIBRARY_NAME=nvidia` 运行游戏，如：

```bash
WINEFSYNC=1 __NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME=nvidia wine ./Duckov.exe
```