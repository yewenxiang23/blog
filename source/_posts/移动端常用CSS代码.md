---
title: 移动端常用CSS代码
date: 2018-09-24 23:52:20
tags:
  - 移动端常用CSS代码
  - CSS
---

### 1px边框问题使用

```css
.xx{
	position:relative;
}
.xx:before{
	content:'';
	position: absolute;
	top: 0;
	left: 0;
	border: 1px solid #ccc;
	width: 200%;
	height: 200%;
	box-sizing:border-box;
	-webkit-box-sizing:border-box;
	-webkit-transform: scale(0.5);
	transform: scale(0.5);
	-webkit-transform-origin: left top;
	transform-origin: left top;
}
```

### 背景图裁切

```css
.xx{
	background-image:url('...');
	background-position:center center;
	background-size:cover;
	background-repeat:no-repeat;
}
```

### 横向滚动

```css
.parent{
	display: -webkit-box;
    overflow-x: scroll;
    overflow-y: hidden;
    -webkit-overflow-scrolling: touch; 
    /* 弹性滑动 */
}
```

### 单行显示，超过省略号

```css
.xx{
	white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
}
```

### 两行显示，超过省略号

```css
.xx{
	overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient: vertical;
}
```
