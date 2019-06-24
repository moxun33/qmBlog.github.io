---
title: 使用Jenkins自动部署React前端项目
date: 2017-10-28 20:58:50
tags:
	 - React
	 - Jenkins

---
+ 经过一个月的开发，公司的 React 项目基本进入测试阶段，最开始是手动打包并传送到目标服务器，效率十分低下。w闻将介绍如何用 Jenkins 实现 React 项目的自动打包和部署。
 <!-- more -->

## 安装Java 和 Jenkins

### 1、略过安装 Java 
### 2、下载 Jenkins 的 [war包](http://mirrors.jenkins.io/war-stable/latest/jenkins.war)
### 3、根据提示安装 Jenkins
war包启动方式：`` java -jar jenkins.war –httpPort=8080 ``,端口可自定义，然后访问<http://ip:8080>
### 4、插件管理
点开系统设置的插件管理页面，如果可选插件列表为空，点击高级标签页，替换升级站点的URL为：<http://mirror.xmission.com/jenkins/updates/update-center.json>
并且点击提交和立即获取
![](https://ws2.sinaimg.cn/large/006tNc79gy1fkyahvg2tbj30i5062a9z.jpg)
### 5、返回可选插件，选择如下插件安装（如果已安装，请忽略）
1).Publish Over SSH
2).GitLab Plugin
3).Email Extension Plugin

## 插件配置 

### 配置Publish Over SSH
在Publish over SSH处点击增加，添加SSH server，并且选择高级设置，勾选使用用户名登录，设置相应的ip,用户名和密码等。（其他选项可忽略）
![](https://ws3.sinaimg.cn/large/006tNc79gy1fkyak4hfh8j30j70apdfv.jpg)

### 配置Email Extension Plugin
![](https://ws1.sinaimg.cn/large/006tNc79gy1fkyampxjwkj30iz05emx2.jpg)

## 新建任务

### 1、新建项目
点击新建，输入项目名称，选择‘自由风格项目’，OK
![](https://ws2.sinaimg.cn/large/006tNc79gy1fkyao3jzb3j30jg080q33.jpg)

### 2、配置源码
选择Git，并填上gitlab项目克隆地址，用户密钥以及分支
![](https://ws4.sinaimg.cn/large/006tNc79gy1fkyaqocvfej30j8081weg.jpg)

### 3、配置构建触发器
勾选Build when a change is pushed to GitLab和Poll SCM即可，其他可忽略，默认提交代码到相应的分支触发该任务。

![](https://ws1.sinaimg.cn/large/006tNc79gy1fkyatmvugcj30q40jlgm1.jpg)

### 4、配置构建
![](https://ws4.sinaimg.cn/large/006tNc79gy1fkyauxerpij30px0883yf.jpg)

### 5、配置构建后操作
增加构建后操作，选择``Send build artifacts over SSH``。即上述操作全部完成并自动生产了部署文件，该步骤将部署文件上传到之前的SSH服务器（Linux服务器），并执行你想让他执行的命名，部署多个服务器及平台，选择添加server并完成相应的配置
![](https://ws3.sinaimg.cn/large/006tNc79gy1fkyazvs1edj30un0gngly.jpg)

#### （1）、构建后配置 Email（可选）
配置后再增加构建后操作，选择Editable Email Notification。根据提示配置需要通知的邮箱，其他可默认。然后选择高级设置，配置失败和成功邮件通知。
![](https://ws4.sinaimg.cn/large/006tKfTcgy1fkybnzkymuj30j307z747.jpg)

#### （2）、构建后执行 Shell命令（可选）
增加构建后操作 ，选择Post build task。根据 匹配 Log 的文本消息执行不同的 shell 命令，
Log 文本的匹配需要根据实际情况修改，可使用简单正则表达式。这里执行的命令可以是一个 shell脚本，把自己需要执行的任务写到脚本中。
![](https://ws1.sinaimg.cn/large/006tNc79gy1fkyb5ue9uuj30ux0kz0sx.jpg)

## 绑定 Gitlabs Web hooks
添加web hook. ``http://jenkins-server/gitlab/notify_commit`` 需要填上的固定格式的URL地址，把jenkins-server替换成对应的Jenkins访问地址，注意：如果是localhost，需要换成ip地址。
添加完之后，点击Test Hook。此时Jenkins界面构建队列出现某个任务正在执行，表示配置成功。
![](https://ws3.sinaimg.cn/large/006tNc79gy1fkyb8013acj30im0820sw.jpg)

## 提交代码验证 Jenkins 功能。
我这里使用的构建后任务是发送公司内部 IM 消息，构建后我收到了这样的消息。
![](https://ws1.sinaimg.cn/large/006tNc79gy1fkybbvlhqgj309b0153yc.jpg)

## nginx 配置
由于构建后的 React 文件都是静态文件，我们需要一个 Web 服务器来托管我们的项目。这里使用 nginx,并配置反向代理。
这里略过如何安装 nginx，可参考[安装 PHP 开发环境的 nginx 安装部分](http://qimajiang.com/2017/02/25/PHP%E5%B7%A5%E5%8C%A0%E5%8E%86%E9%99%A9%E8%AE%B0-%E5%BC%80%E5%90%AF%E7%AF%87-%E4%B8%80%E3%80%81%E6%90%AD%E5%BB%BAPHP%E5%BC%80%E5%8F%91%E7%8E%AF%E5%A2%83/)
讲一下 nginx 配置的注意点。
1、反向代理:反向代理转发时过滤掉 /api路径。

```php
 location /api {
  	rewrite /api(.*) $1 break;
	proxy_pass         http://localhost:8080;
 	proxy_redirect     off;
    proxy_set_header   Host $host;
   }
```

2、react-router 失效。
可能会有朋友注意到，当刷新页面时，nginx 会返回404响应。这是因为nginx 查找的路由是浏览器上的地址，但我们的项目不存在这样的路由，所以会出错。我们需要把所有的请求都重定向到 index.html 文件。在 location / {}里增加：
```php
 try_files $uri /index.html;
```

## 完毕
