---
title: Nuxt入门
date: 2018-10-12 10:05:54
categories: vue
tags:
  - SSR
  - Nuxt
---

### 全局CSS

>默认情况下 Nuxt 使用 vue-loader、file-loader 以及 url-loader 这几个 Webpack 加载器来处理文件的加载和引用。对于不需要通过 Webpack 处理的静态资源文件，可以放置在 static 目录中。

```js
//nuxt.config.js
css:['~assets/css/main.css']
```


### 路由传参

路由和vue中类似，不过Nuxt是根据page目录来动态生成路由表的，不需要我们配置。

![](http://ozrm3516s.bkt.clouddn.com/0a77ae84ac40924089a4329305f5e04d.jpg)

```html
<ul>
  <li><nuxt-link to="/">home</nuxt-link></li>
  <li><nuxt-link :to="{name: 'about', params:{id: 1}}">about</nuxt-link></li>
  <li><nuxt-link to="views">views</nuxt-link></li>
</ul>
```


### 动态路由

在路由里面定义带参数的动态路由。需要创建*以下划线为前缀*的Vue文件或目录

写法一：
```html
<nuxt-link :to="'/community/member/'+ user.id">
```
写法二：
```html
<nuxt-link :to="{name:'news-id', params:{id: 123}}">新闻1</nuxt-link>
```

> news这里是路由名称后面注意加上 -id,代表传的参数为 id字段，使用_id动态路由文件。

js跳转

```js
go(id) { 
  this.$router.push({ path: '/comments/' + id })
}
```

#### 动态路由参数校验

对于动态路由传递的参数，进行校验

```js
<script>
export default {
  validate({ params }) {
    return /^\d+$/.test(params.id)  //返回false 跳转到404
  }
}
</script>
```

### 路由切换动画效果



#### 全局路由添加动画

需要添加全局的CSS类,如下

```css
.page-enter-active, .page-leave-active {
    transition: opacity 2s;
}
.page-enter, .page-leave-active {
    opacity: 0;
}
```

#### 单个动画切换

添加一个唯一的类，并绑定到组件中

```css
.test-enter-active, .test-leave-active {
    transition: all 2s;
    font-size:12px;
}
.test-enter, .test-leave-active {
    opacity: 0;
    font-size:40px;
}
```

在需要动画效果的路由中绑定类
```js
<script>
export default {
  transition: 'test'
}
</script>
```

### 默认模板和默认布局

有时候页面中有一些公共的头部或者尾部，这时候可以考虑使用默认模板，或者默认布局

#### 默认模板

使用默认模板，需要在应用的根目录下创建一个 `app.html` 的文件

```html
<!--app.html-->
<!DOCTYPE html>
<html lang="en">
<head>
  {{ HEAD }}   <!--对应nuxt.config.js中的head字段,必须大写-->
</head>
<body>
  <P>测试</P>  <!--公共头部-->
  {{ APP }}   <!--APP代表page中的组件，必须大写-->
</body>
</html>

```

#### 默认布局

```html
<template>
  <div>
    <p>测试</p> <!--公共的头部-->
    <nuxt/>    <!--页面主体内容-->
  </div>
</template>
```

>区别：模板可以定制很多头部信息（比如IE版本判断），默认布局只能定义页面body中的元素

### 自定义错误页面

nuxt是有默认的错误页面的，有时间需要自定义就需要在 `/layouts/` 目录下建立一个`error.vue` 文件, 它相当于一个显示错误的组件。

```
<template>
  <div>
      <h2 v-if="error.statusCode==404">404页面不存在</h2>
      <h2 v-else>500服务器错误</h2>
      <ul>
          <li><nuxt-link to="/">HOME</nuxt-link></li>
      </ul>
  </div>
</template>
<script>
export default {
  props:['error'],
}
</script>
```

### 个性meta设置

在 `nuxt.config.js` 中 head字段设置的是全局头，如何针对单个页面进行设置？

```js
<script>
export default {
  hear(){
    return{
      title:'个性标题',
      meta:[
        {hid:'description',name:'news',content:'This is news page'}
      ]
    }
  }
}
</script>  
```

> 这里`hid:'description'`是一个唯一标识符，需要和全局的hid值相同，才会覆盖全局的值。

### 异步请求数据

```js
<script>
  export default {
    data(){
      return {

      }
    },
    async asyncData() {
      let { data } = await axios.get('https://api.myjson.com/bins/8gdmr')
      return { info: data }
    }
  }
</script>  
```

由于请求数据时组件不存在，这里也就没有this，return 的对象会直接合并到 data中, 在模板中直接绑定数据即可。
