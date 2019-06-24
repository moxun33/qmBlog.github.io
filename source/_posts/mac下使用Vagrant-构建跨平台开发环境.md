---
title:  mac 下使用Vagrant 构建跨平台开发环境
date: 2017-09-03 14:05:43
tags:
      - Vagrant
      - PHP
      - Homestead
---
+   [Vagrant](vagrant.com) 是一款用来构建虚拟开发环境的工具，非常适合 php/python/ruby/java 这类语言开发 web 应用，“代码在我机子上运行没有问题”这种说辞将成为历史。本文通过安装php开发环境[homestead](https://d.laravel-china.org/docs/5.5/homestead#configuring-homestead)来说明 Vagrant 的安装使用与优点。
 <!-- more -->

+ 我们可以通过 Vagrant 封装一个 Linux 的开发环境，分发给团队成员。成员可以在自己喜欢的桌面系统（Mac/Windows/Linux）上开发,这样不仅保证代码运行环境一致，还可以节省部署开发环境带来的不必要时间开销，让团队新成员快速投入开发工作中。

 

## 安装步骤

### 1、安装VirtualBox
下载地址：<https://www.virtualbox.org/wiki/Downloads>;虽然 Vagrant 也支持 VMware，但是VMware 对应的 Vagrant 版本也是收费的。

### 2、安装 Vagrant 
Vagrant 的下载地址<https://www.vagrantup.com/downloads.html>;下载好后，打开 Vagrant镜像一步一步的安装。使用

<pre>
$ vagrant -v
</pre>

查看是否安装成功。

### 3、添加box镜像
Vagrant 安装成功后，运行

<pre>
vagrant box add laravel/homestead

</pre>

命令将 laravel/homestead 盒子添加到 Vagrant 中安装。下载盒子需要几分钟的时间，具体取决于你的互联网连接速度。

但一般国内网络环境下都会失败。那么我们需要离线安装 box 。下载[homestead 离线安装包]( https://vagrantcloud.com/laravel/boxes/homestead/versions/3.0.0/providers/virtualbox.box);下载完成后，运行

<pre>
vagrant box add laravel/homestead ~/Download/virtualbox.box
</pre>

此处路径只是为了演示。

因为是手动导入包，还需要多更改几个步骤。

编辑Homestead/scripts/homestead.rb文件

修改

<pre>
config.vm.box_version = settings["version"] ||= ">= 1.0.0"

改成

config.vm.box_version = settings["version"] ||= ">= 0
</pre>
这样就安装离线安装好 homestead box 了。可以使用

<pre>
vagrant up
</pre>
来启动 box 了。
### 4、克隆和配置homestead文件
在目标目录，运行

<pre>
git clone https://github.com/laravel/homestead.git Homestead
</pre>

由于 Homestead 的 master 分支并不是稳定分支，你应该用打过标签的稳定版本。你可以在 Github 发行页面 上找到最新的稳定版本。

<pre>
cd Homestead

// Clone the desired release...
git checkout v6.1.0

</pre>

克隆 Homestead 代码库后，从 Homestead 目录中运行`` bash init.sh`` 命令来创建 Homesstead.yaml 配置文件。 Homesstead.yaml 文件会被放置在你的 Homestead 目录中：
运行命令来初始化配置

<pre>
bash init.sh

</pre>

其他详细的配置参考官方文档，这里不再赘述。

### 5、初始化开发环境
切换到目标目录，使用 ``vagrant init laravel/homestead``初始化该目录的环境。然后启动环境``vagrant up``。
看到终端显示了启动过程，启动完成后，我们就可以用 SSH 登录虚拟机了，剩下的步骤就是在虚拟机里配置你要运行的各种环境和参数了。

<pre>
$ vagrant ssh  # SSH 登录
$ cd /vagrant  # 切换到开发目录，也就是宿主机上的 工作目录
</pre>

### 6、其他设置
Vagrant 初始化成功后，会在初始化的目录里生成一个 Vagrantfile 的配置文件，可以修改配置文件进行个性化的定制。

Vagrant 默认是使用端口映射方式将虚拟机的端口映射本地从而实现类似 http://localhost:80 这种访问方式，这种方式比较麻烦，新开和修改端口的时候都得编辑。相比较而言，host-only 模式显得方便多了。打开 Vagrantfile，将下面这行的注释去掉（移除 #）并保存：

<pre>
config.vm.network :private_network, ip: "192.168.33.10"
</pre>

重启虚拟机，这样我们就能用 192.168.33.10 访问这台机器了，你可以把 IP 改成其他地址，只要不产生冲突就行。

### 7、设置共享文件夹
打开 Vagrantfile,输入以下代码或者去掉注释
config.vm.synced_folder "/Users/name/nginxWeb", "/wwwroot"
第一个是你本地的文件夹 第二个是挂在到虚拟机上的文件夹

给共享文件夹设置权限

默认共享文件夹属主 和属组都是 vagrant 我们如果php操作文件夹 是没权限的  要把该文件夹设置成 www用户

<pre>
  config.vm.synced_folder "/Users/gwyy/wwwroot","/wwwroot", create:true, 

  :owner => "www", :group => "www", :mount_options => ["dmode=775","fmode=664"]

</pre>

## 打包分发
虽然我们这里的用 laravel 官方的开发环境， 其他成员可以使用同样方法搭建好一致的开发环境。那么，如果我们需要根据自己的开发需求来定制 box 后该如何打包分发给其他成员呢。
当你配置好开发环境后，退出并关闭虚拟机。在终端里对开发环境进行打包：

<pre>
vagrant package
</pre>

打包完成后会在当前目录生成一个 package.box 的文件，将这个文件传给其他用户，其他用户只要添加这个 box 并用其初始化自己的开发目录就能得到一个一模一样的开发环境了。

## 常用命令

<pre>
 $ vagrant init  # 初始化
 $ vagrant up  # 启动虚拟机
 $ vagrant halt  # 关闭虚拟机
 $ vagrant reload  # 重启虚拟机
 $ vagrant ssh  # SSH 至虚拟机
 $ vagrant status  # 查看虚拟机运行状态
 $ vagrant destroy  # 销毁当前虚拟机
</pre>

## 注意事项
1、使用 Apache/Nginx 时会出现诸如图片修改后但页面刷新仍然是旧文件的情况，是由于静态文件缓存造成的。需要对虚拟机里的 Apache/Nginx 配置文件进行修改：

<pre>
# Apache 配置添加:
EnableSendfile off

# Nginx 配置添加:
sendfile off;
</pre>

2、如果访问出现    no input file specified

输入 ``vagrantprovision``



