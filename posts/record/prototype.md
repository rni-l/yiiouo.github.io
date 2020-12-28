---
title: prototype
date: 2018-06-27 00:00:00
tags: ["js"]
categories: ["记录"]
draft: true
---
# JS 原型

> 了解原型是什么、有什么用，如何实现继承。。。

## 原型

### 什么是原型

原型（[[Prototype]]）的机制就是对象中的一个内部链接引用另一个对象

每个 `object` 都有自己的 `__proto__`，而 `__proto__` 指向它自身的原型对象，其中的 `constructor` 指向生成该对象的构造函数。所有的 `object` 值，原型链的终点都是 `Object`。而 `Object` 的原型对象是 `null`, `null` 没有原型。

```javascript
const Animal = function() {}

Animal.prototype.call = function(name) {
  console.log(name)
}
const a = new Animal()
console.log(a)
// 打印后，会看到 a 对象有一个 __proto__ 的值，展开它
// 会看到 call, constructor, __proto__ 这三个值
// call 就是上面添加的原型方法
// constructor 就是 Animal 构造函数
// __proto__ 指向了 Object 的原型

```

当我们获取属性或者调用方法的时候，js 会从自身属性再到原型链，一步一步地往上找，直到找到为止。

```javascript
const Animal = function() {
  // this.a = 1
}

Animal.prototype.call = function(name) {
  console.log(name)
}
// Animal.prototype.a = 2
Object.prototype.a = 3
Object.a = 4

const a = new Animal()

console.log(a, a.a) // 3

// 当我们给 Object 的原型添加一个属性，再访问 a 的 a 属性，会得到 Object 的值
// 如果把 Animal 那行取消注释，会得到 2
// 如果再把 this.a = 1 取消注释，会得到 1
// 4 的值，怎么都不会从 a 身上得到
```

原型链的属性和方法和共用的。

```javascript
const a = new Animal()
```

这个 `new` 的操作会大概会是以下几个步骤：

1. 生成一个空对象
2. 将对象的 `__proto__` 指向构造函数的 `prototype`
3. 将 `Animal` 构造函数的作用域赋给对象（ `this` 指向新对象）
4. 让 `this` 指向这个对象
5. 执行函数
6. 判断函数返回值的类型，如果是基础类型的，就返回对象
7. 如果是引用类型，就返回“函数的返回值”

mock new 操作符

```javascript
function mockNew(func, ...args) {
  let obj = Object.create(func.prototype)
  obj1 = func.apply(obj, ...args)
  return obj1 instanceof Object ? obj1 : obj
}

```





### 原型的特征

1. 原型之间会通过 `__proto__` 进行链接，这个就是原型链
2. 原型链之间的方法和属性是共用的
3. 原型中的 `constructor` 指向的是当前原型的构造函数
4. 查找一个对象的属性，会先从实例自身查找，再一步步沿着 `prototype` 去找，找不到就返回 `undefined` 



### __proto__，prototype，constructor 之间的关系

```javascript
function Test() {}

Test.prototype.publicValue = 'test'
Test.privateValue = 'test2'
const test1 = new Test()

console.log(Test.prototype.__proto__ === Object.prototype) // true
console.log(Test.prototype.constructor === Test) // true
console.log(test1.__proto__ === Test.prototype) // true
Object.prototype.__proto__ === null

```

[结构图](https://www.processon.com/diagraming/5c1df7dde4b0e83682eb7712)





### 创建抽象的对象

#### 为什么要用原型？

```javascript
function Animal(name, type) {
  this.name = name
  this.type = type
  this.getName = function() {
    console.log(this.name)
  }
}

const cat1 = new Animal('cat1', 'cat')
const cat2 = new Animal('cat2', 'cat')

cat1.getName === cat2.getName // false

```

通过构造函数生成的实例，每个实例之间的属性都是一个新的内存地址，所以他们之间是不一样。大部分的属性是必须的，但是共用的属性/方法，我们可以通过 `this` ，使其变为动态的，所以这就没必要每 `new` 一个实例就要重新创建一次方法，这样会造成很多的消耗。而这就可以用原型进行解决。

#### 原型模式

```javascript
function Animal() {}
Animal.prototype.name = 1
Animal.prototype.getName = () => 2

const a = new Animal()
const b = new Animal()
```

但是如果单单使用原型模式，里面的全部属性和方法都是共用的，这样就做不到实例之间有独立的属性存在。

#### 构造函数 + 原型模式

把这两者合并使用，属性通过构造函数生成，共用的属性/方法放到原型进行共用

```javascript
function Animal(name) {
  this.name = name
}
Animal.prototype.getName = function() {
  consolelog(this.name)
}
```

这样就可以达到实例的属性是单独的，方法是共用的。减低性能的消耗，又实现了动态的效果。

#### 寄生构造函数模式

```javascript
function Test() {
  const values = new Array()
  values.push.apply(values, arguments)
  values.testSplit = function() {
    return this.join('|')
  }
  return values
}

const a = new Test()
a.push(1)
console.log(a)
```

最主要的作用就是，在原生对象的基础上添加额外的方法，且不会影响原生对象。

### 继承

#### 利用原型链继承

```javascript
const Animal = function(name) {
  this.name = name
}
Animal.prototype.getName = function() {
  console.log(this.name)
}
const Cat = function() {}
Cat.prototype = new Animal()
Cat.prototype.getCat = function() {
  console.log('getCat')
}

const a = new Animal('animal')
const cat = new Cat()
```

这样  `Cat` 构造函数的原型就会继承到 `Animal` 的原型，并且后续可以添加属于自己的属性，从而达到继承的效果。

但是这样不能通过传参，继承父类自身的属性。

#### 组合继承

```javascript
const Animal = function(name) {
  this.name = name
}

Animal.prototype.call = function(name) {
  console.log(name)
}
Animal.prototype.getName = function() {
  console.log(this.name)
}

const Cat = function(name) {
  // 复制 Animal 的属性到 Cat 上
  Animal.call(this, name)
}
// 把 Cat 的原型指向 Animal
Cat.prototype = new Animal()

const a = new Animal('animal')
a.getName()

const cat = new Cat('cat')
cat.getName()

```

核心步骤：

1. `Animal.call(this, name)` 将父函数的属性指向当前函数
2. `Cat.prototype = Animal.prototype` 将当前函数的原型的 `__proto__` 指向父函数，形成新的原型链

这样就会形成，有自己一套的构造函数属性，原型上的属性方法进行共用

但是上面一共调用了两次 `Animal` ，会在实例和原型上都创建了构造函数的属性，这样会造成不必要的消耗。



#### 原型式继承

```javascript
function object(o) {
  function f() {}
  f.prototype = o
  return new f()
}

const Cat = {
  name: 'cat',
  getName: () => {console.log('get')}
}

const a = object(Cat)
// or
// const a = Object.create(Cat)
console.log(a, a.getName)
```

用一个临时的构造函数，把它的 `prototype` 指向传入的对象上，再返回一个实例。

这样的做法，会使所有的实例的引用类型的属性变成共享的。

#### 寄生式继承

```javascript
function object(o) {
  function f() {}
  f.prototype = o
  return new f()
}

function createAnother(original, name) {
  var clone = object(original)
  clone.name = name
  clone.sayHi = function () {
    console.log(this.name)
  }
  return clone
}

const Cat = {
  name: 'cat',
  getName: () => {console.log('get')}
}

const a = createAnother(Cat, 'cat1')
const b = createAnother(Cat, 'cat2')
console.log(a.getName === b.getName, a.sayHi === b.sayHi) // true, false
a.sayHi()
```

与原型式继承类似，只不过多了一个步骤，在返回对象前，添加一些属性方法，达到增强的效果。

#### 寄生式组合继承

```javascript
function object(o) {
  function f() {}
  f.prototype = o
  return new f()
}

function inheritPrototype(subType, superType) {
  var prototype = object(superType.prototype)
  prototype.constructor = subType
  subType.prototype = prototype
}

function Animal(name) {
  this.name = name
}
Animal.prototype.getName = function() {
  console.log(this.name)
}

function Cat(name, type) {
  Animal.call(this, name)
  this.type = type
}

inheritPrototype(Cat, Animal)

Cat.prototype.say = function() {
  console.log(this.type)
}

const a = new Cat('hah', 'black')
console.log(a)
```

红皮书说这种是实现继承的最优方法。首先父类的属性还是使用构造函数的方法和继承。而原型的方法，先用父类的 `prototype` 生成一个实例，再将子类的 `prototype` 指向这个实例。



## 原型 / 构造函数 的验证方法

### 验证实例的构造函数

```javascript
function Cat() {}
const black_cat = new Cat()

black_cat.constructor === Cat // true
black_cat instanceof Cat // true
balck_cat instanceof Object // true

```

如果是 `Cat` 是继承于 `Animal` 的话， `instanceof` 也会判断为 `true`

#### 获取原型

```javascript
funciton Cat() {}
const cat = new Cat()
console.log(Object.getPrototypeOf(cat)) // 获取 __proto__ 里面的值

```

#### 获取非原型属性

```javascript
funciton Cat() {
  this.a = 1
}
Cat.prototype.b = 2
const cat = new Cat()

console.log(cat.hasOwnProperty('a')) // true
console.log(cat.hasOwnProperty('b')) // false

```

`hasOwnProperty` 方法可以判断该属性是否存在自身属性上



## 回答

> 面试经典题。。。尝试通过自己的口述，把问题表达清晰明白

###什么是原型

每个实例对象，都有一个属性（__proto__) 指向实例的原型对象（prototype），而 prototype 也有自己的原型对象，__proto__ 指向它的原型对象，直到 Object 为止。而 Object 的原型对象指向的是 null。

原型对象之间的连接，构成原型链



### instanceof 

**`instanceof`** **运算符**用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上

```javascript
function test() {}

const t1 = new test()

function test2() {}

console.log(t1 instanceof test) // true
console.log(t1 instanceof test2) // false
console.log(t1 instanceof Object) // true

```

自行实现：

```javascript
function new_instance_of(leftVaule, rightVaule) { 
    let rightProto = rightVaule.prototype; // 取右表达式的 prototype 值
    leftVaule = leftVaule.__proto__; // 取左表达式的__proto__值
    while (true) {
    	if (leftVaule === null) {
            return false;	
        }
        if (leftVaule === rightProto) {
            return true;	
        } 
        leftVaule = leftVaule.__proto__ 
    }
}
```

具体原理，就是沿着左边对象的 \_\_proto\_\_ 属性，向上一层层寻找，知道找到与右边对象的 prototype 值一样的



### 如何实现继承？

js 实现继承，都是要通过 **构造函数** 或 **原型** 从而实现继承的。常见的继承像：组合继承、原型式继承、寄生继承、寄生组合式继承。

组合继承的话，通过使用  `call` 或 `apply` 方法，去获取父类的属性，再把子类的 `prototype` 连到父类的实例上，因为 `prototype` 的属性是共享的，这样就不用每次生成实例都要创建一次属性或方法，直接继承了父类原型上的属性。但使用组合继承会调用了父类，会调用父类两次，造成不必要的消耗。

而寄生继承，使用一个临时的构造函数，把它的原型连到父类身上，再给到子类，子类在它的基础上可以添加自己的属性，达到增强的效果，但是无法实现函数复用。

寄生组合继承，还是先通过 `call` 或 `apply` 获取父类的属性，再使用临时的构造函数，把它的 `prototype` 连到父类上，生成一个实例，把子类的 `protoype` 连到该实例的 `prototype` 上，从而继承父类的方法。

#### 疑问1：组合继承为什么子类的 prototype 要指向新的父类实例

因为继承，子类不能污染到父类，只能用自身的方法覆盖。如果子类的 `prototype` 指向父类，或父类的 `prototype` 上，修改子类的 `prototype` 时，就会污染到父类了。

#### 疑问2：为什么要使用一个临时的构造函数？

如果直接把 `prototype` 挂到父类上，从而继承父类的方法或属性，这样当该类在原型上有相同的方法或属性，就会直接改变父类的，而不是覆盖父类的。使用了临时的构造函数，把它的 `prototype` 连接到父类上后，

#### 疑问3：Class 和原型有什么区别？

传统的 Class ，继承是通过复制操作的，而 JavaScript 的继承是通过委托关联的。

比方说，子类继承了父类，当修改子类的某个对象里的属性时，传统的 class，因为是复制操作，所以每个子类都不会收到影响；而 JavaScript 的继承是通过原型的，这个对象是挂在 `prototype` 时，其他的子类都会收到影响。因为访问、修改该值时，在当前对象的属性找不到时，就会按照原型链的规则，顺着原型一级级往上找。

#### 疑问4：ES6 的 Class 是传统的 Class 吗？

Es6 的 Class 只不过是基于原型的语法糖

优点：

1. 语法更加简洁，大部分情况不再需要些 prototype
2. 继承操作只需使用关键字 extend 就行了
3. 可以通过 super 实现多态

缺点：

1. 它还是通过 prototype 去实现而已，当修改父类的某个方法时，子类还是会收到影响
2. 无法定义原型上的属性，语法只支持定义方法。如果要定义原型上的属性，只能显式地在 prototype 上添加
3. 