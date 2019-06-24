---
title: hexo常用命令
date: 2017-02-17 21:00:27
tags:
	 - hexo命令
---

+ 备忘常用 hexo 的命令。
 <!-- more -->

1. hexo generate (hexo g) 生成静态文件，会在当前目录下生成一个新的叫做public的文件夹
2. hexo server (hexo s) 启动本地web服务，用于博客的预览
3.  hexo deploy (hexo d) 部署播客到远端（比如github, heroku等平台）

###其他命令

```php
	$ hexo init 
	$ hexo new "articleName"
	$ hexo new page "pageName"
```
### 组合命令

```php
	$ hexo d -g #生成部署
	$ hexo s -g #生成预览
```

