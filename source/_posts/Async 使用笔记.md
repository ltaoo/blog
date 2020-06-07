---
title: Async 使用笔记
categories:
	- JavaScript
tags:
	- async
date: 2017/01/07
---

之前虽然有过关于`async`使用的笔记，但是真正在项目中使用时，发现还是存在一些问题，所以重新对`async`进行更深入的学习。

<!--more-->

## 顺序执行

顺序执行是基本功能，因为`async`引入就是因为异步代码要顺序执行会非常麻烦，无论是`callback`还是`promise`都不是很直观，而使用`async`能够**写异步代码看起来像同步一样。**

不过我比较迷惑的是，在循环中，如何控制代码按顺序执行。假设有这么一个需求：
> 存在一个数组，保存了很多笔记对象，遍历该数组发送异步请求创建云端笔记，在发送请求前，需要先判断该笔记所属的笔记本是否存在，如果不存在则先创建笔记本。

```javascript
const noteAry = [{
	title: 'note1',
	notebook: 1
}, {
	title: 'note2',
	notebook: 1
}, {
	title: 'note3',
	notebook: 2
}]
// 逻辑
noteAry.forEach(note => {
	if(/*如果该笔记所属的笔记本没有创建*/) {
		// 就创建笔记本
		// 笔记本创建完成后，创建笔记
	}
})
```
笔记的创建是依赖于笔记本存在的，所以如果笔记本不存在，就一定要先创建笔记本，并在笔记本创建成功的回调中创建笔记。那问题来了，创建笔记的函数是不是需要调用两次？类似这样：
```javascript
// 请求库
const fetch = require('isomorphic-fetch')
// 将对象作为数据库处理的一个库
const lowdb = require('lowdb')

// 同步判断笔记本是否存在
function existsSync(id) {
	const db = lowdb('./db.json')
	return db.get('notebooks').find({id}).value()
}

// 创建笔记本函数
function createNotebook(notebook) {
	return new Promise((resolve, reject) => {
		fetch('http://127.0.0.1:3000/notebooks', {
			method: 'POST',
			body: JSON.stringify(notebook)
		})
			.then(res => res.json())
			.then(json => {
				resolve(json)
			})
			.catch(err => {
				reject(err)
			})
	})
}

// 创建笔记函数
function createNote(note) {
	return new Promise((resolve, reject) => {
		fetch('http://127.0.0.1:3000/notes', {
			method: 'POST',
			body: JSON.stringify(note)
		})
			.then(res => res.json())
			.then(json => {
				resolve(json)
			})
			.catch(err => {
				reject(err)
			})
	})
}

const noteAry = [{
	title: 'note1',
	notebook: 1
}, {
	title: 'note2',
	notebook: 1
}, {
	title: 'note3',
	notebook: 2
}]

// 先使用 promise 
noteAry.forEach(note => {
	if(!existsSync(note.notebook)) {
		console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
		// 如果笔记本不存在
		createNotebook(note.notebook)
			.then(res => {
				console.log(`笔记本${note.notebook}创建成功`)
				return createNote(note)
			})
			.then(res => {
				console.log(`笔记${note.title}创建成功`)
			})
			.catch(err => {
				console.log(err)
			})
	} else {
		console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
		createNote(note)
			.then(res => {
				console.log(`笔记${note.title}创建成功`)
			})
			.catch(err => {
				console.log(err)
			})
	}
})
```
打印的结果是：
```bash
笔记本1不存在，先创建笔记本
笔记本1不存在，先创建笔记本
笔记本2不存在，先创建笔记本
笔记本1创建成功
笔记本1创建成功
笔记本2创建成功
笔记note1创建成功
笔记note2创建成功
笔记note3创建成功
```
但是希望能够打印:
```bash
笔记本1不存在，先创建笔记本
笔记本1创建成功
笔记note1创建成功
笔记本1已经存在，直接创建笔记
笔记note2创建成功
笔记本2不存在，先创建笔记本
笔记本note3创建成功
笔记创建成功
```
而且数据库文件也存在三个笔记本数据。没有成功的原因很简单，**请求还未结束，就已经开始了下一次的循环，这时候数据库中还未写入新的笔记本，所以判断笔记本1还不存在**

先解决如何实现我们的预期需求。
引入一个临时变量`promise`：
```javascript
// 先使用 promise 
function realCreateNote(note) {
	return new Promise((resolve, reject) => {
		if(!existsSync(note.notebook)) {
			console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
			// 如果笔记本不存在
			createNotebook(note.notebook)
				.then(res => {
					console.log(`笔记本${note.notebook}创建成功`)
					return createNote(note)
				})
				.then(res => {
					console.log(`笔记${note.title}创建成功`)
					resolve(note.title)
				})
				.catch(err => {
					console.log(err)
					reject(err)
				})
		} else {
			console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
			createNote(note)
				.then(res => {
					console.log(`笔记${note.title}创建成功`)
					resolve(note.title)
				})
				.catch(err => {
					console.log(err)
					reject(err)
				})
		}
	})
}

function main() {
	// 临时变量
	let promise = Promise.resolve()
	noteAry.forEach(note => {
		promise = promise.then(() => realCreateNote(note))
	})

	return promise
}

main().then(() => {
	console.log('笔记均创建完成')
}).catch(err => {
	console.error(err)
})
```
能够实现预期的需求。

看起来还好，但是存在一个“复制代码”的问题，如果要修改笔记创建成功的`log`，却需要同时修改两处，如果还需要在笔记创建成功后进行处理，就要复制两份代码，肯定不好，但是却不能写成这样：
```javascript
function realCreateNote(note) {
	return new Promise((resolve, reject) => {
		if(!existsSync(note.notebook)) {
			console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
			// 如果笔记本不存在
			createNotebook(note.notebook)
				.then(res => {
					console.log(`笔记本${note.notebook}创建成功`)
				})
				.catch(err => {
					console.log(err)
					reject(err)
				})
		}
		console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
		createNote(note)
			.then(res => {
				console.log(`笔记${note.title}创建成功`)
				resolve(note.title)
			})
			.catch(err => {
				console.log(err)
				reject(err)
			})
	})
}
```
因为如果笔记不存在，`createNote`函数是必须在`createNotebook`函数的成功回调中，这样写很可能在笔记本还未创建成功时，就发送请求创建笔记，从打印的信息也能够看出问题所在：
```bash
笔记本1不存在，先创建笔记本
笔记本1已经存在，直接创建笔记
笔记本1创建成功
笔记note1创建成功
笔记本1已经存在，直接创建笔记
笔记note2创建成功
笔记本2不存在，先创建笔记本
笔记本2已经存在，直接创建笔记
笔记本2创建成功
笔记note3创建成功
笔记均创建完成
```

没有想到使用`promise`解决的方案。async`是否能够很好的处理这种情况？

## async
由于`async`能够暂停函数的执行，所以使用`await`返回的异步函数，肯定是已经请求结束的，所以如果使用`async`，上面的代码可以改成这样：
```javascript
// 使用 async
async function realCreateNote(note) {
	if(!existsSync(note.notebook)) {
		console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
		// 如果笔记本不存在
		try {
			await createNotebook(note.notebook)
			console.log(`笔记本${note.notebook}创建成功`)
		}catch(err) {
			console.log(err)
		}
	}
	console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
	try {
		await createNote(note)
		console.log(`笔记${note.title}创建成功`)
	}catch(err) {
		console.log(err)
	}
}

realCreateNote(noteAry[0])
```
直接调用该函数，能够正确打印出预期的信息：
```bash
笔记本1不存在，先创建笔记本
笔记本1创建成功
笔记本1已经存在，直接创建笔记
笔记note1创建成功
```
表示的确是按照顺序执行，再次调用也是没问题`realCreateNote(noteAry[1])`：
```bash
笔记本1已经存在，直接创建笔记
笔记note2创建成功
```
而在`forEach`循环中却并没有按照预期执行：
```javascript
noteAry.forEach(note => {
	realCreateNote(note)
})
```
```bash
笔记本1不存在，先创建笔记本
笔记本1不存在，先创建笔记本
笔记本2不存在，先创建笔记本
笔记本1创建成功
笔记本1已经存在，直接创建笔记
笔记本1创建成功
笔记本1已经存在，直接创建笔记
笔记本2创建成功
笔记本2已经存在，直接创建笔记
笔记note1创建成功
笔记note2创建成功
笔记note3创建成功
```

即使将`realCreateNote`函数返回`promise`实例，在循环内使用`await`关键字也不行：
```javascript
async function realCreateNote(note) {
	if(!existsSync(note.notebook)) {
		console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
		// 如果笔记本不存在
		try {
			await createNotebook(note.notebook)
			console.log(`笔记本${note.notebook}创建成功`)
		}catch(err) {
			console.log(err)
			return Promise.reject(err)
		}
	}
	console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
	try {
		await createNote(note)
		console.log(`笔记${note.title}创建成功`)
		return Promise.resolve(`笔记${note.title}创建成功`)
	}catch(err) {
		console.log(err)
		return Promise.reject(err)
	}
}

// realCreateNote(noteAry[1])

noteAry.forEach(async function (note) {
	try {
		await realCreateNote(note)
	}catch(err) {
		console.log(err)
	}
})
```

但是改成`for`循环后就可以了：
```javascript
// 使用 async
async function realCreateNote(note) {
	if(!existsSync(note.notebook)) {
		console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
		// 如果笔记本不存在
		try {
			await createNotebook(note.notebook)
			console.log(`笔记本${note.notebook}创建成功`)
		}catch(err) {
			console.log(err)
			return Promise.reject(err)
		}
	}
	console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
	try {
		await createNote(note)
		console.log(`笔记${note.title}创建成功`)
		return Promise.resolve(`笔记${note.title}创建成功`)
	}catch(err) {
		console.log(err)
		return Promise.reject(err)
	}
}
async function main() {
	for(let i = 0, len = noteAry.length; i < len; i++) {
		const note = noteAry[i]
		try {
			await realCreateNote(note)
		}catch(err) {
			console.log(err)
		}
	}
}
main()
```
> 这里之所以要添加`main`函数，是因为`await`关键字只能在`async`函数内使用。

打印的信息为：
```bash
笔记本1不存在，先创建笔记本
笔记本1创建成功
笔记本1已经存在，直接创建笔记
笔记note1创建成功
笔记本1已经存在，直接创建笔记
笔记note2创建成功
笔记本2不存在，先创建笔记本
笔记本2创建成功
笔记本2已经存在，直接创建笔记
笔记note3创建成功
```

## 总结

虽然使用`Promise`和`async`实现了相同的功能，但`async`在代码逻辑上和思维是一致：
```javascript
async function main() {
	for(let i = 0, len = noteAry.length; i < len; i++) {
		const note = noteAry[i]
		if(!existsSync(note.notebook)) {
			console.log(`笔记本${note.notebook}不存在，先创建笔记本`)
			// 如果笔记本不存在
			try {
				await createNotebook(note.notebook)
				console.log(`笔记本${note.notebook}创建成功`)
			}catch(err) {
				console.log(err)
			}
		}
		console.log(`笔记本${note.notebook}已经存在，直接创建笔记`)
		try {
			await createNote(note)
			console.log(`笔记${note.title}创建成功`)
		}catch(err) {
			console.log(err)
		}
	}
}
main()
```
所以相比`Promise`而言要更好，而且解决了一个使用`Promise`无法解决的“复制代码”的问题。

## 参考 
- [JavaScript Promise迷你书（中文版）](https://www.kancloud.cn/kancloud/promises-book/44249)

























