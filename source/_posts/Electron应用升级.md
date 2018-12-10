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

![image](https://ws4.sinaimg.cn/large/0073tXM5gy1fy1y9cnh7ij30bo070t8u.jpg)


