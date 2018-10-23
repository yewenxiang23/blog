---
title: Vuex入门
date: 2018-10-19 10:18:14
categories: vue
tags:
  - vuex
  - vue
---

### 简单的例子

```js
//store/index.js
import Vuex from 'vuex'
import Vue from 'vue'
Vue.use(Vuex)
const store = () => {
  return new Vuex.Store({
    state: {
      count: 0
    },
    mutations: {
      increment(state) {
        state.count++
      }
    }
  })
}
export default store
```

```js
//组件
this.$store.state.count  //获取状态
this.$store.commit('increment')  //修改状态
```

### State 和 mapStates

最简单的方法是在 `计算属性` 中返回某个状态

```js
computed: {
  count() {
    return this.$store.state.count
  }
}
```

但是状态多了之后，都申明为计算属性，显得会有些重复和冗余。为了解决这个问题，我们可以使用 mapState 辅助函数帮助我们生成计算属性。

```js
import { mapState } from 'vuex'

export default {
  //...
  computed: {
    ...mapState(['count'])
  }
}
```

mapState函数返回一个对象，通过对象结构的方式合并到computed中。

### Getter 和 mapGetters

getter可以认为是 store 的计算属性。

#### 创建Getter 
```js
const store = new Vuex.Store({
  state: {
    name: 'yewenxiang',
    age: 18
  },
  getters: {
    people(state, getters) {  //getters表示其他getter
      return `name:${state.name}, age:${state.age}`
    }
  }
})
```

#### Getter传参

可以通过让getter返回一个函数，来实现给getter传参

```js
//创建
getters: {
  people: (state) => (id) => {
    return console.log(id, state)
  }
}
//使用
this.$store.getters.people(1)  //不管state值是否改变，都会调用
```

#### 使用Getter

普通方式使用

```js
computed:{
  people(){
    return this.$store.getters.people
  }
}
```

mapGetters方式使用

```js
computed:{
  //...
  ...mapGetters(['people'])
}
```

### Mutation 和 mapMutations

更改Vuex的store中的状态只能使用mutation来更改，便于状态追踪和调试。

#### 创建Mutation
```js
const store = new Vuex.Store({
  state: {
    count: 1
  },
  mutations: {
    increment (state, params) { //params 为自己所传递参数
      // 变更状态
      state.count++
    }
  }
})
```

#### 使用Mutation

普通方式使用
```js
this.$store.commit('increment'， params) // 大多数情况params为对象
```

mapMutations方式使用

```js
import { mapMutations }  from 'vuex'
export default {
//...
methods:{
  ...mapMutations(['increment'])
  // 将 `this.increment()` 映射为 `this.$store.commit('increment')`
}
//...
}
```

#### Mutation 使用规范

- Mutation 必须是*同步函数*

每一条 mutation 被记录，devtools 都需要捕捉到前一状态和后一状态的快照，如果异步修改状态，当 mutation 触发的时候，状态还没有被修改，导致此时的状态的改变不可通过devtools追踪。

- 使用常量替代 Mutation 事件类型。

建立 `mutation-types.js` 文件，


- Mutation 需遵守 Vue 的响应规则。

1. 提前在你的 store 中初始化好所有所需属性。
2. 在对象上添加新属性时应该使用 `Vue.set(obj, 'newProp', 123)` 或者 `对象展开运算符` 来合并。

### Action 和 mapActions

Action 类似于 mutation，不同在于：
- Action 提交的是 mutation, 而不是直接变更状态。
- Action 可以包含任意的异步操作

创建一个异步的actions
```js
const store = () => {
  return new Vuex.Store({
    state: {
      count: 0,
      name: 'yewenxiang',
      age: 18
    },
    mutations: {
      increment(state) {
        state.count++
        state.age++
      }
    },
    actions: {
      asyncIncrement(context) {
        setTimeout(function() {
          context.commit('increment')
        }, 2000)
      }
    }
  })
}
```

context包含：
- commit 提交 mutation
- dispatch 提交 action
- getters
- rootGetters
- rootState
- state

可以使用 `this.$store.dispatch('xxx')` 来分发 action， 或者使用 mapActions。

```js
methods: {
  ...mapActions(['asyncIncrement']), //将 `this.asyncIncrement()` 映射为 `this.$store.dispatch('asyncIncrement')`
  addCount() {
    this.asyncIncrement()
  }
}
```