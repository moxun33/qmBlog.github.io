---
title: 搭建PHP开发环境
date: 2017-02-25 21:57:15
tags:
	- PHP
	- 开发环境
	- LAMP
---
+ 从本文开始，记录一直以来的 PHP 开发历程， 一方面，自己温故而知新； 另一方面，跟大家一起学习进步。 第一篇文章当然是搭建 PHP 开发环境啦。至于 PHP是什么，以及 PHP 能干什么等等的问题，我这不赘述，请自行 Google。
 <!-- more -->
## 在 Mac 系统搭建 LAMP开发环境。

#### 一、 启动 Apache
  mac 系统自带 Apache 服务器;只需要知道如何启动即可。一个方法是通过终端启动。
```bash
  sudo apachectl start
 ```
 此时在浏览器中输入http://localhost，会出现It works！的页面。
 另外的方法是：   打开"系统偏好设置"->"共享"，在"互联网共享"那一项前面打√。

 #### 二、运行 PHP
1、打开 Apache 的配置文件
```php
 	sudo vi httpd.conf
```
 找到 #LoadModule php5_module libexec/apache2/libphp5.so 这一行； 把前面的# 删除；然后保存退出。如图：
 
 ![](https://ooo.0o0.ooo/2017/03/18/58cd43e5269e1.png)
 
 2、重启 Apache, 然后 PHP 即可正常使用。
 ```bash 
 sudo apachectl restart
 ```
 3、检验 PHP 是否正常。
 ```php 
 php -v
 ```
 是否看到 PHP 的版本信息；有的话证明已成功。如图：
 ![](https://ooo.0o0.ooo/2017/03/18/58cd44d33d068.png)

4、 关于更多 PHP 命令行，可以自行搜索学习。常用的：
```php
php -i  #查看 php 的详细信息；跟 phpinfo()效果一致。

php -i | grep php.ini #查看 php.ini 所在位置。

php -m #查看 php 扩展模块的安装情况
```

 #### 三、安装 MYSQL
1、在[MYSQL](http://dev.mysql.com/downloads/mysql/)网站上，选择Mac OS X平台，然后选择图中的DMG格式下载
![](https://ooo.0o0.ooo/2017/03/18/58cd467d93a38.png)
下载完后，自行安装。
2、修改 MYSQL 的密码；如设置密码为12345
```php
/usr/local/mysql/bin/mysqladmin -u root password 12345
```
3、 关于 MYSQL 的 GUI 管理工具；本人推荐 [sequel pro](https://www.sequelpro.com/) ；其他工具自行下载安装。

#### 四、要想进行 PHP 开发，当然还要有编辑器，你可以选择自己喜欢的编辑器进行开发，但都推荐免费开源放工具，以免造成不要的麻烦。

## Linux 安装 LAMP开发环境
  在这里推荐一步一步的进行安装，尤其的初学者，必须自己走一遍才知道如何安装，每个人出现的错误会不一样，在处理错误的过程一定会有收获。不建议，一开始为了省事，去使用[LAMP 一键安装脚本](https://lamp.sh/) ，当你完全掌握后，才推荐使用这种方法。由于 LINUX 的发行版本较多，这里以Centos7 系统为例子，其他发行版大同小异。

#### 安装 Apache
```PHP 
yum install httpd httpd-devel
```

#### 启动 Apache
```php 
 systemctl start httpd.service
```
测试以下apache是否安装成功，打开浏览器，输入[http://localhost](http://localhost)， 是否显示apache的主页。

#### 安装MYSQL; mysql在centos7.0版本中被mariadb替代了。
```php
yum isntall mysql mysql-server
```
安装好了，选择修改mysql默认的root用户的密码，启动mysql服务。
#### 启动 MYSQL
```php 
      service mysqld start
```
#### 安装 PHP
```php 
yum install php56w.x86_64 php56w-cli.x86_64 php56w-common.x86_64 php56w-gd.x86_64 php56w-ldap.x86_64 php56w-mbstring.x86_64 php56w-mcrypt.x86_64 php56w-mysql.x86_64 php56w-pdo.x86_64
```
若要安装其它 PHP 版本，把56改成其它版本号即可。

#### 安装 php-fpm
```php
yum install php56w-fpm 
```
#### 安装pear
// 下载 go-pear 脚本
```php
[arthur@arthur Downloads]$ wget http://pear.php.net/go-pear.phar
php
// 先切换到root账户，避免指定安装目录的时候没有权限导致安装失败
```bash
[arthur@arthur Downloads]$ su
Password:
```
// 执行安装
```bash
[root@arthur Downloads]$ php go-pear.phar
```
 
#####配置pear命令 

安装完成后，默认只能使用 安装目录/bin/pear的方式使用命令，为了方便后续直接使用，我们还需要把pear的安装目录加到配置文件中 /etc/profile
输入
```php
export PATH=/opt/pear/bin:$PATH
```

更新
```php
source /etc/profile
```
#### 安装Composer
# 下载composer.phar 
```php
curl -sS https://getcomposer.org/installer | php
```

# 把composer.phar移动到环境下让其变成可执行 
```php
mv composer.phar /usr/local/bin/composer
```

# 测试
```php
composer -V 
```

测试 PHP 是否安装成功；
```php 
php -v 
```
终端显示 PHP 的一些版本信息。

#### 安装PHP 对 MYSQL 的支持
```php 
yum install php-mysql
```
基本的开发环境就可以搭建好了。
 
 ## Windows 系统安装 LAMP 。哦， 我没用过 Windows 系统。个人建议开发还是尽量不要使用 Windows 了。当然非要在 Windows 上做开发的话，我这就不描述如何安装了。
## 总结
 
 开发环境是所有开发的基础，一定要稳扎稳打，静静的自己安装，不要偷懒哦！平时多上网查看其它的文章教程，每个人写的文章不一样。






