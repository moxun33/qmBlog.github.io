---
layout: 使用
title: 使用 Goutte爬取 IP 代理池
date: 2017-11-18 01:16:18
tags:
     - Goutte
     - PHP
     - 爬虫
---
+ 最近几乎所有精力都投入到 React 的开发中，可以说对 JS 的理解更深入了， 使用起来也得心应手了。但对于 PHP, 我还是想找点东西捣鼓下的。这次我们就用 PHP 的爬虫框架 Goutte 来爬取多个 代理IP，实现一个简单的自动投票系统， 该系统可以切换 IP，从而避免被封禁。当然，这里只是抛砖引玉， Goutte 的强大之处远不在于此。
 <!-- more -->

## 一、关于 Goutte
Goutte 是一个 PHP 爬虫框架，提供了优雅的 API 进行链接抓取，和解析 HTML 文档。Goutte 主要使用以下 PHP 类库：

- 页面解析：Symfony 的 BrowserKit ， CssSelector 和 DomCrawler；
- HTTP 请求： Guzzle

相信各位 PHPer 对Guzzle这个库很熟悉吧，这里不展开讨论了。简单来说， 利用 Goutte 可以快速的爬取和解析 Html、xml 的文档， 得到我们的目标数据。

## 准备工作
 既然 Goutte 是解析 HTML 结构的， 那解析规则必须已经设置好，分析目标网站的 DOM 结构是我们的首要工作。

这里我使用的是<https://proxy.coderbusy.com/>, 打开该网站， 通过页面我们可以知道， 我们的目标数据（IP 池）是显示在 Table 中， 但究竟这是个怎样的 Table 呢？我们打开网站的开发者工具， 分析其 HTMl 代码：
1、  <table class='table proxy-server-table'>, 这个就是我们的目标表格， 其还定义了类名， 这样我们可以快速定位到此元素了。
2、  table 的头部有很多列， 这里是每个 IP 的其他属性， 因为我们的目的不是 IP 的属性， 只需要一个代理 IP和端口， 因此我们关心的是 table 的第一列的 IP 地址和第二列的端口。我们直接查看 table 的主体部分<tbody>的第一个``<td>``和第二个``<td>``。

通过查看源码我们可以知道， 我们的 goute 需要解析的是该页面的``class='table proxy-server-table'``的 table 中 tbody 中的第一列和第二列。

## 正式爬取

1、 我们使用 composer 安装 Goutte，然后引入。
```php
require_once dirname(dirname(__FILE__)).'./vendor/autoload.php';
use Goutte\Client;

```

2、编写爬取函数
```

function getProxyIPPool(){
    $client = new Client();
    $ipPoolUrl = 'https://proxy.coderbusy.com/';
    $crawler = $client->request('GET', $ipPoolUrl);
    $ipArr = $crawler->filter('table')->filter('tr')->each(function ($tr, $i) {
        return $tr->filter('td')->each(function ($td, $i) {
            return trim($td->text());

        });
    });

    $newArr = [];
    foreach ($ipArr as $key =>$value){
        if ($key >0){
            $newArr[$key-1] = $value[0].':'.$value[1];//这里就是 table 的第一列和第二列
        }

    }
    $file = file_put_contents('ipPool.json', json_encode($newArr));

    echo "更新代理 IP 尺完毕";
}
```
这里我们直接把解析出来的数据存储到本地 json 文件中（真正应用时最好存到数据库， 以便用于其他分析用途）。
执行完一次后我们得到的 json 文件(仅显示部分数据):
```php
[
    "13.59.33.106:3128",
    "200.116.227.99:53281",
    "190.232.168.242:8080"
]

```
由于该网站每次刷新的时候表格的数据都是不一样，我们既可以存储到不同 json 文件中， 也可以覆盖同一个 json 文件。

### 对， 就是这么简单， 通过设定爬取周期，就可以得到很多代理 IP 了。

## 自动投票系统
其实这种自动刷票的功能不是很愿意做， 但为了学习还是可以练习一下的。

1、目标网站<http://jljapi.manhuadao.cn/Vote/vote>

2、编写投票函数(这里 请求比较简单，直接用 CURL)
```php
function voteByIP ($ip, $port) {

    $post_data = array ("bookid" => "102428");
    $ch = curl_init("http://jljapi.manhuadao.cn/Vote/vote");
    curl_setopt($ch, CURLOPT_POST, 1);
    curl_setopt($ch, CURLOPT_POSTFIELDS, $post_data);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER,1);
    curl_setopt($ch,CURLOPT_PROXY,$ip);
    curl_setopt($ch,CURLOPT_PROXYPORT,$port);
    curl_setopt ($ch, CURLOPT_TIMEOUT, 1200);

    $result = curl_exec($ch);
   print_r($result.PHP_EOL);
    curl_close($ch);
}

```

3、使用不同的 IP 发起投票请求

```php

$ipPoolArr = json_decode(file_get_contents('ipPool.json'));

foreach ($ipPoolArr as $key=>$proxyStr){
     $ip = $proxyArr[0];
    $port = $proxyArr[1];

    voteByIP($ip, $port)
}

```

4、自动循环投票Shell脚本

```php
#!/bin/bash

for((i=1;i<=99999999999999999999999999999999999999;i++)); 
do
     php autoVote.php

     //还可以执行其他的操作，例如 php getIPPool.php 来更新代理 IP 池
done

```

5、功能扩展

其实这里只是做了简单自动爬虫和投票功能， 从获取代理 IP 到投票， 本地的 IP使用完之后刷新 IP 池。因此，可以扩展的功能有：

1、把代理 IP 池存储到数据；
2、不定时发起投票请求， 防止频繁请求被封禁，虽然切换了代理 IP。
3、统计代理 IP 的使用情况和对应的投票成功率；
4、ETC

## 总结
本文使用 Goutte爬虫框架实现爬取代理 IP 池，然后利用得到的 IP来 自动投票。这里只是 Goutte 的简单用法，更多用法可以查看 Goutte 官方文档。

