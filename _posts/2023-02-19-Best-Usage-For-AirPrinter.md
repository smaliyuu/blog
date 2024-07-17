---
layout: post
title: Mac添加Windows共享打印机的最佳实践（AirPrinter）
author: SmaliYuu
tags: 技术
excerpt_separator: <!--more-->
---

家里添置了老款的惠普1020Plus打印机，这个打印机只能USB连接，不支持Wifi，只能放在Windows台式机旁边。如果我Mac上的文件需要打印的话，还需要将文件发送到Windows上，比较麻烦。因此整理总结如何在Mac下添加使用Windows上共享的打印机进行打印。

<!--more-->

## 安装AirPrinter
这部分不在赘述，主要就是安装本体和Bonjour服务，这两个是必须的，详情参考[如何在Mac os系统下连接并使用Windows共享打印机教程Air Printer](https://www.bilibili.com/video/BV1ct411u7fw/)。

## 测试iOS手机和Mac是否可以发现打印机
正常来说，安装完就可以在各种苹果设备上发现打印机，并打印文件了。

## 开机静默启动
AirPrinter本身仅仅支持开启启动，如果在设置里勾选开机启动，那么每次开机都会弹出软件的主界面。我门这里不想每次开机都看到AirPrinter的窗口，所以这里使用了Windows里面的计划任务功能。在计划任务里新建任务，命名为AirPrinter；触发器为计算机启动时；操作就是把AirPrinter的执行文件路径放上，然后点完成。在活动任务里找到刚才新建的AirPrinter，右键属性，**勾选不管用户是否登陆都要运行**，这里是关键，这样勾选就不会有窗口界面启动了。由于我使用了Administrator账户，没有密码，所以还勾选了不存储密码。  

## 注意事项
以上操作，AirPrinter程序会以后台任务的形式启动，这一点可以从任务管理器查看。而且不同于最小化程序，这种启动方式，右下角托盘是没有图标的，只能在任务管理器中查看是否启动。
