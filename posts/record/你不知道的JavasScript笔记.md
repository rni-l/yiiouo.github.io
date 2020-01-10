---
layout: new
title: 你不知道的JavasScript笔记
date: 2017-09-03 21:59:38
tags: ["js"]
categories: ["记录"]
draft: true
---

> 记录读这本书的时候，一些笔记，方便以后再看

# 第一章

## 1.1作用域是什么

传统编译语言，代码在执行前，会经历三个步骤：

1. 分词/词法分析

  词法分析，会把一段代码分成一个个模块：`var a = 1`，会被分成`var  a  =  1` 四个部分

2. 解析/语法分析

  将词法单元流（数组）转换成一个由元素组成的树（AST），抽象语法树

3. 代码生成

  根据语言将AST转换可执行代码


一段代码的执行，需要三个“帮手”，

* 引擎
  
  负责整个程序的编译和执行

* 编译器

  负责语法分析和代码生成

* 作用域

  收集和维护所有声明的标识符，按照规定的规则，对标识符是否有访问的权限

```javascript
var a = 1
```

**编译器会询问作用域是否已经有 a 变量的存在，如果是继续编译，否则会在当前作用域创建一个 a 变量**

编译器在生成代码时，会有两个查询：LHS , RHS

* LHS

  LHS查询，会对当前变量在当前作用域查找它本身，找到后就会进行赋值，找不到会从当前作用域往上找，直到找到为止，如果没有，就会生成一个新的变量，并进行赋值。如果操作是非法（对undefiend, null进行赋值，就会报`Typeerror`错误）

* RHS

  RHS查询，对目标进行查找并且获取值，先在当前作用域寻找，然后往上一级级查找，如果在所有的作用域都找不到`a`的话，就会报`ReferenceError`错误


作用域是一套规则，规定在何处如何查找变量

作用域的查找，会在当前作用域查询，找不到就会一步一步往上找，直到全局作用域。


## 1.2词法作用域

JavaScript 使用的是词法作用域

词法作用域是定义在词法阶段，就是代码将变量和块作用域写在哪里决定（词法作用域由函数被声明时所在的位置所决定）

[词法作用域和动态作用域的解析](https://github.com/mqyqingfeng/Blog/issues/3)

词法作用域（lexical scoping），也叫静态作用域

```javascript
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f();
}
checkscope();

// or
var scope = "global scope";
function checkscope(){
    var scope = "local scope";
    function f(){
        return scope;
    }
    return f;
}
checkscope()();
```

和 `this` 的判断有区别

不要使用`eval`和`with`

## 1.3函数作用域和块作用域

`function`是声明的一个词，就是一个函数声明，否则就是函数表达式

IIFE，立即执行函数表达式

UMD，将运行的函数放在第二位，通过参数导入运行

块作用域

```
try {
  // trigger error
} catch (e) {
  console.log(e)
}
console.log(e) // error
```

try/catch 是有块作用域的，在第六话打印的 e 对象，会报错，而 catch 里面的 e 却是正常的

`let`不会在块作用域，声明提升

    var a = 1
    console.log(a)
    obj.addEventListener('click', function(e) {
      // ...
    }, false)

上面代码，打印`a`后，就没有再引用到`a`，理论上应该是被回收了。然而`click`点击事件函数，覆盖了整个作用域的闭包，引擎很有可能会保存这个结构 

    {
      let a = 1
      console.log(a)
    }

通过块结构，让引擎回收`a`

## 1.4提升

在运行的时候，编译器会先编译 js 代码，遇到 `var`、`function` 等声明的代码，会在当前作用域查找是否存在，不存在就会创建。然后等执行的时候，就会进行赋值操作等。

先声明，后赋值。使用`var`和函数声明，都会把变量提升

避免重复声明同一个变量函数

函数提升会比变量提升靠前

```javascript
foo() // 1

var foo

function foo() {
  console.log(1)
}

foo = function() {
  console.log(2)
}

------

// 首先foo(1)，然后foo(3)覆盖了foo(1)
foo() //3

function foo() {
  console.log(1)
}

var foo = function() {
  console.log(2)
}

function foo() {
  console.log(3)
}

-----

foo() // 'b'
var a = true
if (a) {
  function foo() { console.log('a') }
} else {
  function foo() { console.log('b') }
}
 
```

## 1.5 作用域闭包

 

# 第二章

## 2.1 关于this

`this`的上下文，取决于函数调用时的条件，与函数所在的位置无关

## 2.2 this全面解析

理解函数调用的位置。首先分析`调用栈` --> 就是为了到达当前执行位置所调用的所有函数

    // 书中例子
    function baz() {
      // 调用栈：baz
      // 调用位置： window
      console.log('baz')
      bar()
    }
    
    function bar(){
      // 调用栈： baz -> bar
      // 调用位置：baz
      console.log('bar')
      foo()
    }
    
    function foo(){
      // 调用栈： baz -> bar -> foo
      // 调用位置：bar
      console.log('foo')
    }

根据函数执行的区域，判断调用的位置

绑定规则：

### 默认绑定，在非严格模式下，当`function`在全局调用，`this`默认指向`window`

### 隐式绑定，根据调用位置的上下文对象，也就是说调用这个`function`时，的作用域是什么

    function foo() {
      console.log(this.a)
    }
    
    var obj = {
      a: 2,
      foo: foo
    }
    
    obj.foo() // 2

上面代码，调用`foo`时，是在obj的上下文对象下执行，所以`this`绑定到`obj`对象上。对象属性引用链治国在最后一层的调用位置上起作用

隐式绑定有个常见的问题：隐式丢失

    function foo() {
      console.log(this.a)
    }
    
    var obj = {
      a: 2,
      foo: foo
    }
    
    var bar = obj.foo // 引用了foo
    
    var a = 'global'
    
    bar() // 'global'

上面代码，`bar`引用了`foo`函数，再调用时，是在全局上下文执行的，所以`this`绑定到全局对象上

### 显示绑定

    function foo() {
      console.log(this.a)
    }
    
    var obj = {
      a: 2,
      foo: foo
    }
    
    foo.call(obj) // 2

上面代码，强制把`this`绑定到`obj`上

#### 硬绑定：

    function foo() {
      console.log(this.a)
    }
    
    var obj = {
      a: 2,
      foo: foo
    }
    
    var bar = function() {
      foo.call(obj)
    }
    
    bar() // 2
    setTimeout(bar, 100) // 2
    bar.call(window) // 2

上面代码，在`bar`函数内部，直接调用`foo.call(obj)`，所以无论`bar`在哪里调用，`foo`的上下文都是`obj`

### new绑定，使用new来调用函数时，会发生以下操作：

1. 创建一个新的对象
2. 这个新对象会被执行到`[[Prototype]]`连接
3. 这个新对象会绑定到函数调用的this
4. 如果函数没有返回其他对象，那么`new`后会返回这个新对象

    function foo(a) {
      this.a = a
    }
    
    var bar = new foo(2)
    console.log(bar.a) // 2

### 优先级

默认绑定是最低级的

显式大于隐式

new大于隐式

显示绑定后的对象，new一个新的实例，`this`是指向新的实例

### 绑定例外

`foo.call(null)`, 会触发默认绑定

### 箭头函数

根据外层作用域来判断`this`的指向

## 2.3 对象

### 类型

在`js`中，有六种基本类型：

* string
* number
* boolean
* null
* undefined
* object

为什么`typeof null`也是'object'呢？原理：不同的对象在底层都表示为二进制，在`js`中，前三位为`0`，判断为`object`，而`null`的二进制全为0，所以才会出现`object`

基本类型都有其内置函数

### 复制对象

有浅拷贝和深拷贝

#### 浅拷贝

基本类型值会复制，而对象，则会引用

#### 深拷贝

对象，把每一层的值，复制到新的对象里，如果是数组和对象，就会遍历它，一个个的复制

`JSON.parse(JSON.stringify(data))`，粗暴的复制

### 属性描述符

    var my = {
      a: 2
    }
    Object.getOwnPropertyDescriptor(my, '1')
    // {
    //  value: 2,
    //  writable: true,  可写
    //  enumerable: true,  可枚举
    //  configurable: true  可配置
    // }

使用`getOwnPropertyDescriptor`，就能获取到这几个属性描述符

我们可以使用`defineProperty`来设置这4个值

    var my = {}
    Object.getOwnPropertyDescriptor(my, 'a', {
     value: 2,
     writable: true,  可写
     enumerable: true,  可枚举
     configurable: true  可配置
    })

#### writable

设置为`false`，就不能再修改这个对象，修改了也会无效，严格模式下回报错

#### enumerable

设置为`false`，`for..in`就不会枚举到此属性

#### configurable

设置为`false`,不可配置，就不能设置这四个属性

### Getter and Setter

`Getter` 和 `Setter` 是一个隐藏函数，在获取值和设置值的时候，会触发这个函数

当一个熟悉定义了`Getter` 和 `Setter` ，就会被定义为“访问描述符”

    var myO = {
      get a() {
        return this._a_
      },
      set a(val) {
        this._a_ = val
      }
    }
    // 或者使用defineProperty定义
    Object.defineProperty(myO, b, {
      get: function() {
        return this.a * 2
      },
      enumerable: true
    })

### 存在性

判断属性值是 `undefined` 还是没有这个属性

    var myO = {
      a: 2
    }
    ("a" in myO) // true
    ("b" in myO) // false
    myO.hasOwnProperty("a") // true
    myO.hasOwnProperty("b") // false

`in` 操作符会检查属性是否在对象和原型链中，而 `hasOwnProperty` 不会检查原型链

使用获取、检查属性的方法适合，要确认是否会检查原型链



