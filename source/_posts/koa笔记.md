---
title: koa笔记
date: 2018-09-24 23:57:23
categories: node
tags:
  - koa
---

### Context对象

Koa提供一个Context对象，表示一次对话的上下文（包括HTTP请求和HTTP回复），通过给这个对象赋值可以控制返回给用户的内容。

eg:

```js
const Koa = require('koa');
const app = new Koa();

const main = ctx => {
  ctx.response.body = 'Hello World';
};
app.use(main);
app.listen(3000);
```

### 给响应的body 添加内容

Koa默认返回的类型是 `text/plain`。 
可以使用 `ctx.request.accepts('json')` 来判断客户端接收什么数据，同时设置返回的类型 `ctx.response.type = 'json'`。

```js
ctx.response.body = 'Hello World';               //text
ctx.response.body = { data: 'Hello World' };     //json
ctx.response.body = '<p>Hello World</p>';        //html
ctx.response.body = '<data>Hello World</data>';  //xml

```

### 网页模板

Koa先读取模板文件，然后返回给用户

```js
ctx.response.body = fs.createReadStream('./demos/template.html');

```

### 路由

原生路由是通过 `ctx.request.path` 可以获取用户请求的路径,由此判断该返回什么内容给用户，使用不方便，一般使用 `koa-route`.

```js
const about = ctx => {
  ctx.response.type = 'html';
  ctx.response.body = '<a href="/">Index Page</a>';
};
app.use(route.get('/about', about));
```

### 静态资源

如果服务器需要返回给用户一些静态资源（图片，字体，样式表，脚本），一个个写路由很麻烦，可以使用 `koa-static` 。

```
const Koa = require('koa');
const app = new Koa();
const path = require('path');
const server = require('koa-static')
const publicServer = server(path.join(__dirname) + '/public');
app.use(publicServer);
app.listen(3000);
```

这样用户可以访问  `__dirname` 文件夹下的所有文件。
eg:输入`http://localhost:3000/01.js` ，可以查看 `__dirname` 文件夹下的 `01.js` 文件。

### 路由重定向

`ctx.response.redirect('/')` 方法可以发出一个302跳转,将用户导向另一个路由。

### 中间件

Koa最重要的一个设计就是中间件，比如打印日志中间件简单的写法，可以直接main函数中写 `console.log(...)`， 也可以拆分成一个独立函数（如下）

```js
const logger = (ctx, next) => {
  console.log(`${Date.now()} ${ctx.request.method} ${ctx.request.url}`);
  next();
}
const main = ctx => {
  ctx.response.body = 'Hello World';
};
app.use(logger);
app.use(main);
app.listen(3000);

```

上面的代码 `logger` 函数就叫 `中间件`。

>中间件：处于 HTTP Request 和 HTTP Response 中间，用来实现某种中间功能， app.use() 用来加载中间件。

中间件默认有两个参数，`(Context, next)`, 只要调用 `next()` 就把执行权交给下一个中间件。

```js
const one = (ctx, next) => {
  console.log('>> one');
  next();
  console.log('<< one');
}
const two = (ctx, next) => {
  console.log('>> two');
  next(); 
  console.log('<< two');
}
const three = (ctx, next) => {
  console.log('>> three');
  next();
  console.log('<< three');
}
app.use(one);
app.use(two);
app.use(three);
```
三个中间件: one中间件 、 two中间件 、 three中间件。
执行步骤 ：最外层 one 首先执行,调用next()方法后,next()方法后面的代码并不会执行,而是把执行权交给下一个中间件 two,同理交给 three 。我的理解是：next() 后面的代码被放入了任务队列，当主线程也就是 next() 之上的代码执行完毕后，执行任务队列的代码，任务队列里面的代码以 `先进后出` 的顺序执行。

>如果删除 two 中间件函数中的 next() ,执行权并不会交给 three 中间件，也就是说 three并不会被执行。

### 异步中间件

```js
const fs = require('fs.promised');
const main = async function (ctx, next) {
  ctx.response.type = 'html';
  ctx.response.body = await fs.readFile('./demos/template.html', 'utf8');
};
```

### 中间件合成 koa-compose

```js
const compose = require('koa-compose')

const logger = (ctx, next) => {
  console.log(ctx);
  next()
}
const main = ctx => {
  ctx.response.body = 'Hello World'
}
const middlewares = compose([logger, main])
app.use(middlewares)
```

### 错误处理

如果代码运行过程中发生错误，我们需要把错误信息返回给用户，可以使用 `ctx.throw(statusCode)` 来返回,  `ctx.response.status = 404` 等价于 `ctx.throw(404)` 。

为了方便处理错误，最好使用 `try...catch`, 每个中间件写太麻烦，可以让最外层的中间件负责所有处理

```js
const handler = async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    ctx.response.status = err.statusCode || err.status || 500;
    ctx.response.body = {
      message: err.message
    };
  }
};
const main = ctx => {
  ctx.throw(500);
};
app.use(handler);
app.use(main);

```
以上代码，由于 main中间件中抛出了错误，会执行 最外层中间件handler catch里面的代码。

##### 错误事件监听

代码运行过程中出错，Koa会触发一个 `error` 事件。

```js
//错误事件监听
app.on('error', (err, ctx) =>
  console.error('server error', err);
);
```

如果错误被 `try...catch` 捕获， 就不会触发error事件，必须调用 `ctx.app.emit()` 手动释放 error 事件，才能让监听函数生效。

```js
const handler = async (ctx, next) => {
  try {
    await next();
  } catch (err) {
    //...
    ctx.app.emit('error', err, ctx);
  }
}
```

### 读写 cookies

```js
const main = ctx => {
  ctx.cookies.get('view');
  ctx.cookies.set('view', n);
}
```

### koa-body 提取键值对

他可以用来提取POST请求体中的键值对,类似于 `name=Jack`

```
const koaBody = require('koa-body');
const app = new Koa();

const main = async function(ctx) {
  const body = ctx.request.body;
  console.log(body)
  if (!body.name) ctx.throw(400, '.name required');
  ctx.body = { name: body.name };
};
app.use(koaBody());
app.use(main);
```

### 获取GET请求中的查询参数

- `ctx.request.query`
- `ctx.query`

### koa-view使用

```
const views = require('koa-views')
const { resolve } = require('path')

app.use(views(resolve(__dirname, './views'),{
	extension:pug
})
app.use(async (ctx, next) => {
    await ctx.render('index',{
        title:'ye'
    })
})
```
