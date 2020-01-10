---
title: regexp
date: 2019-05-04 11:53:11
tags: ["js"]
categories: ["记录"]
---

## 常用匹配

### 获取两个字符之间的内容

1. 获取某个字符之前的字符串： str.match(/(\S*)a/)[1]
2. 获取某个字符之后的字符串：str.match(/a(\S*)/)[1]
3. 获取两个字符串之间的字符串：str.match(/a(\S*)b/)[1]



### 注意事项

#### 小心 `g`

全局匹配符有个陷阱，多次匹配同一个字符串，可能会出现不同的结果

引用 mdn 文档说明：

> ​	如果正则表达式设置了全局标志，`test() `的执行会改变正则表达式   [`lastIndex`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/lastIndex)属性。连续的执行`test()`方法，后续的执行将会从 lastIndex 处开始匹配字符串，([`exec()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec) 同样改变正则本身的 `lastIndex属性值`).

```javascript
const reg = /hi/g
  console.log(reg.test('hi')) true
  console.log(reg.test('hi')) false
  console.log(reg.test('hi')) true
```

三次检验的结果，不一样

这里说明下 `lastIndex` ，这个是 `RegExp` 对象的属性，只当使用了 `g` 的时候，才会有用

[mdn](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/lastIndex)

>`lastIndex` 是正则表达式的一个可读可写的整型属性，用来指定下一次匹配的起始索引。

具体规则：

>- 如果 `lastIndex` 大于字符串的长度，则 `regexp.test` 和 `regexp.exec` 将会匹配失败，然后 `lastIndex` 被设置为 0。
>- 如果 `lastIndex` 等于字符串的长度，且该正则表达式匹配空字符串，则该正则表达式匹配从 `lastIndex` 开始的字符串。（then the regular expression matches input starting at `lastIndex`.）
>- 如果 `lastIndex` 等于字符串的长度，且该正则表达式不匹配空字符串 ，则该正则表达式不匹配字符串，`lastIndex` 被设置为 0.。
>- 否则，`lastIndex` 被设置为紧随最近一次成功匹配的下一个位置。

下面解释下每种情况的逻辑：

```javascript
const reg = /hi/g
console.log(reg.lastIndex, reg.test('hi')) // 0 true
console.log(reg.lastIndex, reg.test('hi')) // 2 false;触发了第三条规则，匹配失败，然后 lastIndex 设置为 0;因为 test 方法是从 lastIndex 开始匹配，所以从第三位字符串匹配的时候，失败了
console.log(reg.lastIndex, reg.test('hi')) // 0 true
console.log(reg.lastIndex) // 2
```

```java
const reg = /\w*/g
console.log(reg.lastIndex, reg.test('hi')) // true
console.log(reg.lastIndex, reg.test('1hi')) // 2 true
console.log(reg.lastIndex, reg.test('hi')) // 3 false；触发第一条规则，lastIndex 大于字符串长度，所以变位0,且失败
console.log(reg.lastIndex) // 0
```

使用 `g` 时，尽量不要缓存该正则