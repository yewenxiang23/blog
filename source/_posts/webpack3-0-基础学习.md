---
title: webpack3.0+基础学习
date: 2018-09-25 00:05:17
tags:
  - webpack
---


### **一个webpack demo**
 首先是建立项目结构
 
 - 根目录新建 src 文件夹 (开发环境时的代码)
 - 根目录新建 dist 文件夹（生产环境的代码）
 
webpack 打包的本质是把 **src**下的入口文件 entry.js 文件，及其相关联的文件,打包成bundle.js文件，并且放在 **dist** 文件夹下面

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xpb0u3ij30kv0bhwfm.jpg)

webpack 的打包命令：

- 如果是全局安装的webpack : `$webpack src/entry.js dist/bundle.js`。
- 如果是项目中安装的webpack : `$./node_modules/.bin/webpack src/entry.js dist/bundle.js`

一般我们webpack不会使用全局安装的方式，会造成版本冲突导致的问题。而使用项目中安装的webpack 命令行每次打包输入又太长，可以将命令行写进 `package.json` 文件里面的 `script` 标签中：

```json
"scripts": {
    "bundle":"./node_modules/.bin/webpack src/entry.js dist/bundle.js"
  },
```
这样每次打包只需要 `$npm run bundle` 就可以了。
> 小技巧： bundle 改成 start ，只需要 `npm start`。
> 全局安装 `live-server` 这个插件可以在本地跑一个服务器，使项目跑在 `8080` 这个端口。

### **webpack配置文件**

在项目的根目录新建 `webpack.config.js` 文件，配置的基本结构：

```js
module.exports = {
    entry: {},
    output: {},
    module: {}, //模块解读css，打包css，图片转换压缩等配置。
    plugins: [], //插件
    devServer: {}, //配置开发服务
} 
```
上面将入口出门配置信息写进了 `script` 脚本里面，然后使用 `npm start` 来跑这段脚本，也就是跑这段命令，这种方式来配置是**非常单一**的，没有更多的配置选项，**如果做多入口多出口的配置就无法实现了**。正确开发的姿势是将配置信息写进 `webpack.config.js` 文件中：

```js
const path = require('path'); //注意引入 path 模块
module.exports = {
    entry: {
        entry:'./src/entry.js' //入口文件路径
    },
    output: {
        path:path.resolve(__dirname,'dist'), //出口文件路径 
        filename:'bundle.js' //打包后的文件名
    }
} 
```
`path.resolve(__dirname,'dist')` 代码解读：

-  `path`：node核心模块之一,需要引入`path` 。
- `__dirname`：当前文件所在目录的完整绝对路径。
- `resolve`：`resolve` 会将参数中的路径或路径片段的序列解析为一个**绝对路径**，这样即使项目迁移，地址变更，只要保证相对路径正确即可。
- 代码解读：出口文件存放路径为当前文件夹下的 `dist` 文件夹中。

### **多入口 多出口 配置**
在 `src` 下新建一个 `entry2.js` 文件，也就是第二个入口文件，加入js代码，更改配置。
```js
entry: { //固定命名
        entry: './src/entry.js', //这里的entry名字是自己定义的
        entry2: './src/entry2.js',  
    },
    output: {
        path:path.resolve(__dirname,'dist'),
        filename:'[name].js' //[name] 打包的出口文件名 和 入口文件名是一样的
    },
```
更改 `dist/index.html` 中引入的js文件，就可以查看效果了。
>注意：两个入口肯定需要两个出口文件对应

### **服务和热更新**
自己配置一个本地的服务器，首先需要在项目中安装 `webpack-dev-server` 这个包。
```bash
npm i webpack-dev-server --save-dev
```
shell 中输入命令 `webpack-dev-server` ，是无法识别的，因为没有全局安装，环境变量中也就没有存在命令所在的目录。我们需要再 `package.json` 中加入:
```json
"server":"./node_modules/.bin/webpack-dev-server --open"
```
> 后面加上参数 `--open` 运行后直接弹出浏览器

这样配置后 `npm run server` 他会去找 `webpack.config.js` 文件中的服务器配置信息：

```js
devServer: {
        contentBase: path.resolve(__dirname, 'dist'), //监听的文件夹
        host: 'localhost', //服务器地址，本机ip地址，不建议使用locahost ，防止映射表被修改，出现解析不到的情况。ifconfig 查看 本机ip
        compress: true,//服务器端压缩
        port:1717 //服务器端口，默认80
    }, 
```
> 注意：在翻墙的情况下 `host:` 填写本机ip 会报错，使用 `localhost`

### **打包css文件**

首先需要安装两个包 `style-loader` `css-loader` 来实现对css 文件的转换。

```bash
npm i style-loader css-loader --save-dev
```

- style-loader:处理css中的 url
- css-loader:处理css中的样式

在src文件夹中建立css文件,并且在 **入口文件** 或者 **入口依赖的其他js文件** 中引入：
```js
import css from './css/index.css';
```
webpack.config.js中的配置：

```js
module: {
        rules: [
            {
                test: /\.css$/, //用正则表达式的形式来找到处理的文件
                use: ['style-loader', 'css-loader']//使用哪些loader来处理
            }
        ]
    }, 
```
另一种常用写法可以使每个`loader` 可以配置选项：
```js
 module: {
        rules: [
            {
                test: /\.css$/,
                use: [{
                    loader:"style-loader"
                }, {
                    loader:"css-loader"    
                }]
            }
        ]
    }, 
```
### **压缩js文件**

对js文件做代码压缩，需要使用到 webpack 内置的 `uglifyjs-webpack-plugin` 这个插件。

```js
//webpack.config.js
const UglifyJSPlugin = require('uglifyjs-webpack-plugin');

plugins: [
        new UglifyJSPlugin()
    ],

```

然后 `npm start` 发现打包出来的代码被压缩了。
> 这里遇到了一个坑。。。

接上一步开启服务器 `npm run server` ，命令行报错 ERROR in entry.js from UglifyJs
Unexpected token: name (urlParts) [entry.js:325,4]

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xrrw61dj30eh03l3z5.jpg)

原因在于：没有区分 **开发环境** 和 **生产环境** 。
开发环境中代码是不需要压缩的。如果压缩了，调试没有办法找到对应的行号。代码压缩只需要再 **生产环境** 中进行，在开发环境中压缩代码，跑服务器造成了冲突，具体情况不清楚。
正常的项目开发是不会产生这样的问题的，都会有一个 **开发使用的配置文件** 和 **生产使用的配置文件** 。

### HTML文件的打包

一般项目开发是严格区分 **开发环境** 和 **生产环境** 的，也就是 `src` 文件是我们的 **开发环境** 的项目代码文件夹，`dist` 文件夹是打包后自己生成的，不需要我们自己去创建。把 `index.html` 文件放在 `src` 目录下面，删除文件里面的 `script` 标签，`html-webpack-plugin` 会自动帮我们加入引入js的`script`标签。

```bash
npm i -D html-webpack-plugin
```
项目中的配置

```js
//webpack.config.js
const htmlPlugin = require('html-webpack-plugin')

plugins: [
        new htmlPlugin({
            minify: {
                removeAttributeQuotes: true,//去掉标签中的引号
            },
            hash: true, //引用js时有缓存，加上hash后每次都会给个不同的字符串。
            template:'./src/index.html'
        })
    ], 
```
打包后发现自动的新建了 `dist` 文件夹和里面的打包文件。
> 这里注意配置的写法，大括号太多。我写错了导致打包后的文件异常，找了半天问题 (⊙﹏⊙)b。

### **项目中引入图片**

####  **css中引入图片**
eg:
```css
/* src/css/index.css */
#img{
    background-image:url(../img/img.png);
    width:120px;
    height:101px;
}
```
首先需要安装 `url-loader` 这个插件
配置 `webpack.config.js` 文件：

```js
{
	rules:[
		{
			test: /\.(png|jpg|gif)$/,
            use: [{
                    loader: 'url-loader',   
                    options: {
                        limit:5000,
                    }
            }]
		}
	]
}
```
>注意：loader是不需要引入的。

`limit:5000` 的意思是：图片大于5000字节,自动拷贝图片到 `dist` 文件，并且在 `bundle.js` 文件中（打包后的js文件）修改正确的路径（包含了 `file-loader` 的一些功能）。如果小于5000,会生成base64位格式的图片直接插入到js文件中,好处是减少了http请求。

#### **HTML中引入图片**

webpack 官方是不建议我们在html中引入图片的，如果有这种需求，直接添加 `image` 标签引入图片，打包后发现，图片并没有被打包到 `dist` 文件夹下面，因为图片没有被依赖。
eg:

```html
<-- index.html -->
<div><image src="./img/img.png"/> </div>
```
解决这个问题需要装 `html-withimg-loader`插件，然后配置：

```js
//webpack.config.js
loaders: [
    {
        test: /\.(htm|html)$/i,
        loader: 'html-withimg-loader'
    }
]
```

### **CSS样式分离和publickPath设置**

#### **CSS样式分离**
一般项目中css文件都是直接打包进 `bundle.js` 文件中去的，这样可以减少 `http` 请求。但是在某些时候，我们不想把 css 文件打包进入 `bundle.js` 文件中，比如：一个项目全是靠css样式来布局，js代码非常少得情况下，项目总监要求把项目交给切图仔维护等需求。如何实现呢：

```bash
npm i -D extract-text-webpack-plugin
```
```js
//webpack.config.js
const extractTextPlugin = require('extract-text-webpack-plugin')

plugins:[
	new extractTextPlugin("css/index.css") //把css文件放在服务器的根目录（这里是dist文件夹下），下的css文件夹中。
]

```
css loader 也需要做一些更改

```
{
   test: /\.css$/,
   use: extractTextPlugin.extract({
            fallback: "style-loader",
            use: "css-loader"
   })
}
```
#### **publickPath设置**

打包代码，跑本地服务器，这时候发现图片没有了，查看打包后的项目结构，发现css文件中的路径有问题。

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y7yhbinj30ip03gq3a.jpg)

这时候需要设置 `publiPath` 公用路径，解决静态文件的路径问题。

```js
//webpack.config.js
var website = {
    publicPath:'http://192.168.1.105:1717/' //注意这里的斜杠，ip为服务器ip（在这里是你的计算机ip）
}
output: {
		...
        publicPath:website.publicPath
}
```
重新打包，css文件中图片的路径变为了正确的绝对路径。

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xw9l7l2j30ki03zq3c.jpg)

### **打包和分离LESS**

#### **打包LESS文件到 bundle.js 文件中**

首先需要安装 `less` 、 `less-loader` 这两个包
```bash
npm i -D less less-loader
```
webpack.config.js 配置

```js
// webpack.config.js
module.exports = {
    ...
    module: {
        rules: [{
            test: /\.less$/,
            use: [{
                loader: "style-loader" // creates style nodes from JS strings
            }, {
                loader: "css-loader" // translates CSS into CommonJS
            }, {
                loader: "less-loader" // compiles Less to CSS
            }]
        }]
    }
};
```
> 注意loaderde顺序，顺序错误会造成打包失败的情况。

然后编写LESS文件，引入到入口JS文件中，就可以了。

#### **分离LESS文件**

和分离css文件类似，需要用到 `extract-text-webpack-plugin` 这个包，上面已经安装了，这里不需要安装了。

```js
//webpack.config.js 
const extractTextPlugin = require('extract-text-webpack-plugin')

{
   test: /\.less$/,
   use: extractTextPlugin.extract({
            use: [
                     { loader: 'css-loader' },
                     { loader: 'less-loader' }
            ],
            fallback:'style-loader'
   })
}
plugins: [
	new extractTextPlugin("css/index.css")
]
```

打包后会把LESS中的样式转换成CSS样式，并且打包进index.css 文件中去，这里并不会新建一个CSS文件。

### **SASS的打包和分离**

#### **SASS的打包**

将SASS转换成CSS，并且打包进bundle.js文件中去：

```bash
npm i -D node-sass sass-loader
```

```js
//webpack.config.js
{
	test: /\.scss$/,  //注意这里的 SCSS，不是SASS
	use: [
			{ loader: 'style-loader' },
			{ loader: 'css-loader' },
			{ loader: 'sass-loader' }
		 ]
}
```
在项目中引入打包就可以了。

#### **SASS的分离**

和LESS分离步骤几乎一样

```js
//webpack.config.js 
const extractTextPlugin = require('extract-text-webpack-plugin')

{
   test: /\.scss$/,
   use: extractTextPlugin.extract({
            use: [
                     { loader: 'css-loader' },
                     { loader: 'sass-loader' }
            ],
            fallback:'style-loader'
   })
}
plugins: [
	new extractTextPlugin("css/index.css")
]
```

### 自动添加CSS属性前缀

```
npm i -D postcss-loader autoprefixer
```
需要再项目的根目录新建 `postcss.config.js` 配置文件。

```js
//postcss.config.js
module.exports = {
    plugins: [
        require('autoprefixer')
    ]
}
```

```js
//webpack.config.js
{
test: /\.css$/,
use: extractTextPlugin.extract({
		fallback: "style-loader",
		use: [
				{ loader: 'css-loader' },
				{ loader: 'postcss-loader' } //here
             ]
})
} 
```
更多的配置参考 [postcss-loader](https://github.com/postcss/postcss-loader)

### 分离多余的CSS样式

有些时候css样式有多余的情况，比如使用 Bootstrap 这个库文件一小部分样式，或者项目几次改版产生了很多无效的CSS样式，这种情况下需要去除掉多余的CSS样式，以节省带宽。
安装webpack插件 :
```bash
npm i -D purifycss-webpack purify-css
```
配置选项：

```js
//webpack.config.js
const glob = require('glob');
const purifyCssPlugin = require('purifycss-webpack')
const extractTextPlugin = require('extract-text-webpack-plugin')
plugin:[
	...,
	new purifyCssPlugin({
            paths:glob.sync(path.join(__dirname,'src/*.html'))
        })
]
```
这样就实现了多余代码的去除，好像是需要结合 CSS分离（`extract-text-webpack-plugin`）技术，才能实现代码的去除。自己测试过程：把CSS分离去除后，也就是让CSS代码打包进 `bundle.js` 文件里面，配置好代码，打包后没有去除掉多余的CSS文件，在 `bundle.js` 文件里面还能找到样式。

### **使用Babel转换ES6和ES7语法**
首先需要安装插件，我是结合 React 项目来使用 ES6 ES7 的，所以需要安装 `babel-preset-react` 来解析 React 的 `jsx` 语法。
```
npm i -D babel-core babel-loader babel-preset-env babel-preset-react
```

```js
//webpack.config.js
{
	test: /\.(js|jsx)$/,
	use: [
			{ 
				loader: 'babel-loader' ,
				//options:{    这里注意实际中不会这样配置，一般新建.babelrc文件来写这些配置项    
				//	presets:["env","react"]
				//}
			}
         ],
    exclude: /node_modules/  //不需要转换node_modules下的js文件
}
```

实际开发中 babel 的配置代码会越来越多，不建议在 `use` 中写 babel 的配置选项，而是在项目的根目录新建 `.babelrc` 配置文件。

```js
//.babelrc
{
    "presets": ["react","env"]  //渲染器
}
```

### **打包后的代码调试**

打包有四种模式：

 - `source-map` : 打包速度最慢，最详细，生成了一个 `.map`的独立的文件,放在 `dist` 打包目录下，可以与打包后的文件很好的结合，报错信息包括 **行** 和 **列**。
 - `cheap-module-source-map` : 也生成独立文件，报错信息包括 **行** 和 **不包括列**，比上面的模式快。
 
 - `eval-source-map` : 不生成独立文件,报错信息包括 **行** 和 **列**，直接在 `bundle.js` 文件中生成 `map`, 速度也很快，有安全和性能的隐患，只能在开发阶段使用，上线前一定要删除 `devtool:'eval-source-map'`。
 - `cheap-module-eval-source-map` :不生成独立文件, 报错信息包括 **行** 和 **不包括列列**，
 
```js
//webpack.config.js
module.exports = {
	devtool:'source-map',   //here
    entry: {},
    output: {},
    module: {},
    plugins: [], 
    devServer: {}
} 
```

- [参考教程](https://www.chungold.com/course/32)

