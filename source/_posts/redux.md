---
title: redux
date: 2018-09-29 11:44:41
categories: react
tags:
  - redux
---

# 数据流

数据流是什么？为什么要用数据流？

- 数据流是我们的行为与响应的抽象
- 使用数据流可以帮助我们明确行为对应的响应

React与数据流的关系

- React 是纯V层的框架，不涉及任何的数据和控制，需要数据流进行支撑

主流数据流框架

- Flux

  - facebook 官方出的一个配合React来实现数据流的框架，属于单向数据流绑定，非常的大和重，实用性不强。

- reFlux

  - 属于 Flux 的一个第三方的框架。

- redux

  - 属于 Flux 的一个第三方的框架，简单 / 单一的状态树

# 单向数据流

不只是前端，很多系统开发的时候遵从的都是MVC分离，也就是数据放在Model里面，View来控制显示，Controler做整体的管理。但是随着系统的庞大，它会产生一系列问题。比如举个例子，我们上网shopping，提交订单，那么用户的账号，优惠信息，物流信息，库存等等的Model都会发生更新变化，然后View上的显示也会随之变化，反过来，View的有些变化也会对Model产生影响，这样就使用户下了一个订单以后界面会变得什么样变得不可预测。

![](http://ozrm3516s.bkt.clouddn.com/b8fcd75bd5fbbca99aee523f7784463b.png)


所以在React出现的同时Facebook也搞出了一个Flux `单向数据流` (React是纯V层框架，需要数据流进行支撑)，它的思想如下:它认为用户有各种各样的Action,然后所有的Action由一个统一的Dispacher分发到若干个Store里去，这个Store保存着数据也保存着页面的状态，根据数据和页面的状态，一个store只能向视图层传递信息，而不允许视图层再返回来作用到Store上，然后视图就发生更新，然后再由用户传入新的操作。这样子我们就能预测到Action作用到Store上时，View发生什么变化。

![](http://ozrm3516s.bkt.clouddn.com/62a1013ae2a3bb9da217c05a60155035.png)


Redux是Flux的一种实现方法，但是也有些许不一样，它的思想如下

![](http://ozrm3516s.bkt.clouddn.com/2bc8bf1315d61412884e2a7ec04c546e.png)


![](http://ozrm3516s.bkt.clouddn.com/3143ba299483bac48693d9bb2b0d6404.png)


当页面渲染完，UI就出现了，然后用户触发UI上的Action，然后Action被送到Reducer这个方法里去，然后Reducer更新了Store，Store里包含react开发的State，最后State决定UI层的展现。这就是Redux的一个完整过程。

# 项目结构

四个重要的文件夹

- actions:用户的行为
- components:
- container:
- reducer:负责分发axtions行为，根据用户的行为作出一个响应
- index.html
- server.js
- webpack

![](http://ozrm3516s.bkt.clouddn.com/0e4da2d309b967d93a4e2029ebc15bca.png)


安装

```bash
$ yarn add --save redux react-redux
```

redux本身就是一个工具流，而react-redux则是对redux的绑定。类似的还有ng2-redux、backbone-redux等

# action

- 是行为的抽象
- 是普通的JS对象
- 一般由方法生成

```javascript
const addTodo = (text) => {
  return {
    type: 'ADD_TODO',
    id:nextTodoId++,
    text
  }
}
```

# reducer

- 是响应的抽象
- 是纯方法 ： 可以完全的根据我们的输入来得到输出，非纯方法指的是，比如需要依赖系统的时间，依赖系统的状态。reducer里面只能是纯方法
- 传入旧状态state和action,返回新状态

```javascript
const todo = (state,action)=>{
  switch (action.type){
    case 'ADD_TODO':
      return{
        id:action.id,
        text:action.text,
        completed:false
      }
    case 'TOGGLE_TODO':
      if(state.id !== action.id){
        return state
      }
      return Object.assign({},state,{completed:!state.completed})
    default:
      return state
  }
}
```

# store

store 可以看做是 state 和 reducer 的集合，state是运行的时候才会有的，我们定义好的只有 reducer ,

- store 可以看做是所有数据和状态的存储
- action 作用于 store
- reducer 根据store 响应
- store 是唯一的
- store 包括了完整的 state
- state 完全可预测

```javascript
import {createStore} from "redux"
import todoApp from "./reducers"
let store = createStore(todoApp)
//调用 createStore来生成 store，需要传 reducers
```
