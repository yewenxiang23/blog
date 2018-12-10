---
title: ajax跨域解决思路
date: 2018-09-25 00:01:29
categories: js
tags:
  - ajax
  - 跨域
---

###  为什么会发生AJAX跨域？

#### 浏览器限制
当浏览器发现请求是跨域的时候，它会做一些校验，如果校验不通过它就会报跨域安全错误。

#### 跨域
发出去的请求，协议、端口、请求地址，任何一个不一样，浏览器就会认为是跨域。

#### 发送的是 XHR(XMLHttpRequest) 请求
如果发送的不是XHR请求，就算是跨域，浏览器也不会报错。
![image](https://wx1.sinaimg.cn/large/0073tXM5gy1fy1y79efxoj30of0533zd.jpg)

### 解决问题的思路

1.浏览器限制，我们就解除浏览器的跨域限制，从命令行中启动chrome。

`open -n /Applications/Google\ Chrome.app/ --args --disable-web-security  --user-data-dir=/Users/wenxiangye/MyChromeDevUserData/`

2.JSONP如何解决跨域问题？
利用JQ实现JSONP请求

```js
$.ajax({
	url: base+"/get1",
	dataType: "jsonp",
	jsonp: "callback2",   //默认不写前后端约定的名称是 callback
	cache: true,  //表示结果可以被缓存，jsonp请求url中不会带 `_`字段
	success: function(json){
		result = json;
	}
})
```
![image](https://ws2.sinaimg.cn/large/0073tXM5gy1fy1y72cqo9j30o605w75f.jpg)
> 有三个不同点:
> 1. 请求的Type不同
> 2. 请求返回的 Content-Type 不同
> 3. JSON请求的URL后面自动加了callback  

动态创建`<script>`标签，请求的资源可以跨域，来解决跨域问题的（eg：`<img>`标签中请求图片的地址可以跨域）
后台代码也需要做处理，因为后台代码返回的是JSON，而通过JSONP请求的是JS文件，浏览器会把后台返回的JSON字符串当成是JS来解析，所以报错了。
后台代码需要做的调整就是：请求参数中发现callback 这个字段，需要返回JS代码，callback 后面的值作为函数名，而请求需要返回的参数，作为函数的参数。

动态创建的script请求完毕后会被销毁，所以dom结构中无法查看，需要再JQ源码中9816行出打断点查看。
![image](https://ws3.sinaimg.cn/large/0073tXM5gy1fy1xvns480j30jz08swfo.jpg)
> jsonp请求里面除了callback参数之外还多了一个`_`参数，参数值是一个随机的数字，防止请求被缓存。

JSON弊端：

- 服务器需要做一些改动。
- 只支持GET
- 发送的不是XHR请求，（XHR有很多新的特性，比如异步。）


3.被调用方解决跨域问题
响应头增加字段，告诉浏览器允许跨域。
浏览器发现请求是跨域的时候，他会在请求头增加当前域的字段`Origin: http://localhost:8081` 等请求返回来，他会检查响应头里面的字段信息，是否允许跨域。
后端需要在响应头增加 ：
`Access-Control-Allow-Origin:http://localhost:8081`   
`Access-Control-Allow-Methods:GET`
两个字段都可以填写`*`表示所有域名和请求都可以跨域

### 简单请求和非简单请求
浏览器在发送跨域请求的时候，会先判断跨域是简单请求还是非简单请求，如果是简单请求它会先执行，后判断。如果是非简单请求，会发一个OPTIONS的预检命令，检查通过后，他会把真正的请求发送过去。

- 简单请求的定义：方法为`GET` `POST` `HEAD` 、无自定义头部、`Content-Type`为：`text/plain`、`multipart/form-data`、`application/x-www-form-urlencoded` 三种。
- 非简单请求：方法为 `PUT`, `DELETE` 方法的ajax请求、发送json格式的ajax请求、带自定义头的请求

非简单请求时，如果跨域了，浏览器首先发送一个OPTIONS预检请求，在预检请求中会有一个字段  `Access-Control-Request-Method: content-type` ，询问后台服务器是否允许 `content-type` 这个头，如果响应头部没有通过的信息，就会报跨域的错误。

**解决方法**： 后台代码需要增加头信息 `Access-Control-Allow-Headers: Content-Type`

> 非简单请求每个跨域请求都会请求两次，这样非常影响效率，响应可以增加一个头信息，来缓存预检命令，`Access-Control-Max-Age: 3600` 这个头信息的意思是告诉浏览器，在3600秒内，可以缓存预检命令的结果，不需要发送预检命令。

**带cookie的跨域**：当响应头`Access-Control-Allow-Origin: *` 时，是不能满足带cookie的跨域请求的，前端设置 ：

```js
$.aiax({
	...,
	xhrFields:{
		withCredentials:true
	}
    ...
})
```

后端设置头信息 `Axxess-Control-Allow-Origin: http://localhost:8081`和`Access-Control-Allow-Credentials: true`

但是这样做之后，浏览器只会允许 `http://localhost:8081` 这个地址来跨域，如果需要多个地址请求实现跨域如何实现呢？
之前知识点： **浏览器发现请求是跨域的时候，他会在请求头增加当前域的字段Origin: http://localhost:8081 等请求返回来，他会检查响应头里面的字段信息，是否允许跨域。**
后端可以动态的去取 `Origin`这个字段的值，如果不为空，则 `Axxess-Control-Allow-Origin: Origin`,现在就可以支持任何的跨域调用了。

### 带自定头的跨域

首先前端定义：

![image](https://ws2.sinaimg.cn/large/0073tXM5gy1fy1xwidx7wj30dh083dgg.jpg)

可以发现请求头部增加了如下内容：
![image](https://ws1.sinaimg.cn/large/0073tXM5gy1fy1y5sjavrj30f805njs6.jpg)

此时去请求发现报错
![image](https://wx3.sinaimg.cn/large/0073tXM5gy1fy1y0hz45pj30od02974p.jpg)
报错信息的意思是：在返回头`Access-Control-Allow-Headers` 字段中没有 `x-header2` 的信息，把 `x-header1 x-header2` 加进去就行了，最好是动态的获取请求头`Origin`里面的值去添加，这样就支持所有的自定义头部了。

### 虚拟主机上设置响应头信息
之前我们是直接在应用服务器上面修改响应头信息，现在我们建立一个虚拟主机，在虚拟主机上面修改响应头信息
被调用方的虚拟主机的配置。

第一步首先配置HOST文件eg:

```js
127.0.0.1 b.com  
```
打开nginx中的config目录 新建一个 `vhost`目录，在里面新建虚拟主机的配置文件。打开 `nginx.config` 文件,在最后增加

```bash
include vhost/*.conf  //让nginx载入这个目录下的所有.conf文件
```

在vhost目录中新建一个 `b.com.config`文件，里面写入下面代码

```
server{
    listen 80;        //监听的端口
    server_name b.com;   //监听的域名
    location /{
        proxy_pass http://localhost:8080/; //把所有的请求都转到 8080,监听80端口，域名为b.com
        add_header Access-Control-Allow-Methods *;
        add_header Access-Control-Max-Age 3600;
        add_header Access-Control-Allow-Credentials true;

        add_header Access-Control-Allow-Origin $http_origin;  //获取请求头里面的值
        add_header Access-Control-Allow-Headers $http_access_control_request_headers;
        if ($request_method = OPTIONS){
            return 200;   //把跨域的预检命令直接在虚拟主机上处理，不经过应用主机。
        }
    }
}
```

启动nginx `start nginx`

### 调用方解决跨域

通过反向代理来实现,隐藏跨域。

> 正向代理：它隐藏了真实的请求客户端，服务端不知道真实的客户端是谁，客户端请求的服务都被代理服务器代替来请求，正向代理的过程，它隐藏了真实的请求客户端，服务端不知道真实的客户端是谁，客户端请求的服务都被代理服务器代替来请求。

> 反向代理:它隐藏了真实的服务端，当我们请求 www.baidu.com 的时候，就像拨打10086一样，背后可能有成千上万台服务器为我们服务，但具体是哪一台，你不知道，也不需要知道，你只需要知道反向代理服务器是谁就好了，www.baidu.com 就是我们的反向代理服务器，反向代理服务器会帮我们把请求转发到真实的服务器那里去。Nginx就是性能非常好的反向代理服务器，用来做负载均衡。

> 两者的区别在于代理的对象不一样：正向代理代理的对象是客户端，反向代理代理的对象是服务端。
> [摘自知乎-刘志军的回答](https://www.zhihu.com/question/24723688)


```
127.0.0.1 b.com a.com //增加一个a.com，用它表示调用方的虚拟主机
```

在vhost目录中新建一个 `a.com.config`文件，里面写入下面代码

```
server {
    listen 80:
    server_name a.com;

    location /{
        proxy_pass http://localhost:8081/;
    }

    location /ajaxserver{ //把我们要调用的服务器代理成 ajaxserver
        proxy_pass http://localhost:8080/test/; 
    }
}
```

请求的代码中修改请求的基本前缀

```js
var base = '/ajaxserver';
```
