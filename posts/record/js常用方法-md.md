---
title: js常用方法.md
date: 2019-07-02 21:41:34
tags: ["js"]
categories: ["记录"]
draft: true
---

### 防抖(debounce)

[参考](https://juejin.im/post/5b961773f265da0a9e52f0e3)

> 在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
>
> 对于一些处理时间比较久的函数，使用防抖会减低性能的消耗，对比 throttle，触发的次数会少
>
> 对于连续执行函数时，防抖只会执行一次

### 节流(throttle)

>在连续触发函数时，每隔 n 秒会触发一次函数



在 `github` 看到一位朋友 `bodyno` 说的一句解释：

> 一个是多久触发一次
>
> 一个是延迟多久触发



### 获取 url 的 query 参数

```javascript
function getQuery(str) {
  const arr = str.split('?')
  if (!arr[1]) return {}
  return arr[1].split('&').reduce((acc, cur) => {
    const val = cur.split('=')
    return {
      ...acc,
      [decodeURIComponent(val[0])]: decodeURIComponent(val[1])
    }
  }, {})
}
```

