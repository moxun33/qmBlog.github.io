---
title: 树莓派3安装 Docker记录
date: 2017-02-18 23:14:19
tags:
	 - Docker
	 
	 - 树莓派 raspberrypi
---
+ 今天有空就查询一下在树莓派如何安装 docker，边安装边记录了一下。
 <!-- more -->

树莓派3下安装 docker 记录

关于 docker 的介绍请自行查询  https://www.docker.com/

直接在树莓派3运行命令

![](https://ooo.0o0.ooo/2017/02/18/58a866da23ed6.png)

很好, 提示我已经安装了最新的版本, 那就运行一下吧:

![](https://ooo.0o0.ooo/2017/02/18/58a8677b77222.png)

奇怪了, 说是找不到命令, 那就看看安装到哪里去了:
```php
root@raspberrypi:~# whereis docker
docker:
```
没有找到关于 docker 的可执行文件, 而用 whereis 命令查找 git  的输出如下:

oot@raspberrypi:~# whereis git
git: /usr/bin/git /usr/share/man/man1/git.1.gz
那么用 find 搜索一下:
```php
root@raspberrypi:~# find / -name docker
/usr/share/menu/docker
/usr/share/doc/docker

```
看起来都不是一个独立的软件包的样子, 那么我们用 apt-cache search 命令看看是否还有其他以 docker 为名的软件，发现 docker 跟 raspbian 系统下的一个系统托盘应用重名了, 我们要找的容器 docker 是 docker.io, 那么试着安装一下:

![](https://ooo.0o0.ooo/2017/02/18/58a867cf78e39.png)

 顺利安装完毕, 测试是否成功:
 ```php
root@raspberrypi:~# docker --version

Docker version 1.3.3, build d344625
```


 Good, 现在已经成功地在 raspi3 系统上安装了一个 docker, 之后就可以在 docker 进行操作了.

启动我们的 docker 守护进程:
```php

root@raspberrypi:~# docker -d

2017/02/18 14:22:25 docker daemon: 1.3.3 d344625; execdriver: native; graphdriver:
[143dcfc4] +job serveapi(unix:///var/run/docker.sock)
2017/02/18 14:22:25 pid file found, ensure docker is not running or delete /var/run/docker.pid
```



