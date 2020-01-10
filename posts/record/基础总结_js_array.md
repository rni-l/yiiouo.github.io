---

title: 基础总结_js_array
date: 2018-12-11
tags: ["js"]
categories: ["记录"]
draft: true
---

> 下面的内容仅供自身参考、学习而已，部分内容从 mdn、阮一峰的 《ES6标注入门》等内容摘取。



# Array

## 静态方法

### from

`Array.from(arrayLike[, mapFn[, thisArg]])`

创建一个新的数组，浅复制一个来自类数组或可迭代的对象，生成数组实例

```javascript
console.log(Array.from('foo'));
// expected output: Array ["f", "o", "o"]

console.log(Array.from([1, 2, 3], x => x + x));
// expected output: Array [2, 4, 6]

Array.from({ length: 12 }) // [undefined, ...., undefined]

// 对 set map 都可以进行躺平
// thisArg 的参数，是执行回调函数时，this 的镀锡
```

### isArray

`Array.isArray(value)`

判断传入的值，是否数组，返回 true false

### of

`Array.of(element0[, element1[, ...[, elementN]]])`

根据传入的参数，创建相应长度、按顺序的值的数组

```javascript
Array.of(7) // [7]
Array.of(1,2,3) // [1,2,3]
// 和 Array 方法不一样
Array(7);          // [ , , , , , , ]
Array(1, 2, 3);    // [1, 2, 3]
```



## 原型上的方法

###拼接、处理、移除

#### concat

`var new_array = old_array.concat([value1[, value2[, ...[, valueN]]]])`

在某个数组的基础下，浅复制参数内数组的值，并返回一个新值，不改变原来的值

```javascript
const a1 = [1, 2]
const a2 = [3, 4]
a1.concat(a2) // [1,2,3,4]
a1.concat(1) // [1,2,1]
a1.concat([a2]) // [1, 2, [[3, 4]]]
a1.concat(a2, 1, 2) // [1,2,3,4,1,2]
const a3 = [{ a: 1 }]
const a4 = a1.concat(a3) // [1,2, { a: 1 }]
a4[2].a = 2
a3[0].a // 2, not 1
```

#### copyWithin

`arr.copyWithin(target[, start[, end]])`

浅复制数组的一部分到同一数组中的另一个位置，并返回它，而不修改其长度大小

1. target
    1. 0 为基底的索引，复制序列到该位置。如果是负数，`target` 将从末尾开始计算
    2. 如果 `target` 大于等于 `arr.length`，将会不发生拷贝。如果 `target` 在 `start` 之后，复制的序列将被修改以符合 `arr.length`
2. start
    1. 0 为基底的索引，开始复制元素的起始位置。如果是负数，`start` 将从末尾开始计算
    2. 如果 `start` 被忽略，`copyWithin` 将会从0开始复制
3. end
    1. 0 为基底的索引，开始复制元素的结束位置。`copyWithin` 将会拷贝到该位置，但不包括 `end` 这个位置的元素。如果是负数， `end` 将从末尾开始计算
    2. 如果 end 被忽略，copyWithin 将会复制到 arr.length

```javascript
var array1 = [1, 2, 3, 4, 5];

// place at position 0 the element between position 3 and 4
console.log(array1.copyWithin(0, 3, 4));
// expected output: Array [4, 2, 3, 4, 5]
// 从数组的第三位开始取，取到第四位(取4)，在第 0 位开始覆盖粘贴

// place at position 1 the elements after position 3
console.log(array1.copyWithin(1, 3));
// expected output: Array [4, 4, 5, 4, 5]
// 从数组第三位开始取，取到尽头，在原数组的第1位开始覆盖粘贴
```

#### fill

`arr.fill(value[, start[, end]])`

根据 start 和 end 的索引值，填充指定的值。会修改原来的数组

```javascript
var array1 = [1, 2, 3, 4];

// fill with 0 from position 2 until position 4
console.log(array1.fill(0, 2, 4));
// expected output: [1, 2, 0, 0]

// fill with 5 from position 1
console.log(array1.fill(5, 1));
// expected output: [1, 5, 5, 5]

console.log(array1.fill(6));
// expected output: [6, 6, 6, 6]
```

#### find

`arr.find(callback[, thisArg])`

循环节点，回调函数返回 true 时，终止循环，返回当前节点，如果不存在返回 null

```javascript
var array1 = [5, 12, 8, 130, 44];

var found = array1.find(function(element) {
  return element > 10;
});

console.log(found);
// expected output: 12
```

#### findIndex

`arr.findIndex(callback(element[, index[, array]])[, thisArg])`

循环节点，回调函数返回 true 时，终止循环，返回当前节点的索引值，如果不存在返回 -1

#### indexOf

`arr.indexOf(searchElement[, fromIndex])`

返回找到的第一个值，如果没找到返回 -1；fromIndex 从第几位开始找

```javascript
var beasts = ['ant', 'bison', 'camel', 'duck', 'bison'];

console.log(beasts.indexOf('bison'));
// expected output: 1

// start from index 2
console.log(beasts.indexOf('bison', 2));
// expected output: 4

console.log(beasts.indexOf('giraffe'));
// expected output: -1
```

#### join

`arr.join([separator])`

返回一个新的字符串，根据数组的每个值，在它们之间加入间隔符（默认为','）

```javascript
var elements = ['Fire', 'Wind', 'Rain'];

console.log(elements.join());
// expected output: Fire,Wind,Rain

console.log(elements.join(''));
// expected output: FireWindRain

console.log(elements.join('-'));
// expected output: Fire-Wind-Rain
```

#### lastIndexOf

和 `indexOf` 类似，区别是找到最后一为相同的索引值

#### pop

移除数组的最后一位，返回移除那个节点的值

#### push

在数组最后一次，添加新的值，返回数组的长度

#### reverse

将一个数组的值对换，返回一个新的值，有副作用的

#### shift

移除数组的开头的值

#### slice

`arr.slice([begin[, end]])`

根据 begin 和 end（不包含 end 的值），浅复制数组部分的节点，返回组成的数组，没副作用

begin: 为负数，`则表示从原数组中的倒数第几个元素开始提取`

```javascript
var animals = ['ant', 'bison', 'camel', 'duck', 'elephant'];

console.log(animals.slice(2));
// expected output: Array ["camel", "duck", "elephant"]

console.log(animals.slice(2, 4));
// expected output: Array ["camel", "duck"]

console.log(animals.slice(1, 5));
// expected output: Array ["bison", "camel", "duck", "elephant"]
```

#### sort

`arr.sort([compareFunction])`

默认的执行函数根据字符串的 Unicode 的 code point 进行排序的

```javascript
var months = ['March', 'Jan', 'Feb', 'Dec'];
months.sort();
console.log(months);
// expected output: Array ["Dec", "Feb", "Jan", "March"]

var array1 = [1, 30, 4, 21];
array1.sort();
console.log(array1);
// expected output: Array [1, 21, 30, 4]
```

#### splice

`array.splice(start[, deleteCount[, item1[, item2[, ...]]]])`

有副作用

```javascript
var months = ['Jan', 'March', 'April', 'June'];
months.splice(1, 0, 'Feb');
// 如果第二参数为 0，则根据 start 值，插入相应的值
console.log(months);
// expected output: Array ['Jan', 'Feb', 'March', 'April', 'June']

months.splice(4, 1, 'May');
// replaces 1 element at 4th index
console.log(months);
// expected output: Array ['Jan', 'Feb', 'March', 'April', 'May']
```

#### toLocaleString

`arr.toLocaleString([locales[,options]]);`

将数组的节点用 toLocaleString 转成字符串，并用 ',' 隔开，返回一个新字符串

```javascript
var array1 = [1, 'a', new Date('21 Dec 1997 14:12:00 UTC')];
var localeString = array1.toLocaleString('en', {timeZone: "UTC"});

console.log(localeString);
// expected output: "1,a,12/21/1997, 2:12:00 PM",
// This assumes "en" locale and UTC timezone - your results may vary
```

#### toString

`arr.toString();`

将数组的节点用 toString 转成字符串，并用 ',' 隔开，返回一个新字符串

#### unshift

在数组开头添加元素，并返回数组的长度



### 判断

#### every

`arr.every(callback(element[, index[, array]])[, thisArg])`

循环所有的节点，根据每次回调函数返回的值，判断最后的结果是 true or false；如果有一次出现 false，就会终止循环，返回 false

```javascript
function isBelowThreshold(currentValue) {
  return currentValue < 40;
}

var array1 = [1, 30, 39, 29, 10, 13];

array1.every(isBelowThreshold) // true

var array2 = [1, 30, 40, 29, 10, 13];

array2.every(isBelowThreshold) // false
```

#### includes

`arr.includes(searchElement[, fromIndex])`

判断数组里是否有传入的值，返回 true or false；fromIndex 判断从第几位开始

```js
[1, 2, 3].includes(2);     // true
[1, 2, 3].includes(4);     // false
[1, 2, 3].includes(3, 3);  // false
[1, 2, 3].includes(3, -1); // true
[1, 2, NaN].includes(NaN); // true
```

#### some

`arr.some(callback(element[, index[, array]])[, thisArg])`

循环节点，某个节点返回 false，则终止循环，返回 false，否则返回 true



### 迭代器

#### entries

返回一个新的数组迭代器，包含 key/value

```javascript
var array1 = ['a', 'b', 'c'];

var iterator1 = array1.entries();

console.log(iterator1.next().value);
// expected output: Array [0, "a"]

console.log(iterator1.next().value);
// expected output: Array [1, "b"]	
```

#### keys

返回一个新的数组迭代器，包含 key

```javascript
var array1 = ['a', 'b', 'c'];
var iterator = array1.keys(); 
  
for (let key of iterator) {
  console.log(key); // expected output: 0 1 2
}
```

#### values

返回一个新的数组迭代器，包含 values

```javascript
const array1 = ['a', 'b', 'c'];
const iterator = array1.values();

for (const value of iterator) {
  console.log(value); // expected output: "a" "b" "c"
}
```



### 循环

#### filter

`var newArray = arr.filter(callback(element[, index[, array]])[, thisArg])`

循环所有的节点，根据回调函数返回的 true 时，把当前的节点加到新数组里面

```javascript
var words = ['spray', 'limit', 'elite', 'exuberant', 'destruction', 'present'];

const result = words.filter(word => word.length > 6);

console.log(result);
// expected output: Array ["exuberant", "destruction", "present"]
```

#### forEach

`arr.forEach(function callback(currentValue[, index[, array]]) {
  //your iterator
}[, thisArg]);`

执行回调函数，循环数组里的所有节点

#### map

`var new_array = arr.map(function callback(currentValue[, index[, array]]) {
  // Return element for new_array
}[, thisArg])`

返回每次循环节点时，回调函数返回的值，而组成的数组

#### reduce

`arr.reduce(callback[, initialValue])`

1. callback
    1. accumulator，迭代的值
    2. currentValue
    3. currentIndex
    4. array
2. initialValue，初始的值

返回一个新的值，循环每个节点，将每次回调函数返回的值，传入到下一个节点的回调函数参数上

#### reduceRight

和 reduce 类似，区别是从末尾开始















