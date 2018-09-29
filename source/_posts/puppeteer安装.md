---
title: puppeteer安装
date: 2018-09-24 23:48:03
categories: node
tags:
  - puppeteer
---

puppeteer 在执行安装的过程中需要执行install.js，这里会下载Chromium，翻墙也下载失败，导致安装不成功，官网建议是进行跳过，解决办法：

```bash
> npm i --save puppeteer --ignore-scripts
#忽略 puppeteer js脚本的执行
```

### 手动下载 Chromium

[Chromium 下载](https://download-chromium.appspot.com/)

地址会根据系统来下载对应的 安装文件。

mac环境下载完毕后，把Chromium复制在项目的根目录

测试是否安装成功
```js
const puppeteer = require('puppeteer')

;(async () => {
  const browser = await puppeteer.launch({
    executablePath:   './chromium/Chromium.app/Contents/MacOS/Chromium',
    //自定义程序地址
    
  });
  const page = await browser.newPage();
  await page.goto('https://y.qq.com');
  await page.screenshot({path:'yqq.png'});
  browser.close();
})()
```
puppeteer.launch 参数说明

- executablePath：运行Chromium或Chrome可执行文件的路径
- headless：true为不打开浏览器执行，浏览器运行在内存中，默认为true
- timeout： 等待浏览器实例启动的最长时间（以毫秒为单位）。默认为30000（30秒）。通过0禁用超时
- args： 传递给浏览器实例的其他参数