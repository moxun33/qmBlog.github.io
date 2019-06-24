---
layout: react
title: React开发笔记
date: 2017-10-21 20:18:52
tags:
	 - React
	 - 笔记

---
+ 由于公司产品需要，最近一个月开始了 web 前端开发，技术栈是 webpack+react+mobx 的 react 全家桶架构。
 <!-- more -->

## 什么是 react

文章的开始还是有必要介绍一下 react 的概念。react.js 是 FaceBook 研发的前端 JS 框架，主要 特色是使用虚拟 DOM，性能卓越。其本质还是使用 javascript 作为开发语言。但需要注意的是现在一般使用 ES6的新特性作为主流开发。这里不详细阐述各个概念了，可以自行查看相关资料。

## 应用框架结构设计
这次的 react 开发应该算是我真正第一次独自负责，距离上一次的 react学习以及过去几个月的时间，当时react的版本 已经到了v15.6(现在 v16已经发布)，每个大版本的特性还是不大一样。尤其是 react-router4和 router3的区别很大，所以重新阅读最新的 react 开发文档是最稳妥的。由于这次的应用是对之前用传统 jquery 开发的
系统进行重构，所以在应用目录结构上是未知数。所有东西都是我自己去把握和决定。在开始的时候，使用了 koa2作为开发服务器，当时是基于这样的考虑：上线后可以继续使用 koa 作为 http 服务器，就不用配置 nginx 或者其他服务器，但几天后发现，很多东西还是跟传统的 mvc 框架的开发模式差不多，要定义很多的路由，使得整个开发效率大大降低。这种情况下，我决定推翻现有的架构，从零搭建一个 react 初始化模板。该模板的地址<https://github.com/moxun33/react-mobx-antd-boilerplate>。该模板使用的技术栈是 webpack+react+mobx+router4,前端 UI 是 ant-design。

## 组件化开发
我们都知道，在之前的jquery 开发的方式中，代码的组织和方式基本按照一个页面进行归类，不管这个页面需要多少的 js 代码都需要写在同一个文件，虽然有人会说我可以拆分一些出来啊，然而最后还是要引入到同个文件，而且文件之间的关系不够清晰，带来的问题就是维护难，代码可读性差。正是基于这个考虑，我们公司才决定使用 react 进行组件化开发。所谓的组件化管理，就是利用 ES6或之后的版本的 js 特性配合 react 的设计思想，对一个页面划分不同的组件，最后把这些组件导入到一个文件中，而且在组件化开发的过程中可以抽取一些比较通用的组件归并到组件的模块中，久而久之，就形成了我们自己系统的组件库了，对于主题定制和网站风格的定义还是比较有用 的。至于划分组件的标准是什么就不好定义了，打到一个页面，小到一个按钮都可以作为一个组件，在开发的过程中，会很自然的看出应该如何划分页面组件，但要记住的一点的，所有组件都是继承与 react 顶级 Component 类中。


## 高阶组件和 html 模板
当使用 react 进行组件化开发时，在不经意中会发现很多代码都是重复的（这里主要讲 react 组件的代码复用，忽略逻辑部分），那应该怎么提高组件代码复用率呢？

### 1、高级组件
+ 一个高阶组件只是一个包装了另外一个 React 组件的 React 组件。
这种形式通常实现为一个函数，本质上是一个类工厂（class factory），它下方的函数标签伪代码启发自 Haskell

```php
hocFactory:: W: React.Component => E: React.Component
```

这里 W（WrappedComponent） 指被包装的 React.Component，E（Enhanced Component） 指返回的新的高阶 React 组件。

定义中的『包装』一词故意被定义的比较模糊，因为它可以指两件事情：

(1) 属性代理（Props Proxy）：高阶组件操控传递给 WrappedComponent 的 props，
(2) 反向继承（Inheritance Inversion）：高阶组件继承（extends）WrappedComponent。

高阶组件允许你做：

a、代码复用，逻辑抽象，抽离底层准备（bootstrap）代码
b、渲染劫持
c、State 抽象和更改
d、Props 更改

在目前的开发中，我只用了高级组件来包裹网站的权限组件，轻松管理完整元素权限和逻辑权限问题；还有就是解决在不同页面使用相同布局问题的重复工作；至于其他的用途在用到时将继续更新。

### 2、html 模板
在很多时候，我们并不需要一个 react 组件去实现我们的一个视图，比如：自定义的列表，这个列表在所有地方都是一样的数据结构和样式，那么我们只需要把静态的 html 代码抽离到同个地方，抽象成一个工具函数，那么在使用的时候，传入模板数据就可以得到相同的视图效果而且不用重复考虑 css 布局问题。还有一个使用这种方式比较多的地方就是表单了，一个网站中表单有很多，但表单的输入框效果都大同小异，只要把当前的 react的 input 组件获取 ant-desing 的 input 组件等再高度定制成我们自己风格的 input 组件，那么在使用的时候，只需要传入特定的函数参数就可以 render 到一样的效果，而不用重复的使用 row 和 col 的去布局。而且这样做可以更好的管理css 样式和公用规则选项。这种方式，还是类似于我们以前的做法，但以前的 js 操作的 dom 是真实的 dom，性能很低，现在只是操作虚拟 dom，保证了网站的西能和开发效率。


## 遇到的坑
1、使用 HOC 时，传入的 wrappedComponent 必须的 React type 例如：

```php
const addBtn = (<p>hi!!!</p>)
  const AddProjectAuthBtn = wrapAuth(()=>addBtn)
  ```

常见错误是写成：

```php
const addBtn = (<p>hi!!!</p>)
  const AddProjectAuthBtn = wrapAuth(addBtn)
  ```
2、Javascript 中 formData 会把 null 对象转化成"null" 字符串;判断空对象{}的方法是
```php
const mobject = {}
JSON.stringify(mobject) === "{}"
```
不能直接使用 mobject !== {}或 null 这种方式。

3、ant-design 中 Select 组件同时设置了initialValue 和 placeholder,initialValue 为空的情况下 placeholder不生效,initialValue 不设置或设为 undefined，应该能解决。


4、mobx 中@inject('modalStore') @observer 需要放在其他注解的后面，否则无法更改状态

5、从 http 的 response 中读取图片并显示到 dom 上, 请求的时候 responseType="arrayBuffer"

```php
  getSupplierImageBuffer({id: attachId}).then((res) => {
             const arrayBufferView = new Uint8Array(res);
             return new Blob([arrayBufferView], {type: "image/jpeg"})
         }).then((myBlob) => {
             const urlCreator = window.URL || window.webkitURL;
             const objectUrl = urlCreator.createObjectURL(myBlob);
             screenshot.src = objectUrl;
		 })
		 ```

6、js 获取文件并处罚浏览器的下载功能, http 中 responseType="blob" 可以使用 download.js 或者其他的 js 文件下载库 当然 可以自定义工具
```php
fetch('http://somehost/somefile.zip').then(res => res.blob().then(blob => {
    var a = document.createElement('a');
    var url = window.URL.createObjectURL(blob);
    var filename = 'myfile.zip';
    a.href = url;
    a.download = filename;
    a.click();
    window.URL.revokeObjectURL(url);
}))
```

7、FormData 的 append 函数中,如果如果当前 key 对应的值是 null 或者 undefined，那么填入 FormData 中 的数据会变成 "null"或"undefined" 字符串。目前的解决办法时，只 append 飞非 null 的值。

8、react 中每次调用 setState 方法设置新状态都会出发 render 事件， 但并不一定刷新页面，当DOM 树发生变化才会刷新 UI.

9、样式的书写最好用 LESS,这样既可以定义主题也可以提高代码复用率。

10、在编码的过程中需要不断的抽取共同代码， 封装成通用组件，这样才能提高代码质量， 还可以武装专有的‘武器库’。
11、前端的 pdf 预览最好使用浏览器自带的预览功能， 使用 iframe 加载 PDF 文件流:

- 请求 responseType = 'arraybuffer'
- 响应处理  const data = window.URL.createObjectURL(new Blob([res], {type: "application/pdf"}))
- 预览组件

```
  <object data={this.state.contractPDFUrl} width={'200px'} height={'100px'}  >
        <embed src={this.state.contractPDFUrl}  width={'200px'} height={'100px'} />
   </object>
```

或者

 ``<iframe src={this.state.contractPDFUrl}  width={'400px'} height={'500px'} />``
 


``embed 在 windows 下兼容性价差: 在firefox上 无法渲染；iframe 兼容性较好，推荐。``
 
 12、避免使用 window.history 或者 window.location; 推荐使用 react-router 。 因为当加入了 mobx 状态管理器后， 使用 window.location刷新页面, 体验较差。

 13、 在 使用 mobx 下，当根据mobx 中的@observable isLogined 来判定授权与否， 当登录请求成功后，设置isLoginde
 d 后，立刻跳转的话， 会出现404的情况， 因为未登录下， 没有设置 router， 解决方法：延时300ms 左右再跳转，可临时解决该问题。推荐解决方法是：把登录页写入相同的路由配置中。

## 代码片段

 ```

/*、*
*
* 验证手机号码
* */
export const isMobilePoneValid = (str) => {
    const myreg = /^1[0-9]{10}$/;
    return myreg.test(str)
}

/**
 * 验证身份证号码
 *
 * */
export const isIDNumberValid = (str) => {
    const myreg = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
    return myreg.test(str)
}

/**
 * 函数去抖,针对winodw.onresize等事件
 * @param {Function} method 需要去抖的方法
 */
export const debounce = method => {
	if (method.timeout) {
		clearTimeout(method.timeout);
	}
	method.timeout = setTimeout(() => {
		method();
	}, 500);
};


/**
 * 求两个数组的交集,非对象数组
 * ES7  let intersection = allData.filter(v => editData.includes(v));
 * */
export const findArrayIntersection = (arr1, arr2) => {
	if (
		Object.prototype.toString.call(arr1) === '[object Array]' &&
		Object.prototype.toString.call(arr2) === '[object Array]'
	) {
		return arr1.filter(function (v) {
			return arr2.indexOf(v) !== -1;
		});
	}
};


/**
 * 对象数组的交集,  数据结构必须相同
 * */

export const findArrayIntersectionOfObject = (list1, list2, key,isUnion = true) => {
	if (list2 !== undefined && list2 !== undefined) {
		return list1.filter(a => isUnion === list2.some(b => a[key] === b[key]));
	} else {
		return [];
	}
};


/**
 * 识别是否为微信浏览器
 */
export const isWeixinBrowser = () =>
	/micromessenger/.test(navigator.userAgent.toLowerCase());

  /*
* 根据 key在  object中 获取指定子 object
*  
* key：所查询的 key
* value：查询 key 对应的 value
* */
export const findSubObjectInObject = (data, key, value) => {
	let target = {};
	for (let item in data) {
		const object = data[item];

		if (
			String(object[key]).includes(value) ||
			String(value).includes(object[key])
		) {
			target = object;
		}
	}
	if (!isObjectValid(target)) {
		openNotification('error', `无法获取${key}与${value}的映射`);
	}

	return target;
};

/**
 * 在新窗口打开给定的 html 元素（string）
 * */
export const openWinWithHtml = (myHtmlStr,spec = 'width=940,resizable=no,height=1200,location=no') => {
	const wnd = window.open('about:blank', '_blank', spec);
	wnd.document.write(convertStringToHtml(myHtmlStr));
};

/**
 * 自动调整图片宽高
 * */
export const autoSizeImg = (Img, maxWidth, maxHeight) => {
	const image = new Image();
	//原图片原始地址（用于获取原图片的真实宽高，当<img>标签指定了宽、高时不受影响）
	image.src = Img.src;
	// 当图片比图片框小时不做任何改变
	if (image.width < maxWidth && image.height < maxHeight) {
		Img.width = image.width;
		Img.height = image.height;
	} else {
		//原图片宽高比例 大于 图片框宽高比例,则以框的宽为标准缩放，反之以框的高为标准缩放
		if (maxWidth / maxHeight <= image.width / image.height) {
			//原图片宽高比例 大于 图片框宽高比例
			Img.width = maxWidth; //以框的宽度为标准
			Img.height = maxWidth * (image.height / image.width);
		} else {
			//原图片宽高比例 小于 图片框宽高比例
			Img.width = maxHeight * (image.width / image.height);
			Img.height = maxHeight; //以框的高度为标准
		}
	}

	return Img;
};


 ```



## 后续会继续更新