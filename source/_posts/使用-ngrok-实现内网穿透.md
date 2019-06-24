---
title: 使用 ngrok 实现内网穿透
date: 2017-04-04 00:51:47
tags:
	 - ngrok
	 -  内网穿透

---
+ 由于平时需要进行微信接口开发，最让人头疼的是微信的接口配置一定要是公网 IP；我在本地编码了，每次都要传到服务器进行调试，不仅开发效率低，而且无法定位 BUG 。本文将介绍如何从这种困扰中走出，让我们在本地轻松的进行微信调试。
 <!-- more -->

网上有很多 ngrok 公共端，用户可以直接使用。如果还不知道什么的 ngrok；请参考[ngrok](http://baike.baidu.com/link?url=DqWJtVr6syeLYhxRUAnUiYtFIpxgZdZUm7wx5JDfGgTxT-tqr_iOn-9Gv7S7xc4eZ3D44BS_lNmOAhombkkVZq)
## 本文使用的 ngrok 服务方是[http://ittun.com/](http://ittun.com/)

#### 1、前往[http://ittun.com/](http://ittun.com/)下载对应操作系统的客户端。

#### 2、下载完成后，解压得到可执行文件。Windows 系统比较简单，直接双击startup.bat运行即可;
命令启动：
```bash

linux: ./ngrok [port] (tcp: ./ngrok -proto=tcp 22) 
windows: ngrok [port] (tcp: ngrok -proto=tcp 22)
```
更多:ngrok --help

#### 3、使用二级域名。经过我的测试，其他很多的 ngrok 公共端都需要注册登录而且限制使用自定义二级域名， 这是我找到最好的提供商了。
直接运行下面代码：
```bash
./ngrok --subdomain [demo] 80
```

#### 4、当然也可以指定配置文件
```php
./ngrok -config ittun.yml start [proname]
```
####  5、如果嫌以上不走麻烦，不妨写一个简单的 SHELL 脚本。每次启动只要运行该脚本即可。
（1）创建 startup.sh 文件。
在里面写入 
```bash
 	#! /bin/bash
   ./ngrok  -subdomain mywechat 80
   ./ngrok -config config.yml start
```

(2) 每次需要启动 ngrok 服务的时候，直接运行
```bash
sh startup.sh
```

（3）、运行效果
![](https://ooo.0o0.ooo/2017/04/04/58e28206363be.png)

![](https://ooo.0o0.ooo/2017/04/04/58e2829f1f2d9.png)

#### 6、测试微信接口配置
![](https://ooo.0o0.ooo/2017/04/04/58e283e8953b2.png)
配置成功。然后就专注编码吧！
