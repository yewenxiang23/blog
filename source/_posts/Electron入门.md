---
title: Electron入门
date: 2018-10-31 09:11:44
categories: pc
tags:
  - electron
---

### 创建electron 项目

#### 使用 electron-forge 创建项目

```bash
npm install -g electron-forge
electron-forge init my-app
```

#### 手动创建

package.json 文件中的main字段，必须是electron主进程文件，也就是main.js(或者index.js)

```js
//main.js  主进程文件
var electron = require('electron')

var app = electron.app //创建elecgtron引用

var BrowserWindow = electron.BrowserWindow //创建BrowsetWindow引用

var mainWindow = null  //变量来保存对应窗口的引用
app.on('ready', function(){
  //创建 BrowserWindow 的实例赋值给win 打开窗口
  mainWindow = new BrowserWindow({
    width:400,
    height: 400,
  })
  mainWindow.loadFile('index.html') //把index.html文件加载到主窗口
  mainWindow.webContents.openDevTools() //开启调试模式
  mainWindow.on('close', () => {
    mainWindow = null //关闭窗口时，回收变量。
  })
})
// 当全部窗口关闭时退出。
app.on('window-all-closed', () => {
  // 在 macOS 上，除非用户用 Cmd + Q 确定地退出，
  // 否则绝大部分应用及其菜单栏会保持激活。
  if (process.platform !== 'darwin') {
    app.quit()
  }
})

app.on('activate', () => {
  // 在macOS上，当单击dock图标并且没有其他窗口打开时，
  // 通常在应用程序中重新创建一个窗口。
  if (win === null) {
    createWindow()
  }
})
```

目录结构:

```bash
.
├── index.html
├── main.js
└── package.json
```

官方推荐在项目中安装 `electron`，而不是在全局。

```bash
npm i electron -D
```

运行

```json
//package.json
"scripts": {
  "start": "./node_modules/.bin/electron ."
},
```

### Electron 运行流程

首先开启主进程去找当前目录下的 `package.json` 中的main字段的入口文件 `index.html` 找到后载入并开启渲染进程。

渲染进程和主进程的介绍：

```bash
.
├── index.html
├── main    #主进程js文件夹
├── main.js
├── node_modules
├── package.json
└── renderer  #渲染进程js文件夹
```
主进程：package.json文件中 main 字段指向的文件，如果指向main.js，那main.js和它所引入的js文件，全部为主进程代码。
渲染进程：在main.js 文件中 `mainWindow.loadFile('index.html')` 所引入的html文件中所引入的js文件为渲染进程文件。


### 操作本地文件demo

在electron项目中，主进程和渲染进程都可以调用Node模块，（主进程为main.js文件中执行的代码, 渲染进程为 index.html中视图文件），
一般新建一个renderer文件夹写在渲染进程中

```bash
├── index.html
├── main.js
├── package.json
└── renderer
    └── index.js
```

```html
<!--index.html-->
<body>
  <button id="btn">获取package.json</button>
  <textarea id="textarea" cols="40" rows="20"></textarea>
  <script src="./renderer/index.js"></script>
</body>
```

```js
//renderer/index.js
var fs = require('fs')
window.onload = function(){
  var btn = window.document.querySelector('#btn')
  var textarea = window.document.querySelector('#textarea')
  btn.onclick = function(){
    fs.readFile('package.json', (err, data) => {
      textarea.innerHTML = data
    })
  }
}
```

### 实现拖拽打开文件demo 

```html
<body>
  <div class="content" id="content"></div>
  <script src="./renderer/index.js"></script>
</body>
```

```js
//renderer/index.js
var fs = require('fs')
var content = document.querySelector('#content')
content.ondragenter = content.ondragover = content.ondragleave = function(){
  return false
}
content.ondrop = function(e){
  e.preventDefault()
  console.log(e.dataTransfer.files[0])
  var path = e.dataTransfer.files[0].path
  fs.readFile(path, 'utf-8', (err, data) => {
    content.innerHTML = data
  })
}
```

### 主进程和渲染进程通信

主要使用了两个模块 ipcRenderer 和 ipcMain模块。

#### 异步通信

```js
//渲染进程
var { ipcRenderer } = require('electron')
var sendDom = document.querySelector('#btn2')
sendDom.onclick = function(){
  ipcRenderer.send('sendM', 123) //渲染进程给主进程广播事件
}
ipcRenderer.on('replay', (event, value) => {  //接收主进程的回复
  console.log(value)
})
```

```js
//主进程
var { ipcMain } = require('electron')

ipcMain.on('sendM', (event,value) => {
  console.log(event)
  console.log(value)    //123
  event.sender.send('replay', 'ok')  //给渲染进程回复消息
})
```

#### 同步通信

```js
//渲染进程
var { ipcRenderer } = require('electron')
var sendDom = document.querySelector('#btn2')
sendDom.onclick = function(){
  const msg = ipcRenderer.sendSync('asyncSendM', 123)
  console.log(msg)  //同步通信
}
```

```js
//主进程
var { ipcMain } = require('electron')
ipcMain.on('asyncSendM', (event, value) => {
  event.returnValue = '同步通信'
})
```

### 渲染进程和渲染进程之间的通信

场景：多个新窗口之间的通信

解决方法:

- 使用HTML5 localStorage 来实现多个窗口的传值通信
- 通过 BrowserWindow 和 webContents 模块来实现

通过 BrowserWindow 和 webContents 模块来实现，通俗的来讲，是通过 `ipcRenderer.send` 把值传给主进程，然后主进程通过 win.webContents.send(params),传递给 `win` 这个窗口。

```js
//主进程
win = new BrowserWindow({
  width:300,
  height:300,
})
win.loadFile('news.html')
win.webContents.on('did-finish-load', function(){ //窗口加载完毕后传值
  win.webContents.send('toNews',params)
})
```

```js
//渲染进程
var { ipcRenderer } = require('electron)
ipcRenderer.on('toNews', function(event, params){
  console.log(params) 
})
```

消息如何回传？

比如 渲染线程A 和渲染线程B，下面简称A、B
当A向B传递参数，如果需要回传参数，那么，在传递参数的时候带上当前渲染线程（A）的ID
```js
//主线程
const winId = BrowserWindow.getFocusedWindow().id //获取渲染线程A的ID
```
在渲染线程B中使用 `BrowserWindow.fromId(winId)` 来获取渲染线程A的窗口实例
```js
var BrowserWindow = require('electron').remote.BrowserWindow
//由于 BrowserWindow 是主线程中的模块 所以在渲染线程中使用的时候，需要借助于 remote 模块来调用
let win = BrowserWindow.fromId(winId)
```
后面的步骤和上面类似了

```js
//渲染线程B
win.webContents.send('toA',params)  //传递消息
//渲染线程A
ipcRenderer.on('toA', function(event, params){  //接收消息
  console.log(params) 
})
```

### Electron 模块

- 主进程中使用的模块
- 渲染进程中使用的模块
- 都可以使用的模块
![](http://ozrm3516s.bkt.clouddn.com/c85c0255ff8a54a6a6bc4659f6f06963.jpg)

模块可以让我们用js调用或者操作电脑上某一部分原生的功能。

#### remote模块

渲染进程和主进程中进行进程通讯(IPC)的通讯模块，可以让我们在渲染进程中调用主进程的方法。

##### 打开新窗口demo

```js
var btn = document.querySelector('#btn')
var BrowserWindow = require('electron').remote.BrowserWindow

var win=null
btn.onclick = function(){
  win = new BrowserWindow({
    width:400,
    height:300,
    // frame:false,   //去掉顶部关闭条
    // fullscreen: true, //全屏
  })
  win.loadFile('news.html')
  win.on('closed',() => {
    win = null
  })
}
```

#### menu模块

可以自定义软件菜单，注意，如果在渲染模块中定义会出现软件打开时，能看到初始菜单的情况，所以，最好在主线程中去定义菜单选项

```js
//定义菜单模块
const {Menu} = require('electron')
const template = [
  {
    label: '文件',
    submenu: [
      {
        label: '新建文件',
        accelerator: 'command+n' //自定义快捷键
      },
      {
        label: '新建窗口'
      }
    ]
  },
  {
    label: '编辑',
    submenu: [
      {
        label: '复制',
        role: 'copy',  // 内置快捷键
        click: function(){  //自定义事件
          console.log('111')
        }
      },
      {
        label: '新建窗口',
        click: function(){
          console.log('111')
        }
      }
    ]
  }
]
const menu = Menu.buildFromTemplate(template)
Menu.setApplicationMenu(menu)
```

别忘了在主线程中的 `main.js` 中引入文件 `require('./main/Menu.js')`!


#### 右键菜单

```html
<!-- index.html -->
  <script>
  const {remote} = require('electron')
  const {Menu, MenuItem} = remote
  
  const menu = new Menu()
  menu.append(new MenuItem({label: 'MenuItem1', click() { console.log('item 1 clicked') }}))
  menu.append(new MenuItem({type: 'separator'}))
  menu.append(new MenuItem({label: 'MenuItem2', type: 'checkbox', checked: true}))
  window.addEventListener('contextmenu', (e) => {
    e.preventDefault()
    menu.popup({window: remote.getCurrentWindow()})
  }, false)
  </script>
```

