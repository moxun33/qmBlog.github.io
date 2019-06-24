---
title: SVG入门与实践
toc: true
date: 2018-05-19 11:21:42
tags:
     - SVG
     - 前端  
---

+ 今天来聊聊SVG技术。本文先简单阐述svg的概念和一些特性， 再分享一些我在项目实战中用到的svg例子。
<!-- more -->

# 一、入门

## 概念

Scalable Vector Graphics (SVG) 可扩展矢量绘图，是一种用来描述二维矢量图形的XML标记语言。

## 特性

SVG 可被非常多的工具读取和修改（比如记事本）
SVG 与 JPEG 和 GIF 图像比起来，尺寸更小，且可压缩性更强。
SVG 是可伸缩的
SVG 图像可在任何的分辨率下被高质量地打印
SVG 可在图像质量不下降的情况下被放大
SVG 图像中的文本是可选的，同时也是可搜索的（很适合制作地图）
SVG 可以与 Java 技术一起运行
SVG 是开放的标准
SVG 文件是纯粹的 XML
SVG 的主要竞争者是 Flash。
与 Flash 相比，SVG 最大的优势是与其他标准（比如 XSL 和 DOM）相兼容。而 Flash 则是未开源的私有技术。

## 基础形状

矩形 ``<rect>``
圆形 ``<circle>``
椭圆 ``<ellipse>``
线 ``<line>``
折线 ``<polyline>``
多边形 ``<polygon>``
路径 ``<path>``

## 使用方式

 将SVG文件作为图片，与普通图片使用方式一致；但不会加载自身引用， 如字体、图标等等外部资源。

 将SVG作为应用程序，SVG文件也可以作为``<object>``元素的data属性引入HTML中。但是，MIME type必须是``image/svg+xml``。

 混合文档, 直接跟``HTML``标签混合写入， 会继承父文档的样式，默认以``inline``方式显示。


## 使用总结

在我实际使用svg的过程中， 除了``path``， 我觉得实现起来都不会太难。 path的可以用于十分复杂的场景，也十分灵活， 但学习曲线比较陡。接下来就一起看看path的一些实践例子。


# 二、实践

## svg实现波浪、水泡效果

### 先上效果图

![](https://ws2.sinaimg.cn/large/006tKfTcgy1frx8n76mhrg30j30atb29.gif)

### 实现代码(Reactjs中)

#### 1、波浪的实现

创建单个波浪的svg,``wave.svg``(以下css的背景图引用) ， 主要使用``path``的``C``指令绘制3次贝塞尔曲线， 其他指令可查阅svg教程。



```php

<svg xmlns="http://www.w3.org/2000/svg" width="1600" height="698">
    <defs>
        <linearGradient id="a" x1="50%" x2="50%" y1="-10.959%" y2="100%">
            <stop stop-color="#ffffff" stop-opacity=".25" offset="0%"/>
            <stop stop-color="#ffffff" offset="100%"/>
        </linearGradient>
    </defs>
    <path fill="url(#a)" fill-rule="evenodd" d="M.005 121C311 121 409.898-.25 811 0c400 0 500 121 789 121v77H0s.005-48 .005-77z" transform="matrix(-1 0 0 1 1600 0)"/>
</svg>

```

创建4个``div``

```php
<div className="ocean">
	<div className="wave"></div>
	<div className="wave"></div>
	<div className="wave"></div>
	<div className="wave"></div>
</div>

```

编写样式

```css
.ocean {
  opacity: .4;
  height: 320px;
  width: 100%;
  position: absolute;
  bottom: 0;
  left: 0;
  z-index: 1;
}


.wave {
  background: url('../../assets/login/svg/wave.svg') repeat-x;
  position: absolute;
  top: 119px;
  width: 12800px;
  height: 318px;
  animation: wave 4.6s cubic-bezier(0.36, 0.45, 0.63, 0.53) -.125s infinite;
  transform: translate3d(220, 110, 0);
  opacity: 1;
}
.wave:nth-of-type(2) {
  top: 125px;
  animation: wave 5.5s cubic-bezier(0.36, 0.45, 0.63, 0.53) -.125s infinite;
  opacity: .6;
}
.wave:nth-of-type(3) {
  top: 130px;
  animation: wave 7s cubic-bezier(0.36, 0.45, 0.63, 0.53)  -.125s infinite;
  opacity: .4;
}
.wave:nth-of-type(4) {
  top: 135px;
  animation: wave 6s cubic-bezier(0.36, 0.45, 0.63, 0.53) infinite;
  /* animation: wave 6s cubic-bezier(0.36, 0.45, 0.63, 0.53) -.125s infinite, swell 7s ease -1.15s infinite;*/
  opacity: .3;
}
@keyframes wave {
  0% {
    margin-left: 0;
  }
  100% {
    margin-left: -1600px;
  }
}
@keyframes swell {
  0%, 100% {
    transform: translate3d(0, -115px, 0);
  }
  50% {
    transform: translate3d(0, 5px, 0);
  }
}
```




设定每个``div``的位置和动画， 动画``wave``表示波浪的作用位置，而且无限循环，动画``swell``表示波浪的变换。就是如此简单！

2、水泡的实现

创建单个水泡（ES6）

```php
//单个水泡
createPop = (i) => {
		let bottom = Math.round(Math.random() * 100).toFixed(2);
		bottom = bottom > 55 ? 55 : bottom;
		let left = Math.round(Math.random() * 100).toFixed(2);
		let second = Math.random() * 10;
		return (
			<svg width="8px" height="8px" version="1.1"
				 id={`login-pop${i}`}
				 style={{
					 position: 'absolute',
					 bottom: `${bottom}%`,
					 left: `${left}%`,
					 animation: ` popup ${second}s cubic-bezier(0.36, 0.45, 0.63, 0.53) -.125s infinite`
				 }}
				 xmlns="http://www.w3.org/2000/svg">
				<circle cx='4px' cy='4px' r="4"
						fill='rgba(255,255,255,0.2)'/>
			</svg>
		);
    };
```
创建``svg``画布，大小8，使用``<circle>``创建圆，半径是4。内联样式，规定布局位置是: ``absolute``, 具体坐标是随机生成的，再加入动画，无限循环。

创建多个水泡     

```php    
	//生成指定个数的随机位置水泡
	createRandomPops = () => {
		let arr = [];
		for (let i = 0; i < 40; i++) {
			arr.push(i);
		}
		return arr.map((s, i) => {
			return <div key={i}>
				{this.createPop(i)}
			</div>;
		});
	};

```

在Reactjs的渲染中调用

```php
{this.createRandomPops()}

```

水泡的样式

```css
@keyframes popup {
  0%{
    bottom: 16%
  }
  100%{
    bottom:  55%;
  }
}

```

水泡只需要向上冒即可，开始从底部16%的位置， 终止位置大约在页面中间，无限循环。


## svg实现渐变进度条

### 效果图

![](https://ws1.sinaimg.cn/large/006tKfTcgy1frx8ozts40j304o04rdfo.jpg)

* 图中的进度条是由左右两个半圆组合而成。

### 代码实现（Reactjs）

绘制扇形

```php

    /**
    * 扇形绘图方法,计算终点的 x,y 坐标
    *
    * @param Number progr  进度值， 小于100的整数
    * @param String direction 左半圆还是有半圆
    * @param cx, cy, r默认的x、y坐标， r是半径
    * @return 返回path中d指令参数
    */
	sector = (progr = 0, direction = 'right', cy = 63, cx = 220, r = 156) => {
		let progress = progr > 100 ? 100 : (progr < 0 ? 0 : progr);
		// 计算当前的进度对应的角度值
		const degrees = progress * (360 / 100);
		// 计算当前角度对应的弧度值
		let rad = degrees * (2 * Math.PI / 360);
		let startY = cy;
		let x2 = cx,     //结束X点
			y2 = cy;       //结束Y点
		//   sweepFlag   1 顺时针，0逆时针
		if (direction === 'right') {
			//右边圆环
			if (progress > 50) {
				//多于50%，右边圆环满进度
				x2 = cx;
				y2 = startY + r * 2;
			} else {
				x2 = cx + r * Math.sin(rad);//计算x轴
				y2 = startY + r - r * Math.cos(rad);//计算y轴
			}
		} else {
			//左边圆环
			//需要100-progress
			if (progress > 50) {
				const leftRad = (100 - progress) * (360 / 100) * (2 * Math.PI / 360);
				//console.log(leftRad,811)
				startY = startY + r * 2;
				x2 = cx - r * Math.sin(leftRad);//计算x轴
				y2 = startY - r - r * Math.cos(leftRad);//计算y轴
			}
		}
		return ['M', cx, startY, 'A', r, r, 0, 0, 1, x2.toFixed(0), y2.toFixed(0)];
	};

```
关键点：由于是动态的进度，x、y的值必须是动态计算，这里包含是数学知识是：在圆中根据角度计算坐标值。``M``指令是移动光标到初始位置的坐标， ``A``指令是绘制一条曲线。

在React中定义svg

```php

			<div className="progress-wrap">
				<svg width="160" height="160" viewBox="0 0 440 440" className="center">
					<defs>
						<linearGradient x1="0" y1="0" x2="1" y2="1" id={'left' + idx}>
							<stop offset="100%" style={{stopColor: colors.max}}></stop>
							<stop offset="0%" style={{stopColor: colors.middle}}></stop>
						</linearGradient>
						<linearGradient x1="1" y1="1" x2="0" y2="0" id={'right' + idx}>
							<stop offset="0%" style={{stopColor: colors.mini}}></stop>
							<stop offset="100%" style={{stopColor: colors.max}}></stop>
						</linearGradient>
						<filter id="f1" x="0" y="0" height="200%" width="200%">
							<feOffset result="offOut" in="SourceAlpha" dx="4" dy="4"/>
							<feGaussianBlur result="blurOut" in="offOut" stdDeviation="8"/>
							<feBlend in="SourceGraphic" in2="blurOut" mode="normal"/>
						</filter>
						<linearGradient id={'right-' + idx} gradientUnits="userSpaceOnUse" x1="220" y1="60" x2="220"
										y2="375">
							<stop offset="0" style={{stopColor: colors.mini}}></stop>
							<stop offset="1" style={{stopColor: colors.middle}}></stop>
						</linearGradient>
						<linearGradient id={'left-' + idx} gradientUnits="userSpaceOnUse" x1="0" y1="60" x2="0"
										y2="375">
							<stop offset="0" style={{stopColor: colors.max}}></stop>
							<stop offset="1" style={{stopColor: colors.middle}}></stop>
						</linearGradient>
					</defs>
                    <g>
                    /* 进度条的底色 */
						<circle cx="220" cy="220" r="160" strokeWidth="50" stroke="#e5e8f8" fill="none"
								strokeDasharray="1005 1005"></circle>
						<path d={`${this.sector(percent).join(' ')}`} stroke={`url('#right-${idx}')`} fill="none"
							  strokeWidth="40"></path>
						{percent > 50
						&&
						<path d={`${this.sector(percent, 'left').join(' ')}`} stroke={`url('#left-${idx}')`} fill="none"
							  strokeWidth="40"></path>}
						<circle cx="220" cy="220" r="197" strokeWidth="40" stroke="#f5f6fa" fill="none"
								strokeDasharray="1256 1256"></circle>
					</g>
				 
				</svg>
				<p className="percent-text  " style={{color: colors.max}}>
					<span className={'percent-number'}>
						{percent}
					</span>
					<span className={'percent-symbol'}>
						%
					</span>
				</p>
            </div>
            
```
 ``linearGradient``是表示创建渐变；

 ``g``中的``circle``表示进度条的底色和外部包裹圆环；当进度大于50%时才创建左半圆，边框的宽度都是40px；

 colors是预定义的几种颜色，用于渐变 ：``const colors = {max: '#3cc8bc', middle: '#27aea3', mini: '#9eece7'}``；

百分比显示在圆环中间。

# 总结

本文通过两个简单例子展示svg的强大之处。总的来说，svg不难， 只是涉及的知识比较多， 多实践，就熟练了。

