---
title: 在 sudo PAM 中添加了 Fido2 配置，无法正常调起验证流程
publishDate: 2025-11-17
description: 在为 sudo PAM 添加 Fido2 认证后，插入设备时未质询 PIN 而直接认证失败
tags:
  - Fido2
  - Problem
language: 'Chinese'
---

## 问题描述

在 ArchLinux 下在 sudo 的 PAM 认证配置中添加了 Fido2 的配置后，在刚插入 Fido2 设备后，使用 `sudo xxx` 会出现立刻结束 Fido2 的认证，开始询问用户密码，无法正常使用 Fido2 认证。

配置如下：

``` bash
zinc@zinc ~> cat /etc/pam.d/sudo
#%PAM-1.0
auth      sufficient  pam_u2f.so cue origin=pam://zinc appid=pam://zinc
auth		  include		  system-auth
account		include		  system-auth
session		include		  system-auth
```

出现现象：
``` bash
zinc@zinc ~> sudo ls
Please touch the FIDO authenticator.
[sudo] zinc 的密码：
```

在输出 `Please touch the FIDO authenticator.` 后并不会询问Pin，也不会要求点击设备，而是立刻失败，询问用户密码。



## 解决方案

增加 `pinverification=1` ，要求其质询Pin

总的配置如下：

``` bash
zinc@zinc ~> cat /etc/pam.d/sudo
#%PAM-1.0
auth      sufficient  pam_u2f.so cue pinverification=1 origin=pam://zinc appid=pam://zinc
auth		  include		  system-auth
account		include		  system-auth
session		include		  system-auth
```

## 发现过程

在刚插入Fido2设备时，总是无法在终端中完成 `sudo xxx` 的权限验证，但当我在浏览器或者 `Yubico Authenticator` 完成一次验证流程后，后续就可以正常使用该设备了。其流程上的区别在于，终端中未询问我的PIN，而其他流程中会询问我的PIN。因此推测了是该设备未被解锁导致的。