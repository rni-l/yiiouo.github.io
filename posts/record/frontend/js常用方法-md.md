---
 title: js常用方法.md
date: 2019-07-02 21:41:34
tags: ["js"]
categories: ["记录"]
draft: true
---

### 防抖(debounce)

不断地执行一个函数时，如果函数在某个时间段再次被执行，则会重新计算时间而不执行函数

[参考](https://juejin.im/post/5b961773f265da0a9e52f0e3)

> 在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
>
> 对于一些处理时间比较久的函数，使用防抖会减低性能的消耗，对比 throttle，触发的次数会少
>
> 对于连续执行函数时，防抖只会执行一次

### 节流(throttle)

不断地执行一个函数时，过了某个时间还在执行的时候，就会再次被执行

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



### 判断数据类型方法

```javascript
function checkType(val) {
  return Object.prototype.toString.call(val).replace(/(\[object)|\]/g, '').toLowerCase()
}
```



### 加密字符串方法

```javascript
// 凯撒密码
const encodeCaesar = ({str = "", padding = 3}) =>
  !str
    ? str
    : str
        .split("")
        .map((s) => String.fromCharCode(s.charCodeAt() + padding))
        .join("");

const decodeCaesar = ({str = "", padding = 3}) =>
  !str
    ? str
    : str
        .split("")
        .map((s) => String.fromCharCode(s.charCodeAt() - padding))
        .join("");

console.log(encodeCaesar({str: "hello world"}));
console.log(decodeCaesar({str: "khoor#zruog"}));
```



### apply, call 实现

apply 和 call 类似，改变方法的上下文去执行；区别在传参，call 是一个个的传入，apply 是传入一个数组

```javascript
Function.prototype.mockApply = function(context, args) {
  // 缓存上下文
  context._fn = this
  // 执行方法
  const result = context._fn(...args)
  delete context._fn
  return result
}

Function.prototype.mockCall = function(context, ...args) {
  // 缓存上下文
  context._fn = this
  // 执行方法
  const result = context._fn(...args)
  delete context._fn
  return result
}

const obj1 = {
  a: 'obj1'
}

const obj2 = {
  a: 'obj2',
  fn: function(b, c) {
    return this.a + (b || '') + (c || '')
  }
}

console.log(obj2.fn())
console.log(obj2.fn.mockApply(obj1, ['b']))
console.log(obj2.fn.mockCall(obj1, 'b', 'c'))

```

### bind 实现

bind 就是柯里化函数，执行了会改变方法的上下文，并返回一个函数

```javascript
Function.prototype.mockBind = function(context, ...args) {
  let _this = this
  return function(...args2) {
    return _this.apply(context, [...args, ...args2])
  }
}

function bind(fn, context) {
  return function() {
    return fn.apply(context, arguments)
  }
}

const obj1 = {
  a: 'a1',
  fn: function(b, c) {
    return this.a + b + c
  }
}

const obj2 = {
  a: 'a2'
}

console.log(obj1.fn('b', 'c'))
console.log(obj1.fn.bind(obj2, 'b')('c'))

```



### curry

```javascript
function curry(...args) {
  if (args.length > 1) return args2.reduce((acc, cur) => acc + cur, 0)
  let val = 0
  function add(...args2) {
    val = args2.reduce((acc, cur) => acc + cur, val)
    return add
  }
  add.toString = function() { return val }
  val = args[0]
  return add
}

// curry(1,6)(2,5)(3,4)(7,8,9) -> 45
// curry(1) -> 1
```

这是个加法的 curry 函数

利用 `.toString` 方法和闭包特性，实现切割参数执行的功能

curry 函数的主要实现，是将被返回函数的参数进行排序



### flattenDeep

```javascript
function flattenDeep(arr) {
  return arr.reduce((acc, cur) => {
    const _val = Array.isArray(cur) ? flattenDeep(cur) : [cur]
    return [...acc, ..._val]
  }, [])
}


console.log(flattenDeep(
  [
    1, 2, 3, [4, 5, [6, 7, [8, 9]]]
  ]
))

```



### 深克隆

```javascript
function deepClone(val) {
  function getType(_val) {
    return Object.prototype.toString.call(_val).replace(/[\[\s\]]|(object)/g, '')
  }
  const valType = getType(val);
  if (valType !== 'Object' && valType !== 'Array') return val
  let output = {}
  if (valType === 'Array') {
    output = []
    val.forEach(v => {
      const curValType = getType(v)
      if (curValType === 'Array' || curValType === 'Object') {
        output.push(deepClone(v))
      } else {
        output.push(v)
      }
    })
  } else if (valType === 'Object') {
    Object.keys(val).forEach(v => {
      const curValType = getType(v)
      if (curValType === 'Array' || curValType === 'Object') {
        output[v]= (deepClone(val[v]))
      } else {
        output[v]= (val[v])
      }
    })
  }
  return output
}

const obj = {
  a: 1,
  b: [
    {
      b1: 'b1',
      b2: {
        b2_1: 'b2_1'
      }
    }
  ]
}

const clone = deepClone(obj)

obj.b = { b: 'b' }

console.log(clone)
console.log(obj)

```



### 实现并集

```javascript
function union(a, b) {
  return a.reduce((acc, cur) => {
    if (b.includes(cur)) acc.push(cur)
    return acc
  }, [])
}

const a = [1,2,3,4,5]
const b = [2,3,6,7,8,3,2,3,4]

// 期待输出：[2,3,4]

console.log(union(a, b))

```



## 一维数组转树

```javascript
function listToTree(data) {
  let arr = JSON.parse(JSON.stringify(data))
  const listChildren = (obj, filter) => {
    [arr, obj.children] = arr.reduce((res, val) => {
      if (filter(val))
        res[1].push(val)
      else
        res[0].push(val)
      return res
    }, [[],[]])
    obj.children.forEach(val => {
      if (arr.length)
      listChildren(val, obj => obj.parentId === val.id)
    })
  }

  const tree = {}
  listChildren(tree, val => arr.findIndex(i => i.id === val.parentId) === -1)
  return tree.children
}
```





