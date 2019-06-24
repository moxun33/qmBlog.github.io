---
title: ubuntu64位安装32位依赖库，并安装njit-client
date: 2017-01-23 09:20:01
tags:
	 - Ubuntu

---
+ 自从Ubuntu12.04之后，就移除了32位的库，当时为了安装校园网的客户端，就不得不安装32位的库。
<!-- more -->
###这是很早之前的文章了，当时为了破解学校的网络客户端。
####方法一:
```php
sudo -i
cd /etc/apt/sources.list.d
echo "deb http://archive.ubuntu.com/ubuntu/ raring main restricted universe multiverse" >ia32-libs-raring.list
apt-get update
apt-get install ia32-libs
```



PS:这种方法肯定能安装上ia32-libs，但由于添加的是13.04的源，所以有没有混淆暂时不清楚，
网上也有很多用新立德软件包管理器这样做；够狠，你就这么干。也可以这么安装了ia32-libs后，
把/etc/apt/sources.list.d目录下的ia32-libs-raring.list删掉，然后再sudo apt-get update








####方法二:
切换至中科大的源 http://uestc.edu.cn
 然后
```php
 apt-get update
apt-get install ia32-libs
```
如果终端出现:
```php 
错误 http://archive.ubuntu.com/ubuntu/ raring/main libgd2-xpm i386 2.0.36~rc1~dfsg-6.1ubuntu1  *
  404  Not Found [IP: 91.189.91.23 80] *
错误 http://archive.ubuntu.com/ubuntu/ raring/main libgphoto2-port0 i386 2.4.14-2
* 
  404  Not Found [IP: 91.189.91.23 80] *
错误 http://archive.ubuntu.com/ubuntu/ raring/main libgphoto2-2 i386 2.4.14-2
*
  404  Not Found [IP: 91.189.91.23 80] *---------------------------------------
E: 无法下载 http://archive.ubuntu.com/ubuntu/pool/main/libg/libgd2/libgd2-xpm_2.0.36~rc1~dfsg-6.1ubuntu1_i386.deb  404  Not Found [IP:
* 91.189.91.23 80] *------
E: 无法下载 http://archive.ubuntu.com/ubuntu/pool/main/libg/libgphoto2/libgphoto2-port0_2.4.14-2_i386.deb  404  Not Found [IP: 91.189.91.23 80]*
E: 无法下载 http://archive.ubuntu.com/ubuntu/pool/main/libg/libgphoto2/libgphoto2-2_2.4.14-2_i386.deb  404  Not Found [IP: 91.189.91.23 80]    *
E: 有几个软件包无法下载，您可以运行 apt-get update 或者加上 --fix-missing 的选项再试试？
*
```
自己去网上搜索这三个包,然后安装.
或者使用以下网址:


https://packages.debian.org/wheezy/i386/libgphoto2-port0/download


https://packages.debian.org/wheezy/i386/libgphoto2-2/download


https://launchpad.net/ubuntu/raring/i386/libgd2-xpm/2.0.36~rc1~dfsg-6.1ubuntu1


安装后再执行
```php
apt-get install ia32-libs
```

没有意外应该装上了.

ps:附上缺失的三个包的百度云。。。链接   http://pan.baidu.com/s/1kTJz8k3

原文地址：http://blog.csdn.net/moxxun/article/details/45129833
