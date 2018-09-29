---
title: 生成器Generators
date: 2018-09-29 12:01:53
categories: js
tags:
---

### 什么是生成器?

```js
function* quips(name) {
  yield `你好${name}!`
  yield `这是一个生成器Generators`
  if(name.startsWith('y')){
    yield `你的名字${name}首字母是y`
  }
  yield '我们下次再见'
}
```

这段代码称为生成器函数，与普通函数的区别：

- 普通函数使用`function`声明,生成器函数使用 `function*`声明
- 在生成器函数内部，关键字 `yield` 类似于普通函数的 `return` 。普通函数只能 `return` 一次，而生成器函数可以 `yield` 一次或者多次。在生成器执行的过程中，遇到 `yield`表达式立即暂停，后续可恢复执行状态

> 最大区别：普通函数不能自暂停，生成器函数可以。
>

### 生成器做了什么？

```js
let value = quips('yewenxiang')   //返回一个已暂停的生成器对象
console.log(value.next())
//{value: "你好yewenxiang!", done: false}
console.log(value.next())
//{value: "这是一个生成器Generators", done: false}
console.log(value.next())
//{value: "你的名字yewenxiang首字母是y", done: false}
console.log(value.next())
//{value: "我们下次再见", done: false}
console.log(value.next())
//{value: undefined, done: true}
```

上面调用 `quips('yewenxiang')`时，并非立即执行，而是返回一个已暂停的生成器对象，并且赋值给了 `value` 每当你调用生成器对象的 `.next()` 方法时，函数调用将其自身解冻并一直运行到下一个yield表达式，再次暂停。

> 专业术语描述: 每当生成器执行yields语句，生成器的堆栈结构（本地变量、参数、临时值、生成器内部当前执行的位置）被移出堆栈。然而，生成器对象保留了这个堆栈结构的引用，所以后面调用 .next() 可以重新激活堆栈结构并且继续执行。
>
