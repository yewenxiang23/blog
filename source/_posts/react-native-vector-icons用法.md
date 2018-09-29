---
title: react-native-vector-icons用法
date: 2018-09-29 11:51:02
categories: react-native
tags:
  - react-native
---

### 介绍

react-native-vector-icons 是一个字体图标的库。

- [react-native-vector-icons](https://oblador.github.io/react-native-vector-icons/)

### 使用

#### 首先进入到项目目录

- npm install react-native-vector-icons --save
- npm install rnpm -g
- rnpm link
- 然后在项目目录 打开demo/ios/demo.xcodeproj 文件，弹出xcode，右键工程文件`Add Files to ..`
- 选择 deno_modules/react-native-ver-icons/Fonts文件夹。

在xcode的Info.plist文件中,加入: `Fonts provided by application`数组,并根据需要加入以下9项:

- [Entypo](http://entypo.com/)
- [EvilIcons](http://evil-icons.io/)
- [FontAwesome](http://fontawesome.io/icons/)
- [Foundation](http://zurb.com/playground/foundation-icon-fonts-3)
- [Ionicons](http://ionicframework.com/docs/ionicons/)
- [MaterialIcons](https://www.google.com/design/icons/)
- [MaterialCommunityIcons](https://materialdesignicons.com/)
- [Octicons](http://octicons.github.com)
- [Zocial](http://zocial.smcllns.com/)
- [SimpleLineIcons](http://simplelineicons.com/)
