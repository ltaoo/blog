---
title: 尝试实现懒加载图片组件
categories: React
tags:
- React-Naitve
- Image
date: 2016/12/04
---

类似知乎或者微信，在文章详情页面，图片并非是一次性加载完成，而是在滑动到快看到图片时才开始加载，很明显原生 Image 组件并不支持这种特性，所以需要封装一层。
<!--more-->
## 原理
首先图片组件必须放在 ScrollView 组件内。
图片组件在页面完成布局后，根据`onLayout`属性可以获取到这个组件在容器中的 x、y 值。
监听 ScrollView 滚动事件，`onScroll`会给出内容偏移量 contentOffset。当内容偏移量大于等于图片组件的 y 值减去屏幕高度时，就可以加载图片了。
![](lazyimg.png)
## 具体实现
首先搭建好页面，为了填充内容复制了知乎上的一篇文章[如何去阅读并学习一些优秀的开源框架的源码？](https://www.zhihu.com/question/26766601)。。。
```javascript
// imageLazyloadDemo.js
import React, {Component} from 'react'
import {
	StyleSheet,
	View,
	Text,
	// Image,
	ScrollView,
} from 'react-native'
import Image from '../components/Image'
export default class ImageDemo extends Component {
	constructor(props) {
		super(props);
		this.state = {
			y: 0
		}
		this._onScroll = this._onScroll.bind(this)
	}
	// 滑动触发
	_onScroll(e) {
		// 获取滑动的距离
		let {y} = e.nativeEvent.contentOffset;
		this.setState({
			y
		})
	}
	render() {
		return (
			<ScrollView 
				style = {styles.container}
				onScroll = {this._onScroll}
			>
				<Text style = {[styles.text, styles.title]}>假设这是文章详情页，存在很多文字和图片</Text>
				<Text style = {styles.text}>在我阅读的前端库、Python后台库的过程中，我们都是以造轮子为目的展开的。所以在最开始的时候，我需要一个可以工作，并且拥有我想要的功能的版本。</Text>
				<Text style = {styles.text}>紧接着，我就可以开始去实践这个版本中的一些功能，并理解他们是怎么工作的。再用git大法展开之前修改的内容，可以使用IDE自带的Diff工具：</Text>
				<Image
					source = {{uri: 'https://pic4.zhimg.com/5bc23b15e827a033d2b4966b6038d987_b.jpg'}}
					y = {this.state.y}
				/>
				<Text style = {styles.text}>或者类似于SourceTree这样的工具，来查看修改的内容。</Text>
				<Text style = {styles.text}>在我们理解了基本的核心功能后，我们就可以向后查看大、中版本的更新内容了。</Text>
				<Text style = {styles.text}>开始之前，我们希望大家对版本号管理有一些基本的认识。</Text>
				<Text style = {styles.text}>我最早阅读的开始软件是Linux，而下面则是Linux的Release过程：</Text>
				<Image
					source = {{uri: 'https://pic3.zhimg.com/f9d7c5343f3da040f891149a8993ed7e_b.png'}}
					y = {this.state.y}
				/>
				<Text style = {styles.text}>表格源自一本书叫《Linux内核0.11(0.95)完全注释》，简单地再介绍一下：</Text>
				<Text style = {styles.text}>版本0.00是一个hello,world程序</Text>
				<Text style = {styles.text}>版本0.01包含了可以工作的代码</Text>
				<Text style = {styles.text}>版本0.11是基本可以正常的版本</Text>
				<Text style = {styles.text}>1．项目初版本时，版本号可以为 0.1 或 0.1.0, 也可以为 1.0 或 1.0.0，如果你为人很低调，我想你会选择那个主版本号为 0 的方式；</Text>
				<Image
					source = {{uri: 'https://pic4.zhimg.com/6a32830fab3b9105fecef3d4f830afe7_b.png'}}
					y = {this.state.y}
				/>
			</ScrollView>
		)
	}
}

const styles = StyleSheet.create({
	container: {
		flex: 1,
	    backgroundColor: '#F5FCFF',
	    paddingHorizontal: 10
	},
	text: {
		fontSize: 16,
		marginVertical: 10
	},
	title: {
		fontSize: 20,
		marginTop: 10
	}
})
```

重点在`_onScroll`函数上，获取到每次滑动的内容偏移量 contentOffset.y，并通过 state 传递给 Image 组件。
```javascript
<Image
	source = {{uri: 'https://pic4.zhimg.com/5bc23b15e827a033d2b4966b6038d987_b.jpg'}}
	y = {this.state.y}
/>
```
这样 Image 组件就能够知道自己是否出现在可视范围内了。

这里 Image 组件是一个封装后的组件：
```javascript
// components/Image.js
import React, {Component} from 'react'
import {
	StyleSheet,
	View,
	Image,
	Text,
	Dimensions,
	InteractionManager
} from 'react-native'
// 获取屏幕宽高
const screenWidth = Dimensions.get('window').width
const screenHeight = Dimensions.get('window').height
export default class CustomImage extends Component {
	constructor(props) {
		super(props)
		// 先给图片一个默认宽高
		this.state = {
			width: screenWidth,
			height: 300,
			loaded: false
		}
		this._onLayout = this._onLayout.bind(this)
	}
	// 2、获取组件初始化位置
	_onLayout(e, node) {
		let {y} = e.nativeEvent.layout
		alert(y)
		this.setState({
			offsetY: y
		})
	}	
	// 获取加载中、加载失败视图的函数
	_renderLoad(text) {
		return(
			<View 
				style = {styles.loadContainer}
				onLayout = {this._onLayout}
			>
				<Text 
					style = {styles.loadText}
				>{text}</Text>
			</View>
		)
	}
	render() {
		const {source} = this.props
		if(this.state.loaded) {
			// 如果真正的图片加载好
			return <Image 
				source = {source}
				style = {{height: this.state.height}}
				resizeMode = {'contain'}
			/>
		}
		// 1、会先显示加载中视图
		return this._renderLoad('正在加载中...')
	}
}
const styles = StyleSheet.create({
	// 加载容器样式
	loadContainer: {
		backgroundColor: '#eee',
		justifyContent: 'center',
		alignItems: 'center',
		height: 100,
		marginVertical: 10
	},
	// 加载样式
	loadText: {
		fontSize: 20,
		color: '#ccc'
	}
)
```
1、先加载 loading 视图。
2、获取到该 loading 视图在 ScrollView 容器内的位置，有用的是 loading 距内容顶部的距离 y。

OK，然后可以开始计算了，如果传过来的 y + 屏幕高度 >= loading 视图距内容顶部的距离 y，就可以获取远程图片了。

```javascript
// components/Image.js
//...
	// 从网络请求图片的方法
	_fetchImg() {
		InteractionManager.runAfterInteractions(() => {
			const {source, style} = this.props
			if(source.uri) {
				// 如果是网络图片 在页面还未加载前，获取图片宽高
				Image.getSize(source.uri, (w, h) => {
					let imgHeight = (h/w)*this.state.width
			      	this.setState({
			      		height: imgHeight,
			      		loaded: true
			      	})
			    }, (err) => {
			    	// 获取图片宽高或者下载图片失败
			    	this.setState({
			    		loadFail: true
			    	})
			    })
			}
		})
	}

	render() {
		const {source, y} = this.props
		// 3、判断是否可以加载图片了
		if(y + screenHeight >= this.state.offsetY) {
			// 请求图片
			this._fetchImg()
		}
		// 4、如果加载了远程图片，就会渲染这里
		if(this.state.loaded) {
			// 如果真正的图片加载好
			return <Image 
				source = {source}
				style = {{height: this.state.height}}
				resizeMode = {'contain'}
			/>
		}
		// 1、会先显示加载中视图
		return this._renderLoad('正在加载中...')
	}
```
![](lazyImg.gif)
看起来还不错。。。

## 总结
至此，完成了一个最最最简单的图片懒加载组件。
当然还可以继续做一些优化，比如当网络不好时，加载图片显示加载失败，点击可以重新加载等等。

﻿