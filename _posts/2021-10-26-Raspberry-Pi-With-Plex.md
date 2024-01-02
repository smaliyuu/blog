---
layout: post
title: 树莓派4B+Plex+U盘的配置说明
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

今天把吃灰的树莓派又翻了出来，准备配置下家庭媒体中心。

<!--more-->

## 重新刷入树莓派系统
刷机装系统没什么可说的，树莓派官网提供了刷入的软件，只要选择好系统镜像，直接开始就可以了。随后在boot里面放入ssh空文件，开启树莓派ssh功能，rasp-config配置vnc和默认输入分辨率。

## 配置usbmount
usbmount可以实现usb设备得自动加/卸载，也就是说可以像windows系统那样，插上就能用，拔了自动卸载。
```
# 安装usbmount
sudo apt-get install usbmount

# 修改配置文件，不然中文乱码
sudo mousepad /etc/usbmount/usbmount.conf

# 其中MOUNTOPTIONS，需要配置utf8字符集
MOUNTOPTIONS="iocharset=utf8,sync,noexec,nodev,noatime,nodiratime"

# FS_MOUNTOPTIONS，需要配置权限，否则只能读不能写
FS_MOUNTOPTIONS="-fstype=vfat,umask=0000"
```

配置完重启树莓派，之后就能随意插拔了。我这里使用的是fat32格式的U盘，亲测没问题。

## 还可能需要配置systemd-udevd.service
好像新版本的设备需要额外配置下udevd服务，下面是我引用的：[Raspberry 4 usbmount not working
](https://raspberrypi.stackexchange.com/a/100375)

    I faced the same issue with Raspberry Pi 4 and Raspbian Buster, the solution for me was to modify the following file:

    /lib/systemd/system/systemd-udevd.service
    Before: PrivateMounts=yes
    After: PrivateMounts=no

## 安装plex添加媒体库
去官网下载32位ubuntu安装包，dpkg安装即可。