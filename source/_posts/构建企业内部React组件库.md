---
title: 构建企业内部React组件库
date: 2018-03-24 01:37:40
tags:
     - React
 
---
+ 最近针对日常业务需求使用react封装了一套基于 ant-design 的[业务组件库], 大概记录下整个开发过程中的心得，在这里只对开发过程中的选型和打包等进行讨论，后续再对具体组件的封装进行讨论。
<!--more-->

## 概述

随着公司的 React 应用越来越多， 在编写每个应用时都会封装各自的组件， 但久而久之会发现， 有些组件是每个应用都可以使用的。 然而，在每个应用内维护相同的功能组件， 既浪费时间， 组件功能和样式又得不到同步。于是我就在想能不能把这些通用组件抽取出来， 创建一个企业内部的业务组件库， 打包发布到 npm 内部管理服务器。

由于目前公司采用的组件库是 ant-design ，本次的内部组件库是对 ant-design 的二次封装，以便满足公司的业务。虽然 [antd-pro](https://pro.ant.design/index-cn)是基于 ant-desgin 的中后台业务组件库， 但也无法完全满足公司的业务逻辑， 所以还是自己做一些功夫， 定制自己业务组件库，暂且命名为 hx-react-components

## 搭建 npm 私服

npm 私服有几个选择：

- [sinopia](https://www.npmjs.com/package/sinopia) 已停止维护和升级

- [Verdaccio](https://github.com/verdaccio/verdaccio) 一个sinopia的fork

这里我采用 Verdaccio 搭建 npm 私服。安装教程可参考官方教程。使用也很简单：


- 启动 ``verdaccio``,不要用 root 启动，会引发不可控错误。

- 设置服务访问路径，默认只能本地访问 `` npm set registry http://ip:4873/``

- 添加用户 ``npm adduser --registry http://ip:4873``

- 发布我们的包 ``npm publish --registry http://ip:4873``

就是如此简单！

## 技术选型

### 技术方案选择

Webpack + React

毫无疑问的选择 React 进行业务组件定制。

#### 1、应用初始化

使用 [create-react-app](https://github.com/facebook/create-react-app)进行初始化， 无须过多的配置。

#### 2、引入 ant-design

具体方法参考 [ant-desgin](ant.design) 的官方教程。

#### 3、样式预编译语言

由于该组件库大多是业务逻辑组件， 暂且不考虑 Less 其他样式预编译语言。后期再考虑。所以依然采用 CSS 进行部分样式描绘。

### 开发流程和规范

由于该组件库是完全独立于其他任何 React 项目， 而且不需要编写页面， 因此需要有特有的开发流程和架构。主要有一下几点：

- 包含开发、测试和打包三种模式。

- 使用pure-render，autobind等尽可能保证组件的性能及效率

- 保证props和回调的语义性，如回调统一使用onXXX进行处理

针对不同的模式， 配置不同的 webpack脚本：

```bash

 
    "test": "react-scripts test --env=jsdom",//测试
    "styleguide": "styleguidist server", //文档开发
    "styleguide:build": "styleguidist build", //文档大伯
    "styleguide:deploy": "scp -r styleguide/* root@192.168.1.x:/var/www/styleguide",//文档部署，供公司人员查看
    "predeploy": "npm run styleguide:build",//预处理文档部署
    "build": "babel src -d lib",//组件库打包
    "lint": "eslint src/components/**; exit 0",//组件库规范检查
    "lint:watch": "esw -w src/**",//规范监控
    "prepublish": "npm run build",//预发布
    "push": "git push origin && git push origin --tags",//推送远程仓库
    "release:patch": "npm version patch && npm publish --registry=http://192.168.1.x",// 兼容版本发布到 Verdaccio
    "release:minor": "npm version minor && npm publish --registry=http://192.168.1.x",// 小版本发布到 Verdaccio
    "release:major": "npm version major && npm publish --registry=http://192.168.1.x"// 大版本发布到 Verdaccio
  

```

## 打包

### webpack 配置

针对组件库的打包，我们以UMD格式对其进行打包。webpack可以针对输出进行格式设置：（引自cnode)

- “var” 以变量方式输出
- “this” 以 this 的一个属性输出： this[“Library”] = xxx；
- “commonjs” 以 exports 的一个属性输出：exports[“Library”] = xxx；
- “commonjs2” 以 module.exports 形式输出：module.exports = xxx；
- “amd” 以 AMD 格式输出；
- “umd” 同时以 AMD、CommonJS2 和全局属性形式输出。

配置如下：

```js

output: {
  path: config.build.assetsRoot,
  filename: utils.assetsPath('js/[name].js'),
  chunkFilename: utils.assetsPath('js/[id].js'),
  library: 'TipUi',
  libraryTarget: 'umd'
}

```

注意：

> 这里不应该把React引用进去。一般我们可以采用externals的方式对其进行处理。

## 文档示例

开发组件库不同于页面开发， 可以配置路由来预览我们的开发。

一个完善的文档对于一个组件库是及其重要的，每个组件有什么样的配置参数，拥有哪些事件回调，对应的Demo和展示效果。假设没有这些，除了封装组件的人，没有人知道它该如何使用。但是写文档的过程往往是痛苦的，在这里推荐几个文档生成库，可以极大的简化文档工作：

- docsify 基于Vue的组件生成器，轻量好用
- react-styleguidist 基于React的组件库文档生成器，自动根据注释生成文档，支持Demo展示。超好用
- bisheng ant design自己写的文档生成器

我们使用的styleguidist, 可以将md自动转化为文档，支持在md内直接调用你封装好的组件并进行展示，简单好用

## 组件库使用

在我们的项目里， 其他的依赖包是从 npm 官网拉取， 但我们的x-react-components 是走npm 私服 - verdaccio， 那如何写package.json 的 x-react-components 依赖路径呢？

 目前我们是这样写：

 ```js
 
 "x-react-components":"http://ip:4873/x-react-components/-/0.1.15.tar.gz"



 ```
这样写的缺点就是每次手动改版本号，  后续会考虑采用 dll检查组件库的更新状况。

每次 react 应用打包前检查 x-react-components 是否有更新。有， 先更新组件库， 再打包； 否则， 直接打包。


## 最后

本文概述了如何构建 npm 私服、开发自己的业务组件库。篇幅有限， 可能很多细节没有描述清楚， 但这大致是我在公司搭建的内部react 组件 库的过程。








