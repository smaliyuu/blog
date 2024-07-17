---
layout: post
title: 转载·OpenWRT开启802.11k和802.11v
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

文章转载自：[https://blog.mrning.com/2022/07/328/](https://blog.mrning.com/2022/07/328/)

<!--more-->

开启802.11k/v/r将有助于提升无线漫游质量。802.11r的设置可以在官方Luci界面中完成，802.11k/v的设置可以通过修改/etc/config/wireless配置文件完成。  

首先，确保已经安装了完整的 wpad（如wpad-wolfssl 或 wpad-openssl），如果之前只安装了wpad-basic 或 wpad-wolfssl-basic，需要先卸载再安装完整的 wpad。  

然后在/etc/config/wireless配置文件中，为目标wifi-iface添加以下option

```
	option bss_transition '1'
	option wnm_sleep_mode '1'
	option time_advertisement '0'
	option ieee80211k '1'
	option rrm_neighbor_report '1'
	option rrm_beacon_report '1'
```
然后重启设备或重启wpad服务及网络接口。  

![WinFi](../assets/post-images/202401022105.png)

WinFi（如卡加载，将系统时间调为5月前）的扫描结果  
参考：[Setting up usteer and band-steering in OpenWrt](https://openwrt.org/usteer)