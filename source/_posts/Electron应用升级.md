---
title: Electron应用升级
date: 2018-12-05 21:39:50
categories: pc
tags:
  - electron
---

### 前言

最近公司的一个vue项目，为了安全方面的考虑没有上传到服务器，每次打包需要群发，很是不方便，后来又考虑到使用 Electron打包成客户端，实现自动升级，在此做一些记录

### 桌面应用app图标

图标分为三种，三个平台分别是：
- mac 的 .icns 后缀
- window 的 .ico 后缀 (256x256px)
- linux 的 .png 后缀 (256x256px)

.ico百度搜索有很多生成方式。

.icns生成:

在桌面新建 `icons.iconset` 文件夹,把原图片放在桌面。
打开命令行cd到桌面，执行一下命令，生成不同尺寸的图片到 `icons.iconset`中

```bash
sips -z 16 16     pic.png --out icons.iconset/icon_16x16.png
sips -z 32 32     pic.png --out icons.iconset/icon_16x16@2x.png
sips -z 32 32     pic.png --out icons.iconset/icon_32x32.png
sips -z 64 64     pic.png --out icons.iconset/icon_32x32@2x.png
sips -z 64 64     pic.png --out icons.iconset/icon_64x64.png
sips -z 128 128   pic.png --out icons.iconset/icon_64x64@2x.png
sips -z 128 128   pic.png --out icons.iconset/icon_128x128.png
sips -z 256 256   pic.png --out icons.iconset/icon_128x128@2x.png
sips -z 256 256   pic.png --out icons.iconset/icon_256x256.png 
sips -z 512 512   pic.png --out icons.iconset/icon_256x256@2x.png 
sips -z 512 512   pic.png --out icons.iconset/icon_512x512.png
sips -z 1024 1024   pic.png --out icons.iconset/icon_512x512@2x.png
```
pic.png替换为你的图片文件路径，尺寸要求为1024x1024

然后生成 icns图标

```bash
iconutil -c icns icons.iconset -o Icon.icns
```

### 目录结构

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y9cnh7ij30bo070t8u.jpg)

`static` 文件夹和 `index.html` 为vue打包好的文件

`main.js` 为程序主进程文件

```js
// main.js
const path = require('path');
const url = require('url');
const {
    app,
    BrowserWindow,
    ipcMain,
    dialog,
    shell
} = require('electron');
const { autoUpdater } = require('electron-updater');
const feedUrl = `http://39.107.118.115:3000/${process.platform}`;
//这里39.107.118.115:3000 为服务端ip地址，需要再服务器上搭建一个静态服务器
let isSetupFlash = false
let webContents;
try {
    app.commandLine.appendSwitch('ppapi-flash-path', app.getPath('pepperFlashSystemPlugin'))
    isSetupFlash = true
} catch (error) {
    isSetupFlash = false
}
let createWindow = () => {
    let win = new BrowserWindow({
        maximizable: true,
        width: 1200,
        height: 800,
        minWidth: 1200,
        minHeight: 800,
        show: false,
        center: true,
        webPreferences: {
            plugins: true
        },
        frame: true,
    });

    webContents = win.webContents;

    win.loadURL(
        url.format({
            pathname: path.join(__dirname, 'index.html'),  //指定渲染进程入口文件
            protocol: 'file:',
            slashes: true
        })
    );
    win.on('closed', function() {
        win = null
    })
    win.once('ready-to-show', () => {
        win.show()
    })
};
let updateDescription
let checkForUpdates = () => { //检测更新
    autoUpdater.setFeedURL(feedUrl);

    //更新错误
    autoUpdater.on('error', function(message) {
        dialog.showErrorBox('更新错误！', '更新出错，请稍后再试')
    });
    //检测更新
    autoUpdater.on('checking-for-update', function(message) {

    });
    //可以更新
    autoUpdater.on('update-available', function(message) {
        updateDescription = message.description //打包后 dist/latest.yml 中增加description字段，可以填写更新的描述信息，更加友好。
    });
    //不需更新
    autoUpdater.on('update-not-available', function(message) {

    });

    // 更新下载进度事件
    autoUpdater.on('download-progress', function(progressObj) {

        })
        //下载升级
    autoUpdater.on('update-downloaded', function(event, releaseNotes, releaseName, releaseDate, updateUrl, quitAndUpdate) {

        dialog.showMessageBox({
            type: 'info',
            title: '提示消息',
            message: `
                新版本已下载完成，是否现在退出并安装?
                更新内容：
                ${updateDescription}
            `,
            buttons: ['yes', 'no']
        }, function(index) {
            if (index === 0) {
                autoUpdater.quitAndInstall();
            }
        })
    });

    //执行自动更新检查
    autoUpdater.checkForUpdates();
};
app.on('ready', () => {
    createWindow();
    setTimeout(function() {
        if (!isSetupFlash) {
            dialog.showMessageBox({
                type: 'info',
                title: '提示消息',
                message: `
                    检测到您系统中没有安装Flash，编辑器中的视频可能无法查看，是否去安装？
                `,
                buttons: ['yes', 'no']
            }, function(index) {
                if (index === 0) {
                    shell.openExternal('https://get2.adobe.com/cn/flashplayer/')
                    app.quit()
                }
            })
        }
        checkForUpdates()
    }, 1000);
});

app.on('window-all-closed', () => {
    if (process.platform !== 'darwin') {
        app.quit()
    }
});
app.on('activate', function() {
    if (mainWindow === null) {
        createWindow()
    }
})
```


```json
{
    "name": "electrontoydb",
    "version": "1.0.5",
    "description": "...", //程序的描述，需要填写。
    "main": "main.js", //指定主进程入口文件
    "scripts": {
        "dev": "node_modules/.bin/electron .",
        "build": "rimraf dist && electron-builder -w"
    },
    "keywords": [],
    "author": "",
    "license": "ISC",
    "devDependencies": {
        "electron": "^3.0.7",
        "electron-builder": "^20.29.0",
        "rimraf": "^2.6.2"
    },
    "dependencies": {
        "electron-updater": "^3.1.6"
    },
    "build": {
        "productName": "..",  //打包软件名称
        "appId": "com.toydb.app",
        "win": {
            "icon": "icon/icon.ico" //windows软件图标路径
        },
        "mac": {
            "icon": "icon/icon.icns" // mac图标路径
        },
        "publish": [{
            "provider": "generic",  
            "url": "http://39.107.118.115:3000" //指定更新的地址
        }]
    }
}
```

在打包后，上传dist 中的四个文件到服务器 win32文件夹下
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fykhfclqzcj30g4044gm3.jpg)

苹果应用也是一样。

> 需要注意的是：每次打包升级需要修改 package.json 文件中的 version 版本号，例如：1.0.0 --> 1.0.1。