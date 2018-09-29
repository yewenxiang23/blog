---
title: AMD-CMD-CommonJs和ES6对比
date: 2018-09-24 23:58:29
tags:
  - 模块化规范
---

### AMD

AMD 是 RequireJS在推广过程中对模块定义的规范化产出,推崇依赖前置。

```javascript
define(['package/lib'],fucntion(lib){
	function(){
		lib.log('hello world!')
	}
	return {
	 foo: foo
	}
})
```
优点：代码一旦运行到此处，能立即知晓依赖。而无需遍历整个函数体找到它的依赖，因此性能有所提升。加载依赖的函数的时候是异步加载的，这样浏览器不会失去响应，它指定的回调函数，只有前面的模块都加载成功后，才会运行，解决了依赖性的问题。
缺点：就是开发者必须显式得指明依赖——这会使得开发工作量变大，比如：当你写到函数体内部几百上千行的时候，忽然发现需要增加一个依赖，你不得不回到函数顶端来将这个依赖添加进数组。


### CMD

CMD是SeaJS在推广过程中对模块定义规范化产出，推崇依赖就近。

```
//所有模块通过 define定义
define(function(require, exports. module){
	//通过 require 引入依赖
	var $ = require('jquery')
	var Spinning = require('./spinning')
})
```

### CommonJS

是node服务端的一个规范,只在服务端使用，浏览器并不支持。

```
export.area = function(r){
	return Math.PI * r * r;
}
```

### ES6 export/import
