---

layout: new
title: 基础总结_js_string
date: 2018-12-07
tags: ["js"]
categories: ["记录"]
draft: true
---

> 下面的内容仅供自身参考、学习而已，部分内容从 mdn、阮一峰的 《ES6标注入门》等内容摘取。



## 有关字符的知识

Unicode是目前绝大多数程序使用的字符编码，定义也很简单，用一个码点(code point)映射一个字符

码点值的范围是从U+0000到U+10FFFF

Unicode最前面的65536个字符位，称为基本平面（BMP-—Basic Multilingual Plane），它的码点范围是从U+0000到U+FFFF



## String 的静态方法

### fromCharCode

根据传入的参数，返回对应的 UTF-16 的字节码(Unicode)

```javascript
String.fromCharCode(56) // '8'

String.fromCharCode(55, 56, 57) // '789'
```

### fromCodePoint

返回指定的代码点序列创建的字符串

```javascript
String.fromCodePoint(42);       // "*"
String.fromCodePoint(65, 90);   // "AZ"
String.fromCodePoint(0x404);    // "\u0404"
```



## String 原型方法

### 根据数或字符，获取字符或索引或Unicode

#### charAt

返回传入索引值在字符串所对应的字符

```javascript
const txt = 'abcdefghijk'
txt.charAt(0) // a
txt.charAt(2) // c
txt.charAt('c') // 'a' // 传入字符串的话，会返回第一个值
txt.charAt('z') // 'a'
txt.charAt(111) // '' // 传入不存在的索引值，会返回空
```

#### charCodeAt

返回传入索引值在字符串所对应的字符的 Unicode

```javascript
const txt = 'abcdefghijk'
txt.charCodeAt(0) // 97
txt.charCodeAt(2) // 99
txt.charCodeAt('c') // 97 // 传入字符串的话，会返回第一个值
txt.charCodeAt('z') // 97
txt.charCodeAt(111) // '' // 传入不存在的索引值，会返回空
```

#### codePointAt

返回 一个 Unicode 编码点值的非负整数

```javascript
var icons = '☃★♲'
icons.codePointAt(1) // 9733
```

#### indexOf

返回传入字符串所在的索引值

未找到会返回 -1

`str.indexOf(searchValue[, fromIndex])`

```javascript
const txt = 'abcdefghijk'
txt.indexOf('a') // 0
txt.indexOf('c') // 2
txt.indexOf('b', 0) // 从第 0 位开始搜索 'b' 字符串的索引 1
txt.indexOf('b', 5) // 从第 5 位开始搜索 'b' 字符串的索引 -1
```

### 字符串的拼接、截取、替换等操作

#### concat

连接传入的字符串并返回一个新的字符串

```javascript
const a = 'a'
const b = 'b'
const c = 'c'
a.concat(b, c) // 'abc'
"".concat({}); // [object Object]
"".concat([]); // ""
"".concat(null); // "null"
"".concat(true); // "true"
```

#### match

根据正则返回匹配的值，null 或一个数组

```javascript
var paragraph = 'The quick brown fox jumped over the lazy dog. It barked.';
var regex = /[A-Z]/g;
var found = paragraph.match(regex);

console.log(found);
// expected output: Array ["T", "I"]
```

#### normalize

将字符串返回标准化的 Unicode 格式

```javascript
var first = '\u212B';         // "Å"
var second = '\u0041\u030A';  // "Å"

console.log(first + ' and ' + second + ' are' +
   ((first === second)? '': ' not') + ' the same.');
   // expected output: "Å and Å are not the same."

console.log(first + ' and ' + second + ' can' +
   ((first.normalize('NFC') === second.normalize('NFC'))? '': ' not') + ' be normalized');
   // expected output: "Å and Å can be normalized"

var oldWord = 'mañana';
var newWord = oldWord.normalize('NFD'); // 'The word did change.'
```

#### padEnd

`str.padEnd(targetLength [, padString])`

在字符串后面添加指定的字符串，直到整个字符串的长度在 `taretLength，并返回一个新的字符串`

```javascript
const a = 'a'
a.padEnd(10, 'test') // "atesttestt"
a.padEnd(2, '.') // 'a.')
```

#### padStart

和 `padEnd` 类似，在字符串开始部分添加

#### repeat

重复字符串 n 次，并返回一个新的字符串

`str.repeat(count)`

```javascript
const a = 'a'
a.repeat(3) // 'aaa'
```

#### replace

`str.replace(regexp|substr, newSubstr|function)`

根据传入的正则或字符串，对对象进行替换相应的字符串，并返回一个新字符串

```javascript
const test = 'sdf'
test.replace('sdf') // 'undefined'
test.replace('f') // 'sdundefined'
test.replace('f', '') // 'sd'
test.replace('f', () => 'd') // 'sdd'
test.replace(/f/, '') // 'sd'
```

#### search

`str.search(regexp)`

传入正则，返回要搜索的内容的索引值

```javascript
const test = 'sdf'
test.search(/f/) // 2
test.search(/fs/) // -1
```

#### slice

`str.slice(beginIndex[, endIndex])`

根据开始索引和结束索引值，截取相应的部分，返回一个新的字符串，不会对源字符串修改

```javascript
const test = 'sdf'
// 第一个索引值，是从字符串哪块开始截取
// 第二个索引值，是要截取多少个字符串
test.slice(0) // 'sdf'
test.slice(0, 1) // 's'
test.slice(0, 3) // 'sdf'
test.slice(0, n) // n >= test.length, 'sdf'
test.slice(1) // 'df'
test.slice(1, 1) // 'd'
// 如果是负数，则从后面开始截取
test.slice(-1) // 'f'
test.slice(-2) // 'df' 从后面第二个字符串开始截取
test.slice(-2, 2) // 'd' 从后面第二个字符串开始截取了两个字符，取索引号 2 之前的(索引号按原来的字符串的)
test.slice(-2, 3) // 'df'
```

#### split

`str.split([separator[, limit]])`

对一个字符串根据值进行分割，指定的值，不会出现在数组的字符串中，返回一个新的数组；可以传入 limit 值，指定返回多少个值

```javascript
const test = 'sdf'
test.split() // ['sdf']
test.split('') // ['s', 'd', 'f']
test.split('', 1) // ['s']
test.split('s') // ['', 'df']
```

#### substring

`str.substring(indexStart[, indexEnd])`

在开始和结束的索引内，返回一部分的字符串，不会改变源值

```javascript
var str = 'Mozilla';

str.substring(1, 3) // 'oz'
str.substring(2) // 'zilla'
str.substring(2, -1) // 'Mo'

```

#### toLocalLowerCase

`str.toLocaleLowerCase()
str.toLocaleLowerCase(locale) 
str.toLocaleLowerCase([locale, locale, ...])`

根据特定的语言环境进行映射，将字符串转为小写

MDN 文档的解释：`toLocaleLowerCase()方法返回调用该方法的字符串被转换成小写之后的值，转换规则根据任何本地化特定的大小写映射`**。**`toLocaleLowerCase()并不会影响字符串自身的值。在大多数情况下，该方法产生的结果和调用`[`toLowerCase()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/toLowerCase)的结果相同，但是在某些本地环境中，比如土耳其语，它的大小写映射并不遵循在Unicode中的默认的大小写映射，因此会有一个不同的结果

```javascript
var dotted = 'İstanbul';

console.log('EN-US: ' + dotted.toLocaleLowerCase('en-US'));
// expected output: "i̇stanbul"

console.log('TR: ' + dotted.toLocaleLowerCase('tr'));
// expected output: "istanbul"


'ALPHABET'.toLocaleLowerCase(); // 'alphabet'
// 区别：
'\u0130'.toLocaleLowerCase('tr') === 'i';    // true
'\u0130'.toLocaleLowerCase('en-US') === 'i'; // false

let locales = ['tr', 'TR', 'tr-TR', 'tr-u-co-search', 'tr-x-turkish'];
'\u0130'.toLocaleLowerCase(locales) === 'i'; // true
```

#### toLocalUpperCase

与上面类似，将字符串转为大写

#### toLowerCase

将字符串转为小写

```javascript
var sentence = 'The quick brown fox jumped over the lazy dog.';

console.log(sentence.toLowerCase());
// expected output: "the quick brown fox jumped over the lazy dog."
```

#### toUpperCase

将字符串转为大写

```javascript
var sentence = 'The quick brown fox jumped over the lazy dog.';

console.log(sentence.toUpperCase());
// expected output: "THE QUICK BROWN FOX JUMPED OVER THE LAZY DOG."
```

#### toString

`str.toString()`

返回对象的字符串表现形式

```javascript
var stringObj = new String("foo");

console.log(stringObj);
// expected output: String { "foo" }

console.log(stringObj.toString());
// expected output: "foo"
```

#### trim

`str.trim()`

移除字符串两端之间的空格、换行符等(space, tab, no-break space, etc., LF, CR, etc.)；返回一个新的字符串，不改变源字符串

```javascript
var greeting = '   Hello world!   ';

console.log(greeting);
// expected output: "   Hello world!   ";

console.log(greeting.trim());
// expected output: "Hello world!";
```

#### trimEnd && trimRight

移除字符串末尾的空格；`trimRight` 是这个方法的别名；返回一个新的字符串，不改变源字符串

#### trimStart && trimLeft

移除字符串开头的空格；`trimLeft` 是这个方法的别名；返回一个新的字符串，不改变源字符串

#### valueOf

`str.valueOf()`

返回 `String` 对象的原始值

```javascript
var stringObj = new String("foo");

console.log(stringObj);
// expected output: String { "foo" }

console.log(stringObj.valueOf());
// expected output: "foo"
```

#### raw

**String.raw()** 是一个[模板字符串](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/template_strings)的标签函数，它的作用类似于 Python 中的字符串前缀 `r` 和 C# 中的字符串前缀 `@`，是用来获取一个模板字符串的原始字面量值的。













### 判断

#### endsWith

判断当前字符串是否是以另外一个给定的子字符串“结尾”的

```javascript
const str1 = 'Cats are the best!';

console.log(str1.endsWith('best', 17)); // true

const str2 = 'Is this a question';

console.log(str2.endsWith('?')); // false
```

#### includes

判断一个字符串是否出现在另一个字符串中

```
const a = '123456'
a.includes('1') // true
a.includes(1) // true
a.includes(0) // false
```



#### startsWith

`str.startsWith(searchString[, position])`

判断某个字符串是否在开头出现；可以指定 position ，从第几个开始

```javascript
const str1 = 'Saturday night plans';

console.log(str1.startsWith('Sat'));
// expected output: true

console.log(str1.startsWith('Sat', 3));
// expected output: false
```



