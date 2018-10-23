---
title: node异步编程方案
date: 2018-09-24 23:51:30
categories: node
tags:
  - node
---

#### 回调函数  (第一阶段)

```js
function readFile (cb){
	fs.readFile(path, (err, data) => {
		if (err) console.log(err)
	    else console.log(data)
	})
}
readFile((err, data) => {
	if(!err){
		data = JSON.parse(data)
		console.log(data.value)
	}
})
```

#### promise (第二阶段)

```js
function readFileAsync(path) {
  return new Promise((resolve, reject) => {
    fs.readFile(path, (err, data) => {
      if (err) reject(err)
      else resolve(data)
    })
  })
}
readFileAsync('./package.json')
  .then(data => {
    data =JSON.parse(data)
    console.log(data)
  })
  .catch(err=>{
    console.log(err)
  })
```

#### 通过utli.promisify() 方法包装

只能传递一个异步回调函数，返回一个promise Function,第二个括号为传递的参数

```js
const utli = require('util')

utli.promisify(fs.readFile)('./package.json')
  .then(data => {
    return JSON.parse(data)
  })
  .then(data => {
    console.log(data.name)
  })
  .catch(err=>{
    console.log(err)
  })
```

### 结合使用 async await

```js
const readAsync = utli.promisify(fs.readFile)
async function init (){
  try{
    let data = await readAsync('./package.json')
    data = JSON.parse(data)
    console.log(data.name)
  } catch(err){
    console.log(err)
  }
}
init()
```

#### Generator

其本质是一个迭代器，下面来实现一个类似的迭代器

```js
function makeIterator (arr){
	let nextIndex = 0
	//返回一个迭代器对象
	return {
		next: () => {
			if(nextIndex < arr.length){
				return {value: arr[nextIndex++], done: false}
			} else {
				return {done: true}
			}
		}
	}
}
const it = makeIterator(['吃饭', '睡觉'])
console.log(it.next().value)  //吃饭
console.log(it.next().value)  //睡觉
```
生成器（Generator）：返回迭代器的函数,简化自己创建迭代器繁琐的过程，同时保持逻辑的清晰性。

```js
funciton *makeIterator (arr){
	for(let i = 0; i < arr.length; i++){
		yield arr[i]
	}
}
const gen = makeIterator(['吃饭', '睡觉'])
console.log(gen.next().value)  //吃饭
console.log(gen.next().value)  //睡觉
```

#### co 与 Generator  (第三阶段)

```js
const co = require('co')
const fetch = require('node-fetch')
co(function *(){
		const res = yield fetch('https://api.douban.com/v2/movie/1291843')
		const movie = yield res.json()
		const summary = movie.summary
		console.log(summary)
})
```
通过co 这个库，可以传递一个Generator函数,就可以通过同步的方式，来执行异步的过程。通过 yield 关键字可以实现一个状态的暂停，当 yield后面的异步代码没有执行完毕时，后面的代码不会执行。简单来说co 函数能让里面 yield 暂停的函数都能得到一步步的执行，**实现了 Generator 的自动执行**

#### async Function （第四阶段 统一世界）

```js
const = readAsync = utli.promisify(fs.readFile)
async function init (){
	let data = await readAsync('./package.json')
	data = JSON.parse(data)
	console.log(data.name)
}
```
