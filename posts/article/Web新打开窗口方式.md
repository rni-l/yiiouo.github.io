---
title: Web新打开窗口方式
date: 2018-06-27 00:00:00
tags: ["js"]
categories: ["分享"]
---

>之前的项目，有个功能是下载文件，这里只要在浏览器输入 url 就会下载那个文件了。当时我只是简单得使用 `window.open` ，但是却会被浏览器进行拦截，要手动开启才行，然后就搜索研究其他方法，就看到各种各样的，通过 js 打开新窗口的方法了，这里就总结一下

## 解决下载功能

这里就先说解决下载功能的方法，通过同事的提醒，改用 iframe 进行处理，直接对 iframe 的 src 进行赋值，就会自动进行下载文件了，不过，如果后端在 response  header 设置了某个头部，就会报错了：x-frame-options: DENY

x-frame-options，是否允许 object 和 iframe 展示，有三个参数：

* DENY: 即使是相同域名，也不能显示
* SAMEORIGIN: 可以在相同域名页面展示
* ALLOW-FROM uri: 任何来源都可以展示

[MDN的解释](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/X-Frame-Options)

我的后端同事在要下载的几个接口中，把 `x-frame-options` 设置成 `SAMEOPIGIN` 后，前端就可以在无任何副作用的情况下，通过 js 进行下载文件了

    export: (url) => {
      // 移除旧的节点
      const oldNode = document.querySelector('#g-exportOrder-iframe')
      if (oldNode) {
        document.body.removeChild(document.querySelector('#g-exportOrder-iframe'))
      }
      // 生成新节点，进行下载
      const iframe = document.createElement('iframe')
      iframe.style.display = "none"
      iframe.id = 'g-exportOrder-iframe'
      iframe.src = url
      document.body.appendChild(iframe)
    }

只要调用传个 url ，就会自动下载一个文件了

## 使用 window.open

如果我们是点击一个目标，然后同步执行打开窗口操作，用 `window.open` 是可以的，但是我们把 `window.open` 放在异步操作里就有问题了

    div.addEventListener('click', open, false)
    function open() {
      setTimeout(() => {
        window.open('/api/admin/adslot/all')
      }, 1100)
    }

我在谷歌、火狐和欧朋，这样就会被拦截，然而用 ie9 却不会被拦截，我给10秒，ie 最后还是会弹出来

从你用点击事件，到 `window.open` ，只要异步操作超过某个时间，浏览器就会拦截这个弹窗的操作

如果不添加用户的事件去触发 `window.open` （比如点击事件，鼠标移入移出等），而是在代码直接运行 `window.open` 的话，那样浏览器也会拦截

    window.onload = function() {
      windon.open()
    }

总得来讲，如果没有用用户操作的事件去触发 `window.open` 就会被拦截，而把 `window.open` 放在异步操作，且超过一定的事件，也会被拦截

这里先想到了解决异步也会被拦截的方法

    var test = window.open()
    setTimeout(function() {
      test.location = 'http://www.xxx.com'
    }, 2000)

在异步操作前，先打开窗口，然后再在你要操作的位置，更改这个窗口的 `location` ，不过这个缺陷有点大，要等异步操作完成了，新的窗口才会从空白变到指定页面，而且这种解决不了，没有人为事件触发 `window.open` 导致被拦截的问题

## 使用 a 标签

这是最常见打开一个新标签页面的方法

    <a class='test' href='http://www.xxx.com' target='_blank'></a>

然后，我点击另外一个 div ，再打开新窗口

    function open() {
      setTimeout(function() {
        document.querySelector('.test').click()
      }, 2000)
    }

上面的异步操作，还是不行，就算是重新生成一个 a 标签，再用 `click()` 触发也是不行

## form submit

    <form class='test' target='_blank' @click='open' method='GET' action='http://www.xxx.com'>click me</form>
    function open2() {
      setTimeout(() => {
        document.querySelector('.test').submit()
      }, 2000)
    }

和 a 标签一样

[DEMO 链接](http://www.rni-l.com/plugins/demo/windowOpen.html)

## 总结

如果要下载文件的话，使用 iframe

如果要打开新窗口，而且没有用户操作的前提下打开，是不能显示的，只能提示让用户关闭那个拦截吧

有用户操作，且是异步的情况下，就使用 `window.open` ，然后定义 `location` 这样就好了