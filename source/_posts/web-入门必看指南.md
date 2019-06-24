---
title: web 入门必看指南
date: 2017-02-20 21:24:25
tags:
	 - web 
	 - 入门指南
---
+ 整理本人从零开始学 web 开发的资料，和一些学习心得。
 <!-- more -->
### 一.   开发环境搭建

1.     wndows下可以使用集成的web服务器，推荐wampserver

下载地址：http://pan.baidu.com/s/1qWA0KRq 

（1）Wampserver的安装，一路next下去。

（2）过程中出现选择默认浏览器的操作，可以选择取消。

（3）wampserver安装完成后，点击打开。找到www目录，这就是网页文件存放的地方。

（4）关于phpmyadmin的使用，稍后再数据库mysql的部分介绍。

（5）Windows的数据库管理工具还可以使用 navicat 软件。使用在后面介绍。

（6）访问本地网站，在浏览器输入http://localhost 

2.     在linux下，以ubuntu系统为例子。（推荐使用ubuntu）

(1)安装很简单，打开终端。输入 
```php
sudo apt-get install  apache2 mysql-server mysql-client php5 php5-gd php5-mysql
```
(2) 调整默认Web目录权限
```php
 sudo chmod 777  -R /var/www/html
```
(3) Apache 配置

启用 mod_rewrite 模块:终端命令：
```php
sudo a2enmod rewrite
```
(4) 重启Apache服务器：
```php 
sudo /etc/init.d/apache2 restart
```
(5)数据库管理软件 phpmyadmin 安装
```php
sudo apt-get install phpmyadmin
```
（6） 安装时要求选择Web server：apache2或lighttpd，选择apache2，设置Mysql数据库密码。然后连接phpmyadmin与apache2，以我的为例：www在/var/www/html，phpmyadmin在/usr/share /phpmyadmin目录，用命令：
```php
sudo ln -s /usr/share/phpmyadmin /var/www/html 建立连接。
```
（7）phpmyadmin测试，在浏览器中打开http://localhost/phpmyadmin

更多关于phpmyadmin的用法在数据库部分介绍。

（8）访问本地网站在浏览器输入http://localhost 

### 二. 编辑器的选择与使用

1. 如果有编程的经历，可以使用自己熟悉的编辑器或者IDE

2. 前端开发，可以使用vim. sublime. notepad++. editplus。

3. php开发，可以使用 上面所说的编辑器。也可以用php的IDE，例如PHPstorm。

4. 各编辑器的使用可以参考网上的一些教程。只要一个原则，适合自己的编辑器才是最好的编辑器。

5. 各编辑器的下载大家自己从网上下载。

### 三. SVN的使用，下载地址 http://pan.baidu.com/s/1bntqrk7  ，此针对windows系统。

1. SVN是Subversion的简称，是一个开放源代码的版本控制系统，相较于RCS. CVS，它采用了分支管理系统，它的设计目标就是取代CVS。互联网上很多版本控制服务已从CVS迁移到Subversion。

2. 更多关于svn的使用教程，参考 http://pan.baidu.com/s/1hq7ZqIw  上面的文档。

3. svn的好处就是一个团队的协作开发得到更好的控制，而且防止不可抗因素的出现导致源代码丢失。当然还可以使用github。

### 四. ubuntu系统下SVN的安装与使用

1.  安装svn客户端：
```php
apt-get install subversion，然后根据提示一步一步，就完成了svn
```php

的安装。当然，也可以源码安装svn，下载subversion一个最新版本的源码包，解压之后就可以安装了。 

2.  新建一个目录，cd 到新建目录下，将文件checkout到本地目录：
```php
svn checkout 
```
svn://192.168.100.249/server ，按提示输入相应的用户名和密码。 3.  可以输入：svn help 来查看svn提供的命令。 

4.  往版本库中添加新的文件，例如: svn add *.c (添加当前目录下所有的c文件)。 5.  将改动的文件提交到版本库，例如：
```php
svn commit -m “add test file for my test“  test.c 。 
```

6.  删除文件，例如：
```php
svn delete svn://192.168.100.249/server/common/test.c -m “delete test file” 。 
```
7.  如果哪个命令不会使用可以通过输入：svn help 命令 的形式来查看帮助信息，例如：
```php
svn help chechout  输出信息如下： 

checkout (co): Check out a working copy from a repository. usage: checkout URL[@REV]... [PATH]
```
8. 更多关于ubuntu的svn使用，参考 http://pan.baidu.com/s/1mgxbHGW 

 

### 五. 编程过程

1. 不管哪种编程都应该学会当遇到困难时，可先百度. 谷歌. 知网. 知乎；还可以加入一些对应的Q群，必要时询问Q群的大神；确实不行就跟团队成员交流。网上搜索，我十分推荐去谷歌搜索问题的解决办法，此处分享一个能上谷歌的番茄神器。 Hoststool。此处贴出windows的版本

http://pan.baidu.com/s/1jGtccf0  使用方法按照上面的提示操作。

关于其他操作系统的可以去官网下载，地址：

https://hosts.huhamhire.com/

3.     这里也给出集成了翻墙软件的谷歌浏览器， 地址

链接：http://pan.baidu.com/s/1c05zAqW  密码：n3j9

4.     由于web开发的运行环境就是平时使用的浏览器。至于使用哪个浏览器，在这我建议 谷歌浏览器 chrome ，还有火狐浏览器。若是前端开发，需要测试网站兼容性的，可以使用其他浏览器查看，但debug的功能最好在chrome或者火狐浏览器完成。

5.     谷歌浏览器和火狐浏览器的前端debug使用方法，可以再网上搜索，在这里不再叙述。

 
### 六. 开发建议，所有语言教程，建议查看官方开发手册。

1. 关于html，css和JavaScript的入门和使用教程，推荐去W3CSchool这个网站学习 ，网址http://www.w3cschool.cc/

也可以下载离线版本 http://pan.baidu.com/s/1kTj6DwF

2.  为了能快速开发，在开发过程中推荐使用前端框架，避免不必要的代码重复书写工作，加快项目进度。现在大多数网站采用bootstrap 这个前端框架 ，网址 http://www.bootcss.com/  至于这是什么样的框架和使用教程可以参考官网，不再叙述。

还有AmazeUI 这个框架，使用教程参考官网 ，网址：http://amazeui.org/

*注：至于实际开发采用哪一个，看实际需要。

3. JavaScript的使用可以参考w3cschool 网站的教程。

网址：http://www.w3cschool.cc/ ，前端开发者，要熟悉使用jquery，还有封装项目中使用到的JavaScript 类库。

4. PHP教程，同样可以在w3cschool 网站找到。推荐查看php官网的手册，离线版本下载 http://pan.baidu.com/s/1hhhaY  ，这里还有php的基础教程，仅供参考http://pan.baidu.com/s/1eQy9SMu

5. PHP框架，比如，国内的thinkphp 国产中较好的之一，网址：http://www.thinkphp.cn/ 国外的laravel ，最受欢迎的，网址：http://laravel.com/ 。选择哪一个框架要结合项目开发需求和自己的开发经验。

学习框架的最好方法就是不断看手册和逛官方社区

6. mysql教程，同样可以在w3cschool 网站找到。推荐查看mysql官网的手册，离线版本下载

http://pan.baidu.com/s/1pJqPW1P  

（1）     无论在windows还是ubuntu中，都可以使用数据库图形管理工具 phpmyadmin 。phpmyadmin的使用教程

http://pan.baidu.com/s/10JWtC 

（2）     在windows中，我们还可以使用navicat for mysql 这个数据库图形管理工具 ，下载地址http://pan.baidu.com/s/1m6N1c  使用方法跟phpmyadmin差不多，但功能会多很多，自行探讨。

（3）     数据库的开发可能需要用到建模工具，此处给出一个叫powerdesigner 的软件，下载地址：http://pan.baidu.com/s/1o6MGgXk  使用教程自行查询。

7. Git 的使用指南。 http://rogerdudler.github.io/git-guide/index.zh.html 最开始我是使用 SVN 进行代码托管，后来改用 Git.两者的优缺点在这就不描述，看个人喜好。

### 七. 总结

1.  不管做什么编程，都应该以实践为主，不要一直捧着书本或者一直看别人的视频，自己看懂了却不去动手敲代码。

2.  以上的教程仅作参考，软件涉及到32位和64位的问题时，自行解决。

3. web编程中，不管是专注前端还是后台，都应该去了解自己未负责的部分，互相渗透。

4. 若发现错漏之处，欢迎提出。



## 希望大家技术猛进！
