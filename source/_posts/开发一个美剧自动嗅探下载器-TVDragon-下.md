---
title: 开发一个美剧自动嗅探下载器:TVDragon(下)
toc: true
date: 2018-01-28 13:52:31
tags:
     - 爬虫
---
+ 通过上篇的准备工作， 我们基本掌握整个爬虫的工作流程。接下来我们就要实现该爬虫系统了。在程序语言上有了改动， 因为最近基本是 JavaScript开发， 所以我们采用 Nodejs 作为开发语言， 而不是 PHP。毕竟， 一个程序的好坏跟程序语言没有必然联系。
 <!-- more -->

## 一、搭建框架
#### 1、使用 ``package.json``初始化。
#### 2、构建目录结构，目前的目录结构如图。
![](https://ws4.sinaimg.cn/large/006tKfTcly1fnwaa886m9j306n097wef.jpg)

 解释：
- ``src``
    - ``dev.js spider.js index.js ``根据部署环境调用不同文件作为入口。
    - ``config`` 数据库和 程序参数等基本设置
    - ``constants`` 一些种子网站列表和 api 等定义。
    - ``core ``核心的爬虫类， 定义播放表获取、下 html 代码 html解析类和 transmissiorpc 连接类等核心类库。  
    - ``storage``  数据库 Schema 定义和 CRUD 的基本服务类。

#### 3、定义 Mongoose Schema

这里我们使用 Mongoose 来操作MongoDB，首先需要定义Schema。

1、 播放表 Schema，用了存储从字幕组 APP 获取的美剧播放表。

```php
import mongoose from 'mongoose'

const Schema = mongoose.Schema
const SeriesSchema = new Schema(
    {
        id: {type: Number, required: true},
        cnname: {type: String, required: true},
        enname: {type: String, required: true},
        poster:  String ,
        season: {type: Number, required: true},
        episode: {type: Number, required: true},
        play_time: {type: String, required: true},
        poster_a: String,
        poster_b: String,
        poster_m: String,
        poster_s: String,
        status: { type: Number, required: true, default: 1000 },
        created_at: {type: Date, default: Date.now},
        updated_at: {type: Date, default: Date.now}
    },
    {
        collection: 'series'
    }
)

export default   mongoose.model('series', SeriesSchema)

```

2、种子 Schema,用来存储从种子网站爬取解析得到的 torrent 列表。

```php
import mongoose from 'mongoose'

const Schema = mongoose.Schema
const TorrentSchema = new Schema(
    {
        series_id: {type: Number, required: true},
        torrent_name: String,
        hash: String,
        url: String,
        size: String,
        seeds:Number,//种子数
        downloaded: {type: Boolean, required: true, default: false},
        status: { type: Number, required: true, default: 1000 },
        created_time: {type: Date, default: Date.now},
        updated_time: {type: Date, default: Date.now}
    },
    {
        collection: 'torrent'
    }
)

export default   mongoose.model('torrent', TorrentSchema)

```

#### 4、爬取美剧播放表信息（根据当天日期），持久化存储到 MongoDB

关键代码：

```php


     fetchSchedule (startDate, endDate)  {
        const queryStr =  {
            accesskey:'e519f9cajd7qc8059d175449479u61a827',
            client:1,
            g:'api/v2',
            m:'index',
            a:'tv_schedule',
            start:  startDate,
            end: endDate
        };
        const options = {
            url: RR_PLAY_SCHEDULE,
            method: "get",
            qs: queryStr,
            headers: {
                'User-Agent': UserAgent[0]
            }
        };
        return new Promise((resolve, reject) => {
            request(options, (error, response, body) => {
                if (!error && response.statusCode === 200) {
                    const info = JSON.parse(body);
                    return resolve(info)
                } else {
                    return reject(error)
                }
    
            })
        })
    
    }
```
#### 5、使用 Pupeteer 下载 html 源码。

Puppeteer是谷歌官方出品的一个通过DevTools协议控制headless Chrome的Node库。可以通过Puppeteer的提供的api直接控制Chrome模拟大部分用户操作来进行UI Test或者作为爬虫访问页面来收集数据。


Puppeteer的使用十分简单，通过操作Browser实例来操作浏览器作出相应的反应。

此处使用Puppeteer下载 html页面的关键代码：

```php
  puppeteerDownloadHTML () {
    return new Promise(async (resolve, reject) => {
      try {
        const browser = await puppeteer.launch({
            headless: true,
            ignoreHTTPSErrors: true,
        })
        const page = await browser.newPage()
          await page.setCookie(this.cookie)
        await page.goto(this.url)
        const bodyHandle = await page.$('body')
        const bodyHTML = await page.evaluate(body => body.innerHTML, bodyHandle)
        return resolve(bodyHTML)
      } catch (err) {
        console.log(err)
        return reject(err)
      }
    })
  }
```

#### 6、使用 cheerio 从 html 文本中解析数据。

cheerio是一个node的库，可以理解为一个Node.js版本的jquery，用来从网页中以 css selector取数据，使用方式和jquery基本相同。事实上，Cheerio 从jQuery库中去除了所有 DOM不一致性和浏览器尴尬的部分，性能更优。

前面我们已经使用 Pupeteer 下载了 html 源码， 我们将提取我们需要的数据列表，然后持久化到 mongodb 中。

关键代码

```php

  torrentz2HubExtract() {
        let nodeList = this.$('#wrap').find('.results').find('dl')
        nodeList.each((i, e) => {
            let a = this.$(e).find('a');
            const hash = a.attr('href')?a.attr('href').replace('/', ''):a.attr('href');
            const seeds = this.$(e).find('dd').find('span').eq(3).text()
            this.extractData.push(
                this.extractDataFactory(
                    'http://itorrents.org/torrent/' + hash.toUpperCase() + '.torrent',
                    a.text(),
                    seeds,
                    hash,

                )
            )
        })
        return this.extractData
    }
```

##### 当然每个网站的页面结构不一致， 对不同的站点，我们需要不同解析模式，但使用 cheerio 让这项工作变的超级简单。

#### 7、使用 TransmissionRPC 进行 torrent 上传及资源下载。

这里我们采用 [transmission-promise](https://github.com/grantholle/transmission) 进行 transmission 接口调用。根据其文档，我们很容易实现一个自己的 rpc 类库。

这里就不贴代码了， 与文档的例子差不多。

#### 8、cron定时作业

爬虫的意义之一就是能定时自动化爬取目标数据。

关键代码

```php
/!**
 *  # ┌────────────── second (optional)
 *  # │ ┌──────────── minute
 *  # │ │ ┌────────── hour
 *  # │ │ │ ┌──────── day of month
 *  # │ │ │ │ ┌────── month
 *  # │ │ │ │ │ ┌──── day of week
 *  # │ │ │ │ │ │
 *  # │ │ │ │ │ │
 *  # * * * * * *
 *!/
function job () {
    let cronJob = new cron.CronJob({
        cronTime: cronConfig.cronTime,
        onTick: () => {
            spider()
        },
        start: false
    })
    cronJob.start()
}

job()

```

每天定时跑一遍上述的流程即可。

## 二、后记

目前 TVDragon 还在开发中， 平时时间不是很充裕， 现在基本完成上述模块。

1、需要订阅所追的所有美剧名称，根据订阅的美剧和播放表去搜索 Torrent， 节省耗损。
2、搜索的 Torrent 列表可能有多个， 视频的质量参差不齐， 需要一个从优筛选 torrent 的算法。
3、 使用日志代替各个 console，针对下仔的信息使用微信公众号模板消息推送， 方便实时跟踪程序运行情况。

4、todo



