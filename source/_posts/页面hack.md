---
title: 页面hack
date: 2018-10-10 16:08:29
categories: HTML
tags:
  - html
---

### 弹窗后禁止页面滚动

首先考虑的是

```css
body {
  overflow-x: hidden;
  overflow-y: hidden;
}
```
但是这种方法，在关闭弹窗时页面出现滚动条会导致页面偏移，影响用户体验。

对于偏移的问题可以设置body的padding-right:15px，来解决。但是在不同的浏览器中的滚动条宽度不一，还是会导致几个像素的偏移。

完美解决办法：
动态的计算滚动条的宽度，然后设置padding值

```css
.fancybox-lock-test {
  overflow-y: hidden !important;
}
```

```js
var w1 = $(window).width()
Dom.htmlDom.addClass('fancybox-lock-test')
var w2 = $(window).width()
Dom.htmlDom.removeClass('fancybox-lock-test')
$("<style type='text/css'>.fancybox-margin{margin-right:" + (w2 - w1) + "px;}</style>").appendTo("head")
Dom.bodyDom.addClass('fancybox-margin')
$('body').css({
  "overflow-x": "hidden",
  "overflow-y": "hidden",
});
```

### 图片懒加载

解决的问题：对于大图首次加载产生白屏的问题。

解决的思路：把大图压缩到几十KB，也就是当做缩略图来展示，首次加载缩略图，加载完毕之后再加载真实的高清图，减少白屏时间，提升用户体验。

```html
<img class="introduce-1-img thumbnails" src="https://i.toydb.cn/1-introduce_1_min.jpg" data-img="https://i.toydb.cn/1-introduce_1.png"
        alt="">
```

```css
.thumbnails {
  filter: blur(4px);  /*图片产生模糊效果*/
  transition: all 0.7s;
}
```

```js
var thumbImg = $('.thumbnails') //所有需要懒加载的图片
for (var i = 0; i < thumbImg.length; i++) {  
  (function (i) {
    var imgObject = new Image();
    var currentImg = thumbImg.eq(i);
    var img = currentImg.data('img');
    imgObject.src = img
    imgObject.onload = function () {
      currentImg.attr("src", img)
      currentImg.css("filter", "blur(0)")
    }
  })(i)
}
```

### 移动端rem布局

对于页面切图不要使用750pxPSD设计图，在dpr=3的高清屏下，图片可能存在不清晰的情况,切完图片后，在把PSD图片调回 750px宽度，加入下面JS代码，计算值为 `rem=设计图px/100`

```js
(function(win, lib) {
      var doc = win.document;
      var docEl = doc.documentElement;
      var metaEl = doc.querySelector('meta[name="viewport"]');
      var flexibleEl = doc.querySelector('meta[name="flexible"]');
      var dpr = 0;
      var scale = 0;
      var tid;
      var flexible = lib.flexible || (lib.flexible = {});

      if (metaEl) {
          var match = metaEl.getAttribute('content').match(/initial\-scale=([\d\.]+)/);
          if (match) {
              scale = parseFloat(match[1]);
              dpr = parseInt(1 / scale);
          }
      } else if (flexibleEl) {
          var content = flexibleEl.getAttribute('content');
          if (content) {
              var initialDpr = content.match(/initial\-dpr=([\d\.]+)/);
              var maximumDpr = content.match(/maximum\-dpr=([\d\.]+)/);
              if (initialDpr) {
                  dpr = parseFloat(initialDpr[1]);
                  scale = parseFloat((1 / dpr).toFixed(2));
              }
              if (maximumDpr) {
                  dpr = parseFloat(maximumDpr[1]);
                  scale = parseFloat((1 / dpr).toFixed(2));
              }
          }
      }

      if (!dpr && !scale) {
          var isAndroid = win.navigator.appVersion.match(/android/gi);
          var isIPhone = win.navigator.appVersion.match(/iphone/gi);
          var devicePixelRatio = win.devicePixelRatio;
          if (isIPhone) {
              // iOS下，对于2和3的屏，用2倍的方案，其余的用1倍方案
              if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {
                  dpr = 3;
              } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)) {
                  dpr = 2;
              } else {
                  dpr = 1;
              }
          } else {
              // 其他设备下，仍旧使用1倍的方案
              dpr = 1;
          }
          scale = 1 / dpr;
      }

      docEl.setAttribute('data-dpr', dpr);
      if (!metaEl) {
          metaEl = doc.createElement('meta');
          metaEl.setAttribute('name', 'viewport');
          metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
          if (docEl.firstElementChild) {
              docEl.firstElementChild.appendChild(metaEl);
          } else {
              var wrap = doc.createElement('div');
              wrap.appendChild(metaEl);
              doc.write(wrap.innerHTML);
          }
      }

      function refreshRem() {
          var width = docEl.getBoundingClientRect().width;
          // 适配平板
          if (width / dpr > 750) {
              width = 750 * dpr;
          }
          var rem = 100 * (width / 750);
          docEl.style.fontSize = rem + 'px';
          flexible.rem = win.rem = rem;
      }

      win.addEventListener('resize', function() {
          clearTimeout(tid);
          tid = setTimeout(refreshRem, 300);
      }, false);
      win.addEventListener('pageshow', function(e) {
          if (e.persisted) {
              clearTimeout(tid);
              tid = setTimeout(refreshRem, 300);
          }
      }, false);

      if (doc.readyState === 'complete') {
          doc.body.style.fontSize = 12 * dpr + 'px';
      } else {
          doc.addEventListener('DOMContentLoaded', function(e) {
              doc.body.style.fontSize = 12 * dpr + 'px';
          }, false);
      }
      refreshRem();
      flexible.dpr = win.dpr = dpr;
      flexible.refreshRem = refreshRem;
      flexible.rem2px = function(d) {
          var val = parseFloat(d) * this.rem;
          if (typeof d === 'string' && d.match(/rem$/)) {
              val += 'px';
          }
          return val;
      }
      flexible.px2rem = function(d) {
          var val = parseFloat(d) / this.rem;
          if (typeof d === 'string' && d.match(/px$/)) {
              val += 'rem';
          }
          return val;
      }

  })(window, window['lib'] || (window['lib'] = {}));
```


