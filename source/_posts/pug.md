---
title: pug
date: 2018-09-29 11:39:39
tags:
  - pug
---

### 命令行中的pug

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y2q710qj30ni0cftfy.jpg)


### 安装pug

全局安装pug
```bash
$ npm install --global pug
$ npm install --global pug-cli
```

编译pug文件

```bash
$ pug -P -w index.pug
```

- `-P` 编译后的html便于查看，保留空白节点
- `-w` 持续监控文件修改，修改马上编译

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y7qp6kyj30kf07lt9v.jpg)


### 类、ID、属性

```
h1#title(class='title2').title pug stydy
#test.test
input(checked,value='ye')
```

编译后
```html
<h1 class="title2 title" id="title">pug stydy</h1>
<div class="test" id="test"></div>
<input checked value="ye">
```

- 属性一般在标签后的括号里面，用逗号隔开。
- 类和ID有点特殊，可以直接在标签后面以链接的形式来写，当然由于也属于属性，也可以使用括号包裹的写法
- div标签可以省略， `div#test.test` 等价于 `#test.test` 。
- 对于没有属性值的直接写就行了，比如 input 里面的checked.

### 多行文本

```
p
  | 1. aa
  | 2. bb
  | 3. cc
```
或者
```
p.
  1. aa
  2. bb
  3. cc
```
编译后
```html
<p>
  1. aa
  2. bb
  3. cc
</p>
```

#### 多行文本中标签的写法分成两种

第一种是带点的纯文本的方式
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xs3bvz5j30iq047mxl.jpg)


第二种采用缩进的节点方式来写，把文本看做一个文本节点
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y1l7j0sj30q0049aaq.jpg)


### 注释

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xr1ktmmj30qg04jq4f.jpg)


对于IE的条件注释
)![pug5](./someImg/pug5.png

### 行内的style和script写法

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y61yqgbj30rf08n40d.jpg)


### 声明变量和替换数据

#### 在html中声明变量

括号里面可以对变量做一些js的处理
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y3gyuznj30rs06sq4i.jpg)



#### 命令行中传值

```bash
$ pug -P -w index.pug --obj '{"name":"xiangwang"}'
```
可以看到页面中接收到了 name 这个变量的值
>如果html中声明的变量和命令行中的变量同名，则html中的变量优先级高
>

#### 通过json文件传变量

创建一个json文件然后命令行中执行以下代码:
```bash
$ pug -P -w index.pug --O data.json
```

### 安全转义与非转义

`安全转义` 也就是有时候我们希望页面中显示标签，也就是html中是转义后的字符。
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xob48z8j30st05pgn5.jpg)

`非安全转义` 有时候我们就是希望html中显示的就是标签
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y8qdtjaj30ur03qdgt.jpg)


#### 其他的方式

和上面的结果一样
```
p= htmlData
p!= htmlData
```

如果我们有时候想页面中输出 `#{}`和`!{}` 怎么办呢？

```
p \#{htmlData}
p !#{htmlData}
```
就显示成了

```html
<p>#{htmlData}</p>
<p>!{htmlData}</p>
```

### 流程代码 for-each-while

#### for和each的使用方式

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xqrmd6lj30qb0ftn1j.jpg)


#### white使用方式

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y2yt5wsj30g805a0td.jpg)


### 流程代码 if-else-unless-swic

#### if-else 的使用方式

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xufv2gzj30o608djsu.jpg)


#### unless（除非）

如果是 false 就会往先执行
```
unless !what
  p #{what.length}
```

#### case-when

switch 语句是if-else-if 的兄弟语句,我们在 pug中可以使用 cass-when 来实现的
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xxvdq0oj30iq08xmys.jpg)


### mixin

使用 `mixin` 可以让pug的代码块重用,编译时会转换 javascript 中的函数，可以看做是一个函数，既然是函数，就可以传递参数
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y09jd83j30ky0bf76k.jpg)


#### mixin的嵌套

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y00ktpaj30rn083wgr.jpg)


#### 内联mixin代码块

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xsd3owhj30ge07tt9t.jpg)

解释：调用 `team`函数的时候，下面包含一个 `p Good job!` 就代表代码块`block`，注意缩进关系。函数中说明的是，如果有代码块就执行代码块，否则执行`p no team`。

除了代码块和文本，mixin还支持传递属性，比如传递classname
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y2gu38bj30ri0cpq6t.jpg)


### pug模板的继承

pug通过 `block` 和 `extends` 这两个关键字来实现模板的继承，一个块可以看做是一个block，它在子模板中来实现，同时支持递归。

```
//- layout.pug
html
  head
    title My Site - #{title}
    block scripts
      script(src='/jquery.js')
  body
    block content
    block foot
      #footer
        p some footer content
```

```
//- pet.pug
p= petName
```

```
//- page-a.pug
extends layout.pug

block scripts
  script(src='/jquery.js')
  script(src='/pets.js')

block content
  h1= title
  - var pets = ['cat', 'dog']
  each petName in pets
    include pet.pug
```

>include pet.pug 就相当于把 pet.pug中的内容放在了这里
>

编译 `page-a.pug` 后 page-a.html

```html

<html>
  <head>
    <title>My Site - </title>
    <script src="/jquery.js"></script>
    <script src="/pets.js"></script>
  </head>
  <body>
    <h1></h1>
    <p>cat</p>
    <p>dog</p>
    <div id="footer">
      <p>some footer content</p>
    </div>
  </body>
</html>
```

`block` 默认是替换`replace`，也就是不写的时候，当同名的时候会替换掉继承过来的 `block`，可以添加 `prepend` 或者 `append` 来实现 `block` 的合并

```
//- layout.pug
html
  head
    block head
      script(src='/vendor/jquery.js')
      script(src='/vendor/caustic.js')
  body
    block content
```

```
//- page.pug
extends layout.pug

block append head
  script(src='/vendor/three.js')
  script(src='/game.js')
```
编译 `page.pug` 后，生成 `page.html`

```html
<html>
  <head>
    <script src="/vendor/jquery.js"></script>
    <script src="/vendor/caustic.js"></script>
    <script src="/vendor/three.js"></script>
    <script src="/game.js"></script>
  </head>
  <body>
  </body>
</html>
```

### pug api

有五个核心的API：
  - pug.compile(source,options)
    - source:string  pug模板的字符
    - 返回的是一个函数，这个函数是来生成HTML的，对这个函数也可以传入本地变量 eg `pug.compile('div #{course}',{})({course:'pug'})`
  - pug.compileFile(path,options)
  - pug.compileClient(source,options)
  - pug.render(source,options)
    - source:string  pug模板的字符
    - options 可以直接传入一个变量对象 例如 `{course:'pug'}`
  - pug.renderFile(filename,options)
    - source:string pug模板的路径
    - options 可以直接传入一个变量对象 例如 `{course:'pug'}`

options参数是一个对象，可以设置以下属性，更多参考[pugAPI](https://pugjs.org/api/reference.html#pugcompileclientsource-options)

- filename:string  正在编译的文件的名称,默认为pug。
- self:boolean 保持一个变量到 self 的 locals (命名空间)
- debug:boolean 开启调试模式