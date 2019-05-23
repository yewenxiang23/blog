---
title: react-native笔记本
date: 2018-09-25 00:04:29
categories: react-native
tags:
  - react-native
---

##### android App跳转到设置页面如何原生模块实现?

-  [Stack Overflow](https://stackoverflow.com/questions/41677735/react-native-unable-to-open-device-settings-from-my-android-app) android 设置 [常量列表](https://developer.android.com/reference/android/provider/Settings.html)

##### android App，设置禁止横屏。

-  AndroidManifest.xml 文件中的 `<activity>`  标签内添加 `android:screenOrientation="portrait"`

#####  ios android 设置App 名称
-  ios : 在 `Info.plist`中修改
- ![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y5022qmj30f502wgm2.jpg)
-  Android : `android\app\src\main\res\values\strings.xml` 文件中修改 `<string name="app_name">MyProject</string>`

##### ios android 设置icon图标

- ios: 首先[在这里](http://makeappicon.com/) 上传你的图标，注意：四个圆角边要透明的，如果是白色的在安卓上可能显示出来。
- 然后直接 `project/ios/project_name/images.xcassets/` 直接替换。
- android: 直接替换掉 `project/android/app/src/main/res` 里面的文件夹。

##### ios android 设置启动图

- ios: 使用图片工具 `App Icon Gear` 来生成不同尺寸的图片
- 之后 [参考](http://www.jianshu.com/p/735ba76594b5)

注意：
![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1xtussjij30es03swes.jpg)
- android: 使用 `react-native-splash-screen` 来实现。
- 可以使用 [这里](http://ticons.fokkezb.nl/) 来生成android 的图片，注意生成后,每个文件夹里面的每张图名称需要更改为 `launch_screen` 然后配置 `react-native-splash-screen` ，就可以使用的了。
- 碰到的坑：按照文档配好后，出现ios 正常 ，android启动闪退, `“Unfortunately, app has stopped”`, [解决办法](https://github.com/crazycodeboy/react-native-splash-screen/issues/124)

#####  解决TextInput 框点击空白处不失去焦点问题

- 在 `TextInput` 最外层的根节点加一个 `ScrollView` ,添加 keyboardShouldPersistTaps={'never'}。
- 或者在最外层添加 `TouchableOpacity`

```js
import dismissKeyboard from 'dismissKeyboard'
dismissKeyboardClick = () => {
        dismissKeyboard()
    }
render(){
	return （
	...
	<TouchableOpacity
                    style={{ flex: 1 }}
                    onPress={this.dismissKeyboardClick}
                    activeOpacity={1}
                >
                //TextInput组件
    <TouchableOpacity/>
    ...
）	
}
```

##### react-native webview 如何引入本地html文件

- ios : 直接 `source={require('../../assets/html/message.html')}`
- android : Android 需要先把静态资源放到 `android/app/src/main/assets` 目录下面，然后把 `require('../../assets/html/message.html')` 换成 ` {uri: 'file:///android_asset/html/message.html'}`。

##### 解决 android 绝对定位元素定位在底部被键盘顶起问题

![image](http://ywx.store:86/kodexplorer/data/User/admin/home/图片/0073tXM5gy1fy1y5k91cdj30h305ht9m.jpg)

如果希望被顶起：`android:windowSoftInputMode="adjustResize"`

##### TextInput 组件设置 value  值不显示的问题

value 值类型为字符串，设置成Number 类型不显示