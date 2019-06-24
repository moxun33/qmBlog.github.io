---
title: MAC环境下用Clover制作原版安装U盘
date: 2016-12-22 10:02:45
tags:
	 - MAC OSX86
	 - Clover
	 - U盘
---
+ 本文介绍如何制作黑苹果的U盘镜像
 <!-- more -->

###一、模板电脑的硬件概览
![enter image description here](https://app.yinxiang.com/shard/s43/res/690f6ffc-8e3f-4861-8988-0657d4ec5d28)
###二、准备工作
1、mac 系统环境，白苹果、黑屏或虚拟机都行。
2、UniBeast 工具（到https://www.tonymacx86.com上下载）
3、 MultiBeast 工具(官网 http://www.multibeast.com/)
4、 mac 系统安装镜像。
5、8G以上的 U盘。

###三、制作过程
1、注册 App ID，登录 App Store，搜索macOS Sierra ，下载系统安装装包，或者到网上下载 DMG 镜像，解压出安装包。（两者基本没什么区别）。

2、系统下载完成后，把系统安装包复制到Application(应用程序)目录下（从 App Store 下载的跳过此步）。

3、把系统的语言设置成英语，UniBeast 工具只支持英语环境。点击电脑左上角的苹果标志=》选择“系统偏好设置”=》选择“语言和地区”=》选中“英语”网上拉，如图：

然后注销用户，重新登录，即可把系统语言更改为英语。
![enter image description here](https://app.yinxiang.com/shard/s43/res/4ed33835-22b1-4cfc-9dd4-141be57479fe)
4、格式化 U盘
（1）插入 U盘，打开“磁盘工具”，选中目标 U盘
![enter image description here](https://app.yinxiang.com/shard/s43/res/8b90d831-b44e-4e2c-bf36-1da48cc16089)
（2） 点击 Erase 按钮，命名 USB 的名称（稍后可以重命名），格式化的格式选择  OS X Extended (Journaled) ，分区选择  GUID Partition Map
![enter image description here](https://app.yinxiang.com/shard/s43/res/3a955afe-1e77-4047-97b1-5c315706b604)
（3）点击 Erase ，格式化完成后退出。

5、U盘制作
（1）打开 UniBeast 工具，点击 Continue, Continue, Continue, Continue, Agree
（2）在目标选择处，选择 USB ，然后选择 Continue 。
![enter image description here](https://app.yinxiang.com/shard/s43/res/0115fea2-e8c4-451f-83f5-c678594b7d37)
（3）在选择安装系统处，选择 Sierra 然后点击 Continue 。
（4） 在启动项选择界面选择 UEFI Boot Mode（电脑主板支持 EFI） 或者Legacy Boot Mode.
https://app.yinxiang.com/shard/s43/res/8b3d355f-10a6-4059-b1d4-8dc986d7d913
（5）（可选）在显卡配置界面选择你需要的配置，然后点击 Continue
（6）验证安装选项，然后点击 Continue，输入系统密码，点击 Install。
https://app.yinxiang.com/shard/s43/res/7b1d3211-df84-485d-b777-3b4307c048a2
（7）UniBeast 开始制作，整个过程需要花费一定的时间。慢慢等待吧。
https://app.yinxiang.com/shard/s43/res/77124551-59bc-4cdf-86e3-4ec30fc04c63
四、主板设置
Disable Secure Boot
Disable VT for Direct I/O
Set Auto OS Recovery Threshold to OFF
Set SATA Operation to AHCI or Disabled
Optional: Disable Computrace
Optional: Enable Unobtrusive Mode
Optional: Enable USB PowerShare
Optional: Set Fn Lock Options to Lock Mode Enable/Seconda
![enter image description here](https://app.yinxiang.com/shard/s43/res/056bb752-99bc-4a23-9366-4c48034dfecd)