---
title: React应用优化
toc: true
date: 2018-11-21 15:48:00
tags:
     - React
     - 优化  
---
随着功能不断增加，不断迭代更新，React应用会越来越臃肿了，性能也将随之下降。本文从打包和运行两个方面着手，谈谈React应用改如何优化。
<!-- more -->

# 一、webpack打包优化

## 1、缓存`node_moduels`

我公司的项目每次上线部署的时候，虽然说都要要Jenkins上，但项目越来越多，每个项目部署占用时间都很长，导致每次部署完一个环境的所有项目耗费很多时间。

如果将同一项目的`node_mudules`在每次打包完毕后缓存起来，下次打包前先判断是否与上次`node_moduels`相同。若相同，则直接使用上次缓存的`node_modules`,否则才重新安装依赖包。

那该如何实现上面所说的逻辑？

- 检查`packages.json`的 `md5`；
- 打包完成后以该次`packages.json`的`md5`值作为文件名，压缩`node_modules`并缓存到指定位置；
- 下次打包前，同样先检查当次`packages.json`的 `md5`，若相同直接使用上次的`node_moduels`;

具体的SHELL如下：

```js
#!/bin/bash
PKG_SUM=$(md5sum package.json | cut -d\  -f 1)
NPM_TARBALL_CACHE=${HOME}/.cache/ReactCache/npmtarball/reactSPA
NPM_TARBALL=node_modules-${PKG_SUM}.tgz
NPM_TARBALL_MD5SUM=${NPM_TARBALL}.md5sum
		[ ! -e ${NPM_TARBALL_CACHE} ] && mkdir -p ${NPM_TARBALL_CACHE}
   
    TARBALL=${NPM_TARBALL_CACHE}/${NPM_TARBALL}
    TARBALLMD5SUM=${NPM_TARBALL_CACHE}/${NPM_TARBALL_MD5SUM} 
    echo "checking node modules "${TARBALL}
    if [ ! -f ${TARBALL} ];then          
            echo "package.json has some changes, reinstall node modules"
            rm -rf ${NPM_TARBALL_CACHE}/*
         		yarn
         		echo "yarn success"
         		if [ -d node_modules ];then
         			echo "tar and caching..." 
             	tar zcf ${TARBALL} node_modules || return 1
             	md5sum ${TARBALL} > ${TARBALLMD5SUM}
             	echo "checking current MD5."
             	md5sum -c ${TARBALLMD5SUM} || rm -f ${TARBALL} ${TARBALLMD5SUM}  
             	echo "install completed, cached" 
       			fi         	     		
    else
    		echo "package.json has no changes, clone previous node modules"
    		if [  -d node_modules ];then
    				echo "node_modules dir existed "${TARBALL}
    			else
    				echo "unpacking..."	${TARBALL}    	
    				tar  xzf ${TARBALL}   
    		fi
        
    fi
chmod -R a+rwx node_modules 

```

## 2、加速代码压缩

`webpack`提供的UglifyJS插件由于采用单线程压缩，速度很慢 ,
`webpack-parallel-uglify-plugin`插件可以并行运行`UglifyJS`插件，这可以有效减少构建时间，当然，该插件应用于生产环境而非开发环境，配置如下：

```js

var ParallelUglifyPlugin = require('webpack-parallel-uglify-plugin');
new ParallelUglifyPlugin({
   cacheDir: '.cache/',
   uglifyJS:{
     output: {
       comments: false
     },
     compress: {
       warnings: false
     }
   }
 })
```

## 3、`HappyPack`加速构建

`happypack`的原理是让loader可以多进程去处理文件，原理如图示：

![](https://camo.githubusercontent.com/d819eb6ed893a849f0a83e1cb0b69a2c37e768b4/68747470733a2f2f73692e6765696c6963646e2e636f6d2f687a5f696d675f303864623030303030313561333536326361363830613032363836305f3830305f3438365f756e61646a7573742e706e67)

目前项目中基本只对js和less文件使用`HappyPack`加速,具体配置如下：

```js

var HappyPack = require('happypack'),
  os = require('os'),
  happyThreadPool = HappyPack.ThreadPool({ size: os.cpus().length });

modules: {
	loaders: [
	  {
        test: /\.js|jsx$/,
        loader: 'HappyPack/loader?id=jsHappy',
        exclude: /node_modules/
      }
	]
}

plugins: [
    new HappyPack({
      id: 'jsHappy',
      cache: true,
      threadPool: happyThreadPool,
      loaders: [{
        path: 'babel',
        query: {
          cacheDirectory: '.webpack_cache',
          presets: [
            'es2015',
            'react'
          ]
        }
      }]
    }),
    //如果有单独提取css文件的话
    new HappyPack({
      id: 'lessHappy',
      loaders: ['style','css','less']
    })
  ]

```

## 4、`DLL `& `DllReference`

针对第三方`NPM`包，这些包我们并不会修改它，但仍然每次都要在build的过程消耗构建性能，我们可以通过`DllPlugin`来前置这些包的构建.
我们使用`dllplugin`把第三方的NPM包生成一个名为 `manifest.json` 的文件，这个文件是用来让 `DLLReferencePlugin` 映射到相关的依赖上去的。在文件中引入该dll文件即可。
其原理是通过引用 `dll` 的 `manifest` 文件来把依赖的名称映射到模块的 id 上，之后再在需要的时候通过内置的 `__webpack_require__` 函数来 `require` 他们。

但对于`antd`这样的按需加载UI库，不能放在`dll`中，否则会全部打包进去，按需加载就无效了。

具体配置如下：

```js
///dll.entry.js 定义dll的入口
const DLL_ENTRY = {
	react: ['react', 'react-dom', 'react-router-dom', 'prop-types'],
	echarts: ['echarts'],
	vendor: ['mobx', 'mobx-react', 'axios'],
	db: ['dexie'],
};
// 定义需要dll 分离的包名
const DLL_CHUNKS_NAME = Object.keys(DLL_ENTRY);
module.exports = { DLL_ENTRY, DLL_CHUNKS_NAME };

//dll.config.js
const path = require('path');
const webpack = require('webpack');
const dllConstants = require('./dll.entry.js');
module.exports = {
	entry: dllConstants.DLL_ENTRY,
	output: {
		filename: '[name].dll.js', // 动态链接库输出的文件名称
		path: path.join(__dirname, '../dll'), // 动态链接库输出路径
		libraryTarget: 'var', // 链接库(react.dll.js)输出方式 默认'var'形式赋给变量 b
		library: '_dll_[name]_[hash]' // 全局变量名称 导出库将被以var的形式赋给这个全局变量 通过这个变量获取到里面模块
	},
	plugins: [
		new webpack.DllPlugin({
			// path 指定manifest文件的输出路径
			path: path.join(__dirname, '../dll', '[name].manifest.json'),
			context: __dirname,
			name: '_dll_[name]_[hash]' // 和library 一致，输出的manifest.json中的name值
		})
	]
};

//dll.utils.js

const path = require('path');
const constants = require('../conf/dll.js');
const webpack = require('webpack');
const CopyWebpackPlugin = require('copy-webpack-plugin');
const HtmlIncludeAssetsPlugin = require('html-webpack-include-assets-plugin');
//创建 dll 的关联包，返回[]
// 当我们需要使用动态链接库时 首先会找到manifest文件 得到name值记录的全局变量名称 然后找到动态链接库文件 进行加载
const createDllReferences = () => {
	const dllChunks = constants.DLL_CHUNKS_NAME;
	const tmpArr = [];
	dllChunks.forEach(item => {
		tmpArr.push(
			new webpack.DllReferencePlugin({
				manifest: require(path.join(__dirname, `../../dll/${item}.manifest.json`)) //)
			})
		);
	});
	return tmpArr;
};
//copy dll 的文件到输出目录
const copyDllToAssets = () => {
	const dllChunks = constants.DLL_CHUNKS_NAME;
	const tmpArr = [];
	dllChunks.forEach(item => {
		tmpArr.push({ from: `dll/${item}.dll.js`, to: 'dll' });
	});
	return new CopyWebpackPlugin(tmpArr);
};
//对dll资源添加相对html的路径
const addDllHtmlPath = () => {
	const dllChunks = constants.DLL_CHUNKS_NAME;
	const tmpArr = [];
	dllChunks.forEach(item => {
		tmpArr.push(`dll/${item}.dll.js`);
	});
	return new HtmlIncludeAssetsPlugin({
		assets: tmpArr, // 添加的资源相对html的路径
		append: false // false 在其他资源的之前添加 true 在其他资源之后添加
	});
};
module.exports = { createDllReferences, copyDllToAssets, addDllHtmlPath };

```

然后在`build.config.js`中加入dll插件：

```
dllUtils.copyDllToAssets(),
...dllUtils.createDllReferences(),
dllUtils.addDllHtmlPath(),

```

## 5、缓存`dll`

对于上文所说的，使用dll抽离第三方npm库可以加速打包，但还存在一种情况就是，dll可能很久不会改变，那每次`build`的时候都要重新生成dll包，要不然每次收到复制到指定目录。

参考`node_modules`的缓存机制，我们可以将生成的`dll`包缓存起来，每次检查对象`dll.entry.js`的`md5`值，只要dll的入口定义不变则认为无需生成新的dll包。具体配置就不写了，跟上面的差不多。


## 6、其他

- 开启`devtool: "#inline-source-map"`会增加编译时间
- `DedupePlugin`插件可以在打包的时候删除重复或者相似的文件，实际测试中应该是文件级别的重复的文件
- 减少构建搜索或编译路径
- 缓存与增量构建:`babel-loader`可以缓存处理过的模块，对于没有修改过的文件不会再重新编译，`cacheDirectory`有着2倍以上的速度提升，这对于rebuild 有着非常大的性能提升。


# 二、React运行优化

## 1、组件懒加载
