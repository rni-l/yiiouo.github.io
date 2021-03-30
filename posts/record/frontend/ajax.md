---
title: ajax
date: 2017-06-27 00:00:00
tags: ["js", "ajax"]
categories: ["记录"]
draft: true
---
# ajax 总结

> 总结 ajax 原生的使用，并写一个小库使用（这篇总结不兼容 ie7 和不说 XML）

## XMLHttpRequest 构造函数

`ajax` 整个流程是通过 `XMLHttpReuqest` 构造函数完成的。下面看看一段简单得 `get` 请求

```javascript
const xhr = new XMLHttpRequest()
xhr.onreadystatechange = function() {
  console.log(xhr)
}
xhr.open('get', '/api/user/checkLogin', true)
xhr.send(null)

```

首先生成一个 `xhr` 对象，监听 `onreadystatechagne` 方法，首先调用 `open` 传入参数，再调用 `send`，然后控制台就会打印出几条数据了。

下面一个个方法解析。

### open

`xhr.open('get', '/api/user/xxx', true)`,  `open` 方法接受三个三参数

1. 请求方法，例如： get, post, patch, put, delete...
2. 请求你地址
3. 是否异步

### post

`xhr.send(data)`, `send` 方法接受一个参数，调用了 `send` 方法后，就会正式开始发送请求。

### 发送的数据结构

通过 `send` 方法发送的数据，根据 `content-type` 会有不同的格式要求。

#### get

`get` 请求的参数放在请求的 Url 后面即可。`http://localhost:8080/api/user/get?id=1&num=2` 。但是参数都要经过 `encodeURIComponent` 转译才行，不然有些字符（像中文）会出现问题。

#### POST

想下面直接发送数据，后端接受的就是 `a=1` 的字符串

```javascript
xhr.open('post', '/api/user/login', true)
xhr.send('a=1')
```

通过设置 `Content-type` 为 `application/x-www-form-urlencoded`，浏览器就会对发送的主体进行处理，后端接受到的就是一个对象结构

```javascript
xhr.open('post', '/api/user/login', true)
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded')
xhr.send('a=1&b=2')

// 后端接受到的
// 伪代码
request.body === { a: 1, b: 2 }
```

`Content-Type` 设为 `application/json` 的话，发送的主体就要是一个对象

```javascript
xhr.open('post', '/api/user/login', true)
xhr.setRequestHeader('Content-Type', 'application/json')
xhr.send({ a: 1, b: 2 })

// 后端接受到的
// 伪代码
request.body === { a: 1, b: 2 }
```

`Content-Type` 如果是 `formData` 的话，就要传入 `formData` 类型的数据结构



### xhr 字段

#### readyState

表示请求/响应过程，属于哪个阶段

0. 未初始化，尚未调用 open
1. 启动，已调用 open,尚未调用 send
2. 发送，已调用 send, 尚未收到响应
3. 接受，接收到部分响应数据
4. 完成，接受到全部响应数据

#### setRequestHeader

`xhr.setRequestHeader('Content-Type', 'application/x-www.form-URLencoded')` 可以用来设置 `request` 的 `header`。

#### onreadystatechange

只要 `readyState` 的值进行了变化，就回触发一次

#### abort

`xhr.abort()` 会停止请求。

#### timeout

`xhr.timeout = 1000` ，单位是毫秒。1秒后如果数据还没接受完毕，就会停止请求，触发 `xhr.ontimeout` 事件

#### overrideMimeType

`xhr.overrideMimeType('text/xml')` 用来强制设置响应的 `mime` 类型。

### 进度事件

先不了解。。。



## 回答

> 总结下面试可能会问到的问题，如何用口述回答

### 什么是 ajax

通过 `XmlHTTPReuqest` 对象，与后台进行异步的数据传输和获取。不需要页面刷新，让后台把数据渲染到 html 上。

### 什么是跨域？如何解决？

跨域资源共享，CORS(cross-origin-resource-sharing)，当请求一个不同域的资源时，会发起一个跨域的 http 请求。当请求静态资源的时候，是允许进行跨域 http 请求，例如样式表、脚本、字体、canvas 的 `drawImage` 方法，或者 ajax 的请求。

不同协议、不同域、不同端口，这三者其中一个不同，都会属于跨域。

#### 解决跨域

1. 通过后台在 `response` 加上 `Access-Control-Allow-Origin` 等头部，允许跨域访问资源
2. 通过 node 开启代理。前端访问 node 服务器的接口，node 进行转发，去访问对应域名的这个接口，再把 response 返回给前端
3. 通过 jsonp 的方法。不过 jsonp 只支持 get。创建 `script` 去请求对应的数据，通过事先定义好的 `callback` 进行回调，拿到数据
4. nginx 转发
5. 



