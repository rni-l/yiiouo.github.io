---
title: TypeScript
date: 2017-06-27 00:00:00
tags: 
categories: ["记录"]
---
# TypeScript 使用

>   记录一些主要概念，方便查看

## 什么是 typeScript

*   TypeScript的核心原则之一是对值所具有的*结构*进行类型检查

## 语法

### 基本类型

```typescript
// 布尔值
const a: boolean = false
// 数字
const b: number = 1
// 字符串
const c: string = '1'
// 数组
const d: Array<元素类型> = [1, 2, 3]
const d_1: Array<number> = [1, 2]
const d_2: number[] = [1, 2]
// 枚举
enum Color { Red, Green, Yellow = '2' }
const e: Color = Color.Red
// Any
let f: any = 1
f = '1' // success
// Void
// 用于定义函数没有返回值
// Null 和 Undefined
const g: undefined = undefined
const h: null = null
// Never
// never 表示不存在的值的类型，用于那些无法达到终点的函数
// Object
declare function create(o: object | null): void
create({ prop: 0 })
// 类型断言
let someValue: any = "this is a string"
// 有两种写法，第一种不兼容 jsx
let strLength: number = (<string>someValue).length
let strLength: number = (someValue as string).length
```



### 接口

> 在TypeScript里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

*   用来描述 Object、函数、类等多种的类型
*   简单来讲，就是用来描述一种结构的语法

#### 对象类型

如果一个 object 类型的参数，不确定部分参数的话：

```typescript
// 使用 [propName: xx]: xx 的方式
interface SquareConfig {
    color?: string
    width?: number
    bg: string
    readonly id: string // readonly 表示只读，不能改
    [propName: string]: any // 索引签名，可以额外添加任意数量的其他属性值
}
const test: SquareConfig = {
  color: 'red',
  bg: 'black',
  id: 'sdf'
}
```

#### 函数类型

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

let mySearch: SearchFunc;
mySearch = function(ddd: string, ttt: string) {
  let result = ddd.search(ttt);
  return result > -1;
}
```

定义了一个函数类型的，声明一个对象，指向对应的接口。参数名不会要求是一样，会按照定义的顺序进行检测。

#### 索引类型

```typescript
interface StringArray {
  [index: number]: string; // 定义了数字索引，规定传的参数是字符串类型
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

上面也有一个是字符串索引的例子

#### 类类型

```typescript
// 使用接口描述一个类
interface ClockInterface {
    currentTime: Date; // 描述一个属性
    setTime(d: Date); // 描述一个方法
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

#### 继承接口

```typescript
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

#### 混合类型

...

#### 接口继承类

....

### 类

基本的使用，和 es6 的 class 一样

```typescript
class Greeter {
    // 定义一个属性，可以在定义时赋值
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

let greeter = new Greeter("world")
```

继承也和 ES6 的 Class 类似

#### 公共，私有与受保护的修饰符

public / private / protected 

默认使用 `public` 修饰符

使用 `private` 不能在声明它的类外使用

```typescript
class Animal {
  private name: string
  readonly id: string = 'sdf' // 不能修改,必须在声明时或构造函数里被初始化
  constructor(theName: string) { this.name = theName; }
}
const obj = new Animal()
obj.name // throw error
```

使用 `protected` 的属性，在它的派生类可以访问，其他情况和 `private` 一致

而构造函数也可以被标记成 `protected` 。这意味着这个类不能在包含它的类外被实例化，但是能被继承

#### 存取器

```typescript
class Animal {
  protected _name: string;
  constructor(theName: string) { this.name = theName; }
  get name(): string {
    console.log('get')
    return this._name
  }
  set name(value) {
    console.log('set')
    this._name = value
  }
}

let animal = new Animal("Goat");

console.log(animal.name)
animal.name = 'set'
console.log(animal.name)
```

和 `defineProperty` 类似的作用

#### 静态类型

```typescript
class Animal {
  static a = 123
}

console.log(Animal.a)
```

可以通过类对象访问的属性。静态方法也是这样。

#### 抽象类

>   抽象类做为其它派生类的基类使用。 它们一般不会直接被实例化。 不同于接口，抽象类可以包含成员的实现细节。 `abstract`关键字是用于定义抽象类和在抽象类内部定义抽象方法

抽象类，是无法实例化的。

```typescript
abstract class Parent {
  private name: string
  constructor(name) {
    this.name = name
  }
  t1(): void {
    console.log('t1')
  }

  abstract t2(): void
}

console.log(new Parent('123')) // [ts] 无法创建抽象类的实例。
```

如果对某个类，指向抽象类为接口

```typescript
abstract class Parent {
  private name: string
  constructor(name) {
    this.name = name
  }
  t1(): void {
    console.log('t1')
  }

  abstract t2(): void
}

class Child extends Parent {
  constructor(name) {
    super(name)
  }
  t2(): void {
    console.log('t2')
  }
  t3(): void {
    console.log('t3')
  }
}

let a: Parent // 注意，如果没有这步骤，下面不会报错
a = new Child('123')
console.log(a)
a.t1()
a.t2()
a.t3() // Property 't3' does not exist on type 'Parent'.

```

### 函数

```typescript
// 简单地定义
function add(x: number, y: number): number {
    return x + y
}
let myAdd = function(x: number, y: number): number { return x + y }
```

可选参数、默认参数，不再累赘

#### 剩余参数

```typescript
// 定义一个名称，加上 ... ，用来说明剩余的参数
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie")
```

#### this 问题

this 在回调函数中使用会报错，需要声明 this

```typescript
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

#### 重载

```typescript
let suits = ["hearts", "spades", "clubs", "diamonds"]

// 根据不同类型的参数，返回不同类型的值
function pickCard(x: {suit: string; card: number; }[]): number
function pickCard(x: number): {suit: string; card: number; }
function pickCard(x): any {
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    } else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }]
let pickedCard1 = myDeck[pickCard(myDeck)]
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit)

let pickedCard2 = pickCard(15)
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit)
```

### 泛型

泛型可以统一变量的类型，我觉得相当于一个变量的作用

函数或类里面，变量、属性使用了该泛型，在调用或实例化的时候定义了是什么类型，则引用了该泛型的变量、属性就是什么类型

```typescript
// 描述了返回的值的类型，和传入的值的类型一致
// 传 number 就返回 number，传 string 就返回 string
function identity<T>(arg: T): T {
    return arg;
}
```

当使用泛型时，定义泛型变量，而这个变量的类型就是任意的

#### 类

```typescript
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };

// 定义了 string 类型
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

实例化类的时候，传入的类型，就是当前泛型的类型，类里面定义的所有该泛型的类型，都变成了传入的类型

很方便地统一类里面的属性的类型

#### 接口

```typescript
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

与类类似

### 枚举

枚举和数组、对象类似，可通过 key 或 value 还有索引直接获取值

```typescript
enum Direction {
    Up, // 0
    Down, // 1
    Left, // ...
    Right,
}
enum Direction1 {
    Up = 1,
    Down, // 2
    Left, // ...
    Right
}
// 字符串枚
enum Direction2 {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
// 获取值
interface Circle {
    e: Direction1.Up // 2
    d: Direction2.Up // UP
    f: Direction1[2] // Down
}
```

生成的代码中，枚举类型被编译成一个对象，它包含了正向映射（ `name` -> `value`）和反向映射（ `value` ->`name`。而字符串枚举不会生成反向映射

### 高级类型

#### 交叉类型

#### 联合类型

#### 类型别名

```typescript
// 使用 type 关键字创建的变量，是定义了一种新的类型
type testName = string
let v: testName = 'sdf'
console.log(v)
v = 1 // Type '1' is not assignable to type 'string'. 
```

接口和类型别名的区别：

1. 接口创建了一个新的名字，可以在其它任何地方使用。 类型别名并不创建新名字
2. 另一个重要区别是类型别名不能被 `extends`和 `implements`（自己也不能 `extends`和 `implements`其它类型）
3. 如果你无法通过接口来描述一个类型并且需要使用联合类型或元组类型，这时通常会使用类型别名

#### 字符类型、数字字面量类型

```typescript
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

可以定义固定的值

### 模块

任何声明都能通过 export 导出

```typescript
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
export const numberRegexp = /^[0-9]+$/;
export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export type hah = string
```

#### 引用非 TypeScript 的库

通过定义 .d.ts 文件。比如我要引入一个叫 `EventBus` 的库，而它是由 JavaScript 编写的，我们则需要在 .d.ts 文件添加相对应的模块声明

```typescript
declare module 'eventbus' {
  // 里面有一个叫 `emit` 的方法
  export function emit(): void
}
  
// 使用
/// <reference path="index.d.ts"/>
import { emit } from 'eventbus'
// ...
```

### 命名空间

```typescript
namespace Test {
  export interface Hah {}
  export type a = string
}
  
// use
const hah: Test.a = 'sdf'
```

```typescript
// 别名，减少代码量
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // Same as "new Shapes.Polygons.Square()"
```

模块和命名空间的区别：

1. 命名空间位于全局的命名空间下的对象，模块与命名空间一样，可以包含代码和声明，但是模块可以声明它的依赖



### 项目配置文件





## 优劣

使用 TypeScript 相对于 JavaScript 有什么优势、区别？

区别：

1. TypeScript 是 JavaScript 的超集，增加了静态类型、类、接口、模块等多种注解，配合编译器，会很方便地显示信息
2. TypeScript 会对静态类型进行检查，在运行前就把错误信息提示出来

优势

1. 静态类型检查，会相对提高代码的质量，提示更加清晰，方便维护
2. 团队协作更友好
3. 配合编辑器，开发时的提示更好
4. 与 ES6、ES7 等没有兼容问题

劣势：

1. 学习成本更高
2. 开发时间可能会相对慢点，需要写大量的注释（静态类型、类说明、接口）



## Vue 项目改造 TypeScript 后的总结

### 参考链接

1. [TypeScript 文档](https://www.tslang.cn/docs/home.html)
2. [Vue TypeScript 文档](https://cn.vuejs.org/v2/guide/typescript.html#ad)
3. [vue-property-decorator 文档](https://github.com/kaorun343/vue-property-decorator/)
4. [vue-class-component 文档](https://github.com/vuejs/vue-class-component)
5. [vuex-class 文档](https://github.com/ktsn/vuex-class/)

### 改造过程

1. 以 vue-cli@v3.x.x 版本构建的项目进行改造
2. 安装依赖：
   1. npm i vue-class-component vue-property-decorator vuex-class -S
   2. npm i @vue/eslint-config-typescript @vue/cli-plugin-typescript typescript -D
3. 添加配置文件和声明文件，具体内容去事例项目复制
   1. 添加 tsconfig.json 文件在项目根目录，需要修改配置，详细看 TypeScript 文档
   2. 添加 types 文件夹在项目根目录
4. 将 ./src/main.js 改成 ./main.ts 文件，而且里面的 js 语法也要转换成 ts 的
5. 修改 ./.eslintrc.js 文件，添加 TypeScript 的语法检查，具体内容去事例项目复制

### 使用过程

使用了 `TypeScript` 最大的感受就是调用函数时，知道入参类型，有多少个，函数的返回值等，知道对象有什么类型的字段，想看具体的声明是怎样时，vscode 可以直接跳到那个指定文件。接口、类型等都可以快速查询

因为定义了类型，它就会限制你的使用，比如在 `js` 里面：

```javascript
const a = new Date()
a.sdf(); // 不会立即提示你，没有这个方法，只有运行时才知道
```

而在 `ts` 里面，编辑器或 ide 就会立即提示你，不能这样使用，因为你没在 `Date` 对象原型中，定义了 `sdf` 方法

因为有了这层限制，所以我所写的 `js` 处处都会有限制，判断的逻辑也多了，一些类型、语法、判空的错误确实有所减低。前提是你的代码都有做了明确的定义，而没有使用 `any`

像公共方法、接口这些都是必须做类型定义，所以写接口方法的时候，多了一堆 `interface` 定义，而赋值接口返回的字段，也多了一层初始化定义，比如：

```typescript
export interface CustomerInfoDetailsParams {
  refId: RefId
}

export interface CustomerInfoDetails {
  ageGroup: string; // 年龄段
  career: string; // 职业
  childStatus: string; // 子女状况
}

/**
 * 获取客户明细信息
 */
export const apiGetCustomerInfoDetails = (params: CustomerInfoDetailsParams, opts?: RequestOpts) => {
  return request<CustomerInfoDetails>({
    url: `/api/getCustomerInfoDetails`,
    ...opts,
    params
  });
};

// 这是定义了一个接口方法，明确了入参和返回值，然后在调用的代码，要这样写：
// 声明是什么类型
let data: CustomerInfoDetails = {
  ageGroup: '',
  carrer: '',
  childStatus: ''
}
// 而 data 不能这样定义：
// let data: CustomerInfoDetails = {}

async getData() {
  const { isSuccess, result } = await apiGetCustomerInfoDetails({ refId: 'sdf' })
  if (isSuccess) {
    data = result; // result 的类型是 CustomerInfoDetails
  }
}

```

代码量会激增，虽然接口的定义可以直接复制后端 `swagger` 文档的定义

### Vue 改造为 ts

改造为 `ts` 的时候，只有 `js` 部分会变，而 `HTML` 是没影响的，这就有个问题，调用组件的时候，`prop` 和 `event` 的传入是没提示的，这点比较尴尬

`js` 部分的改造，需要引入三个依赖：`vue-property-decorator`、`vue-class-component` 和 `vuex-class`，代码从原来的  `Object` 结构改为  `Class` 结构，配上依赖提供的装饰器使用，我使用的版本，已经可以覆盖大部分地方的功能，将其转换为 `ts`：`Mixin`, `Ref` 等…..

`jsx` 和 `render` 模式还没尝试过



## 遇到的问题和总结点

1. 用好泛型，像封装接口的时候，每个接口返回的  `response` 可能都不一样，这时候利用泛型，就可以解决问题
2. `*.d.ts` 文件，`declare` 和 `export` 不能共存 // 需要验证
3. 处理 `event`  对象时，可以自定义一个类型进行处理

```typescript
type HTMLElementEvent<T extends HTMLElement> = Event & {
  target: T;
  // probably you might want to add the currentTarget as well
  // currentTarget: T;
};

function handleInput(e: HTMLElementEvent<HTMLInputElement>) {
  this.$emit('input', e.target.value);
}
```

4. `canvas` 的 `getContext(‘2d’)` 方法，返回的值，提示可能是 `null` ，但真实情况不会出现，可以使用 `!` 解决

```typescript
const canvas = document.createElement('canvas')
const ctx = canvas.getContext('2d')!
```

5. 小心使用 `as`，当我们开发比编辑器“聪明”的时候，知道这个值一定是某个类型，就可以用 `as`

```typescript
type Abc = string | number

let a: Abc = '1'

console.log((a as string).substring(0))

```

6. *// @ts-ignore* 可以忽略某行 `ts` 的错误提醒

7. 枚举类型好不好用？

8. 自动推导成指定类型

9. ```typescript
   const obj = {
     type: 'dog' // 这样 ts 会自动推导成 string
   }
   // 如果我想要推导成 'dog'
   const obj = {
     type: 'dog' as 'dog'
   }
   ```

10. 善用 `Partial`, `ReturnType`, `Required`, `Readonly` , `Record` 等等这些高级泛型，可以节省很多工作量

11. 对象字面量的惰性初始化

    1. ```typescript
       interface Foo {
         bar: number;
         bas: string;
       }
       
       let foo = {} as Foo;
       foo.bar = 123;
       foo.bas = 'Hello World';
       
       // 然后我们尝试这样做：
       foo.bas = 'Hello Stranger';
       ```

12. 减少使用 `export default` ，[链接](https://jkchao.github.io/typescript-book-chinese/tips/avoidExportDefault.html)

13. 比如添加修改的接口，它们的接口定义都是类似的，只不过添加没有 id，这个要怎么重用？

