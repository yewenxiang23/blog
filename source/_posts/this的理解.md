---
title: this的理解
date: 2018-09-29 12:00:14
categories: js
tags:
---

### this的定义

- this的定义
  - this是在 **执行上下文** 创建时确定的一个在执行过程中不可更改的变量**
- 执行上下文
  - 执行上下文就是JavaScript引擎在执行一段代码之前将代码内部会用到的一些变量、函数、this提前声明然后保存在变量对象中的过程。

this只在 **函数调用阶段确定** ，也就是执行上下文创建的阶段进行赋值，保存在变量对象中。这个特性也导致了this的多变性:函数在不同的调用方式下都可能会导致this的值不同

### 严格模式和非严格模式的区别

```js
var a = 1;
function fun() {
   'use strict';
    var a = 2;
      return this.a;
}
fun();//😨报错 Cannot read property 'a' of undefined
```

```js
var a = 1;
function fun() {
    var a = 2;
      return this.a;
}
fun();//1
```

结论：**当函数独立调用的时候，在严格模式下它的this指向undefined，在非严格模式下，当this指向undefined的时候，自动指向全局对象(浏览器中就是window)**

### 在对象中使用this

```js
var a = 1000;
var obj = {
  a:1,
  b:this.a + 1
}
function func() {
  var obj = {
    a:1,
    c:this.a + 2
  }
  return obj.c
}
console.log(func()); //1002
console.log(obj.b); //1001
```
结论: **当obj在全局声明的时候,obj内部属性中的this指向全局对象，当obj在一个函数中声明时，严格模式下this指向undefined,非严格模式自动转为window**


### 一个例子加强理解

```js
var a = 1;
var obj = {
  a:2,
  b:function(){
    function fun() {
      return this.a
    }
    console.log(fun())
  }
}
obj.b();//1
```

解析:这个例子中 fun函数在obj.b方法中定义,为什么结果为1呢，开始我觉得很奇怪，想通后就理解了,上面的结论, **(当函数独立调用的时候，在严格模式下它的this指向undefined，在非严格模式下，当this指向undefined的时候，自动指向window)** , fun函数是在 obj.b 函数中独立调用的，所以这里的this是指向window的

### 作为对象的方法

```js
var a = 1;
var obj = {
  a: 2,
  b: function() {
    return this.a;
  }
}
console.log(obj.b())//2
```

另一个例子加强理解

```js
var a = 1;
var obj = {
  a: 2,
  b: function() {
    return this.a;
  }
}
var t = obj.b;
console.log(t());//1
```

>解释: obj.b属性储存的是对该匿名函数的一个引用（一个地址）,当赋值给t时，直接把地址复制给了t，调用t()  相当于直接调用了那个匿名函数
>

### 作为构造函数

- 如果函数作为构造函数使用，那么其中的this就代表它即将new出来的对象

```js
function Person(){
  this.name = 'yewenxiang';
  this.age = 24;
  this.sex = 'man';
  this.run = function(){
    return this.name + '正在跑步'
  }
}
Person.prototype = {
  constructor : Person,
  say:function (){
    return this.name + '正在说话'
  }
}
var  person1 = new Person()
person1.run()
person1.say()
```

> new 其实是一个语法糖
>

```js
//伪代码来看看new 做了什么
function Person(){
  //new 做的事情
  var obj = {}
  obj.__proto__ = Person.prototype //继承原型上的方法和属性

  obj.name = 'yewenxiang';
  ...     //对于构造函数中的this 一系列赋值,
  return obj
}

//1.创建一个临时的对象
//2.给临时对象绑定原型
//3.给临时对象对应的属性赋值
//4.返回一个临时对象
```

> prototype对象的方法中的this指向实例对象，因为实例对象的__proto__已经指向了构造函数的prototype,方法会沿着原型链进行查找
>

### 箭头函数

- 箭头函数的this不是在调用时候确定的，这是箭头函数的好处之一，因为他的this不会变来变去。箭头函数会捕获其所在上下文的this的值,箭头函数this在词法层面就完成了绑定。apply,call方法只是传入参数，却改变不了this。