---
title: VirtualBox Mac安装一直卡死在验证pkg处
date: 2016-12-18 22:17:21
tags: Mac
---
# 主要现象就是在解压安装包后不能通过验证，一直卡死

## 解决办法
VirtualBox pkg 解压后在***/Volumes/VitualBox/***
进入该文件夹
使用命令
``` Bash
sudo installer -pkg /Volumes/VirtualBox/VirtualBox.pkg -target /
```
## 安装成功
可以看见
``` Bash
MyMac:VirtualBox zl$ sudo installer -pkg /Volumes/VirtualBox/VirtualBox.pkg -target /
installer: Package name is Oracle VM VirtualBox
installer: Installing at base path /
installer: The install was successful.
MyMac:VirtualBox zl$
```

DashBoard中已经有了其图标

