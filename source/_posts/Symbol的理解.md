---
title: Symbol的理解
date: 2018-09-29 12:01:07
categories: js
tags:
---

### Symbol是什么

- 它是JavaScript的第七种原始类型（ES6新特性，Symbol也是值，不是字符串，也不是对象。）

### Symbol 为什么会出现？

- 它能避免命名冲突

### 获取Symbol的三种方式

- 调用Symbol()
  - 这种方式每次调用都会返回一个新的唯一Symbol
- 调用Smybol.for(string)
  - 这种方式会访问Symbol注册表，其中储存了已经存在的一系列Symbol。这种方式与Symbol()定义的独立Symbol不同，Symbol()每次调用都返回一个新的并且唯一的Symbol。如果连续调用 Symbol.for('ye'),每次都会返回相同的Symbol。