---
title: js代码片段难点集合
date: 2018-09-25 00:02:05
categories: js
tags:
 - js
---

### 如何理解

```js
function test(a,b,c) {
	var _args = [].slice.call(arguments);
	console.log(_args)
}
test(1,2,3)
```

### 如何封装一个柯里化通用式

思路柯里化运行的过程是一个参数收集过程，并在最里面处理

```js
// 简单实现，参数只能从右到左传递
function createCurry(func, args) {

    var arity = func.length;
    var args = args || [];

    return function() {
        var _args = [].slice.call(arguments);
        [].push.apply(_args, args);

        // 如果参数个数小于最初的func.length，则递归调用，继续收集参数
        if (_args.length < arity) {
            return createCurry.call(this, func, _args);
        }

        // 参数收集完毕，则执行func
        return func.apply(this, _args);
    }
}
```