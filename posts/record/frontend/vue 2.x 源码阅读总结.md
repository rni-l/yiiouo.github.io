---
title: vue 2.x 源码阅读总结
date: 2020-01-10
tags: [“js", “vue"]
categories: ["记录"]
draft: true
---

## 参考文章

1. [muwoo](https://github.com/muwoo/blogs)
2. [染陌](https://github.com/answershuto/learnVue)



## 开始

[vue 生命周期](http://md.rni-l.com/md/vuelifecycle.png)



### 初始化

#### initGlobalAPI - core/index.js

初始化配置，内部工具，全局方法(set, delete, nextTick)，添加公共组件(keep-alive)

先后执行 4 个初始化方法：initUse, initMixin, initExtend, initAssetRegisters

##### initUse

`initUse` 方法，就是为 Vue 添加 `.use` 方法

```javascript
Vue.use = function (plugin: Function | Object) {
  const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
  if (installedPlugins.indexOf(plugin) > -1) {
    return this
  }

  // additional parameters
  const args = toArray(arguments, 1)
  args.unshift(this)
  if (typeof plugin.install === 'function') {
    plugin.install.apply(plugin, args)
  } else if (typeof plugin === 'function') {
    plugin.apply(null, args)
  }
  installedPlugins.push(plugin)
  return this
}
```

每当我们执行 Vue.use 就是执行上面的内容

首先查找是否有已经注册过的组件 ->

然后判断这个对象是否是函数 || 它的 .install 属性是否是函数 ->

是的话缓存起来 ->

返回 this，方便链式操作

##### initMixin

```javascript
Vue.mixin = function (mixin: Object) {
  this.options = mergeOptions(this.options, mixin)
  return this
}
```

就是合并 option 操作~

##### initExtend(TODO)

通过 extend 方法继承的，每个 Class 都会有一个唯一的 cid；会重复对象执行了 extend，会从缓存里返回出去

然后就会进行原型继承，并手动初始化子类的 options, extend, mixin, use 等等属性

##### initAssetRegisters

```javascript
ASSET_TYPES.forEach(type => {
  Vue[type] = function (
    id: string,
    definition: Function | Object
  ): Function | Object | void {
    if (!definition) {
      return this.options[type + 's'][id]
    } else {
      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && type === 'component') {
        validateComponentName(id)
      }
      if (type === 'component' && isPlainObject(definition)) {
        definition.name = definition.name || id
        definition = this.options._base.extend(definition)
      }
      if (type === 'directive' && typeof definition === 'function') {
        definition = { bind: definition, update: definition }
      }
      this.options[type + 's'][id] = definition
      return definition
    }
  }
})
```

`ASSET_TYPES` 就是 filter, component, directive；这里据说初始化它的三个静态方法

#### initMixin - instance/index.js

##### _init

在 Vue.prototype 添加 _init 方法，接收一个参数：options

每个 Vue 实例都会有一个 _uid 记录着

如果 _options 有 _isComponent 的标识：

执行 initInternalComponent 方法

如果没有，则将 vm.constructor 和 options 进行合并，生成 vm.$options 对象

然后继续初始化操作和执行生命钩子

```
initLifecycle
initEvents
initRender
callHook
initInjections
initState
initProvide
callHook
```

最后再执行 $mount

###### initInternalComponent(TODO)



##### resolveConstructorOptions

根据当前组件对象，一层层递归它的父类，去处理、剔除重复属性的 options，最终还是志贤 `mergeOptons`

##### mergeOptions

执行下面三个方法，对 options 里面的初始化

然后再处理父类和子类的 options，根据 start 进行合并，赋值到 options

######normalizeProps

props 支持数组和对象格式，这里主要做了兼容处理而已

里面有个方法，将 ‘-’ 去除，并转为驼峰格式

处理成一个 Object 类型，并赋值到 options 上

######normalizeInject

和 normalizeProps 大同小异

######normalizeDirectives

和 normalizeProps 大同小异

###### starts(TODO)

源码的注解：

> Option overwriting strategies are functions that handle how to merge a parent option value and a child option value into the final value.

首先创建一个空对象：`const strats = config.optionMergeStrategies -> Object.create(null)`

```javascript
strats.data = function (
  parentVal: any,
  childVal: any,
  vm?: Component
): ?Function {
  if (!vm) {
    if (childVal && typeof childVal !== 'function') {
      process.env.NODE_ENV !== 'production' && warn(
        'The "data" option should be a function ' +
        'that returns a per-instance value in component ' +
        'definitions.',
        vm
      )

      return parentVal
    }
    return mergeDataOrFn(parentVal, childVal)
  }

  return mergeDataOrFn(parentVal, childVal, vm)
}
```

定义一个 data 的方法，对父类的 val 和子类的 val 进行合并

添加一些 compute, props, components, inject, filters….. 等必有的属性

它的作用，**muwoo** 总结是这样的：

>Vue提供了一个strats对象，其本身就是一个hook,如果strats有提供特殊的逻辑，就走strats,否则走默认merge逻辑



##### initLifecycle

定义一些字段，和生命周期的状态值

##### initEvents(TODO)

初始化事件，添加监听器

##### initRender

给 vm 添加属性： 

* $slots
* $scopedSlots
* _c
* $createElement
* $attrs
* $listeners

resolveSlots, createElement, defineReactive 后面再讲





## 核心模块

### Observer

先上源码：

```javascript
/**
 * Observer class that is attached to each observed
 * object. Once attached, the observer converts the target
 * object's property keys into getter/setters that
 * collect dependencies and dispatch updates.
 */
export class Observer {
  value: any;
  dep: Dep;
  vmCount: number; // number of vms that has this object as root $data

  constructor (value: any) {
    this.value = value
    this.dep = new Dep()
    this.vmCount = 0
    def(value, '__ob__', this)
    if (Array.isArray(value)) {
      const augment = hasProto
        ? protoAugment
        : copyAugment
      augment(value, arrayMethods, arrayKeys)
      this.observeArray(value)
    } else {
      this.walk(value)
    }
  }

  /**
   * Walk through each property and convert them into
   * getter/setters. This method should only be called when
   * value type is Object.
   */
  walk (obj: Object) {
    const keys = Object.keys(obj)
    for (let i = 0; i < keys.length; i++) {
      defineReactive(obj, keys[i])
    }
  }

  /**
   * Observe a list of Array items.
   */
  observeArray (items: Array<any>) {
    for (let i = 0, l = items.length; i < l; i++) {
      observe(items[i])
    }
  }

```

初始化的时候，对属性判断，根据是否数组类型进行额外操作，每个子项执行 `observe` ，否则执行 `defineReactive`

### observe

### defineReactive

源码经过删减：

```javascript
/**
 * Define a reactive property on an Object.
 */
export function defineReactive (
  obj: Object,
  key: string,
  val: any,
  customSetter?: ?Function,
  shallow?: boolean
) {
  const dep = new Dep()

  const property = Object.getOwnPropertyDescriptor(obj, key)

  // cater for pre-defined getter/setters
  const getter = property && property.get
  const setter = property && property.set

  let childOb = !shallow && observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter () {
      const value = getter ? getter.call(obj) : val
      if (Dep.target) {
        dep.depend()
        if (childOb) {
          childOb.dep.depend()
          if (Array.isArray(value)) {
            dependArray(value)
          }
        }
      }
      return value
    },
    set: function reactiveSetter (newVal) {
      const value = getter ? getter.call(obj) : val
      if (newVal === value || (newVal !== newVal && value !== value)) {
        return
      }
      if (process.env.NODE_ENV !== 'production' && customSetter) {
        customSetter()
      }
      if (setter) {
        setter.call(obj, newVal)
      } else {
        val = newVal
      }
      childOb = !shallow && observe(newVal)
      dep.notify()
    }
  })
}
```

主要是根据该属性的原型配置，再使用 `defineProperty`，对该属性的 `get`, `set` 进行拦截

`get` 主要做了 `dep.depend()`

`set` 主要做了 `dep.notify()`

### Dep

这是 Dep 的源码

```javascript
let uid = 0

/**
 * A dep is an observable that can have multiple
 * directives subscribing to it.
 */
export default class Dep {
  static target: ?Watcher;
  id: number;
  subs: Array<Watcher>;

  constructor () {
    this.id = uid++
    this.subs = []
  }

  addSub (sub: Watcher) {
    this.subs.push(sub)
  }

  removeSub (sub: Watcher) {
    remove(this.subs, sub)
  }

  depend () {
    if (Dep.target) {
      Dep.target.addDep(this)
    }
  }

  notify () {
    // stabilize the subscriber list first
    const subs = this.subs.slice()
    if (process.env.NODE_ENV !== 'production' && !config.async) {
      // subs aren't sorted in scheduler if not running async
      // we need to sort them now to make sure they fire in correct
      // order
      subs.sort((a, b) => a.id - b.id)
    }
    for (let i = 0, l = subs.length; i < l; i++) {
      subs[i].update()
    }
  }
}

// the current target watcher being evaluated.
// this is globally unique because there could be only one
// watcher being evaluated at any time.
Dep.target = null
const targetStack = []

export function pushTarget (_target: ?Watcher) {
  if (Dep.target) targetStack.push(Dep.target)
  Dep.target = _target
}

export function popTarget () {
  Dep.target = targetStack.pop()
}

```



### Watcher

```javascript
/**
 * A watcher parses an expression, collects dependencies,
 * and fires callback when the expression value changes.
 * This is used for both the $watch() api and directives.
 */
export default class Watcher {
  vm: Component;
  expression: string;
  cb: Function;
  id: number;
  deep: boolean;
  user: boolean;
  lazy: boolean;
  sync: boolean;
  dirty: boolean;
  active: boolean;
  deps: Array<Dep>;
  newDeps: Array<Dep>;
  depIds: SimpleSet;
  newDepIds: SimpleSet;
  before: ?Function;
  getter: Function;
  value: any;

  constructor (
    vm: Component,
    expOrFn: string | Function,
    cb: Function,
    options?: ?Object,
    isRenderWatcher?: boolean
  ) {
    // ...
    this.value = this.lazy
      ? undefined
      : this.get()
  }
  // ...
}
```



### initData

核心代码：

```javascript
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
```

在初始化 data 的时候，会从 data, props, methods 组成一个集合；如果有重复就会报错

`      proxy(vm, `_data`, key)` 这里有个代理的操作

proxy 源码：

```javascript
export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

data 的值是挂在 `vm._data` 上面的，但我们平时是 `this.xxx` 就可以获取到，原理就是 proxy 做了一层类似转发的功能，当我们访问 `this.xxx` 是，其实就是访问到 `this._data.xxx`

props 的值也是一样

而 methods 的值，是直接挂在 `this` 上





## 常问问题



#### Vue 原理

参考：

1. [Vue 的双向绑定](https://github.com/Advanced-Frontend/Daily-Interview-Question/issues/34)
2. 

Vue 的核心是基于 `Object.defineProperty` 和订阅发布模式，从而实现所谓的 MVVM 模式。当 Vue 初始化时，会通过 `observe` 方法给对应的属性配置 `defineProperty` 的 `get` 和 `set` 拦截器，利用 `get` 拦截器收集依赖，当属性被修改时，触发 `set` 拦截器，根据当前属性收集到的依赖，它们进行 `update` 操作，从而更新 DOM。

##### 这个所谓的依赖指的是什么？

首先我们要理清哪个是发布者哪个是订阅者

在使用 `defineProerty` 配置属性的时候，里面的 `set` 拦截器相当于发布者，它通过 `Dep` 去管理订阅者，也就是 `Watcher`。

在源码里，这个依赖就是 `Watcher` 类，它相当于 MVVM 模式中的 `VM`，是用于视图和控制层之间通信的。

我们在目标上使用了某个属性时，就会开始收集依赖。这时候会触发属性的 `get` 拦截器，让当前的 `Watcher` 缓存当前属性的 `Dep`，并让 `Dep` 收集依赖 `Watcher`。

该属性收集到依赖，就是为了后面更新的时候使用。当我们修改属性时的时候，就会触发 `set` 拦截器，让当前属性的 `Dep` 通知收集到的依赖做相关操作。

##### get 拦截器是如何收集依赖？

Vue 中有一个叫 `Dep` 的类，专门来收集管理 `Watcher`，而这个 `Dep` 类有一个静态属性，控制当前是哪个 `Watcher`。Vue 这里有个限制，同一时间内只能有一个全局的 `Watcher` 进行处理，不然会乱套了，比如可能会出现 a 组件的 `Watcher` 来处理 b 组件的更新。

每个属性在使用 `defineProperty` 配置的时候，都会初始化一个 `Dep` 的实例来收集改属性的依赖。比如我们再模板上使用了一个属性，会设置当前组件的 `Watcher` 为 `Dep` 的静态属性，然后会触发 `get` 拦截器，就当前的依赖 `Watcher`。

#### watcher 是如何更新？

这里面有个 `queueWatch`，队列的功能在里面….



#### 为什么使用 vue

简单易上手，官方文档、教程很友好，很快就可以开发项目。而且官方有一套完整的生态，可以很快得开发一个简单的项目。而 vue 的单文件组件，我觉得开发起来更方便，不需要不断地切换文件

vue-cli 3.x 版本，可以快速建一个 TypeScript 项目，而且 vue3.0 版本是用 TypeScript 重构的，未来 vue 对 TypeScript 会更加得友好

而且 vue 在国内的生态非常得热，对于项目出现问题的解决办法，也会有很多



#### vue 的生命周期

有 beforeCreate, created, beforeMount, mounted, update, beforeDestory, destoryed；如果该组件是在 keep-alive 组件内的，还会有 activited 和 deactivited；主要过程就是初始化 option 、各种内置方法和组件数据，编译模板、挂载 dom，到渲染、更新、卸载等



#### v-model 原理

原理就是一个语法糖，它会帮你创建一个回调事件，当表单控件的数据更改后，触发这个回调事件去修改绑定的属性的值



#### 组件通信的方式

父子：

1. 通过传入 props，给到子组件
2. 通过 refs 直接获取子组件的属性、或调用方法
3. provide/inject，这个甚至可以隔代，让它的所有子组件都有生效

子父：

1. 通过 emit 方法，通知父组件
2. 获取 this.$parent ，得到父组件

兄弟、或无关联组件

1. eventBus
2. vuex



#### computed 和 watch 区别

computed：

1. 可以对多个属性进行监控，某个属性发生变化，就会调用定义的函数，相当于声明一个新的属性
2. 而且 computed 会基于里面的属性进行缓存，只有相关的属性发生变化才会重新求值。所以当里面的属性没有变化时，重复的获取 computed 的值，只是从缓存里拿，而没有再次执行里面的代码

watch：

1. 对某个属性近监控，当这个属性发生变化，就会调用定义的函数
2. 只有属性变化才会触发这个回调，更多地是当属性变化从而处理一些业务逻辑



#### nextTick 原理

`Vue` 的 `nextTick` 原理，其实就是利用了 `JS` 的事件循环机制从而实现的。

这里要说明下  `JS` 的事件循环机制…..（见面试问题）

在 Vue 初始化的时候，首先 `nextTick` 会根据当前宿主环境，比如浏览器或者 Node，判断哪些全局方法可以使用，在浏览器会用 `Promise` 或者 `setTimeout` 实现。

因为 `nextTick` 要的效果是异步的，通常在浏览器 `nextTick` 会使用 `Promise` 去实现异步，也就是事件循环机制里面的 `microtask`，利用这个特性，在下一次同步任务调用前，这期间可能会有多个 `nextTick` 被调用，会有一个数组专门收集传入的回调函数。这里面还会有一个锁，用来防止回调函数被重复调用。当 `JS` 线程的同步任务都完成后，就会执行这个数组里面所有的回调函数，然后清空数组。



#### $set 方法原理

首先会判断是数组还是对象，如果是数组：

则会调用 splice 进行更新

如果是对象，如果当前要修改属性存在，就直接进行修改；如果该对象不是响应式数据，直接赋值；否则对属性进行响应式处理



##### 为什么 this.xxx 可以直接获取到属性

去到 vue 2.x 源码阅读总结 proxy 栏



#### key 的作用

这里分两种情况，使用 key 会怎样，不使用会怎样。

不使用 key: 

1. 对于无状态的列表组件，不使用 key 会触发“就地复用节点”逻辑，当数据更新时，不需要频繁地销毁和创建节点，更高效
2. 因为触发“就地复用节点”逻辑，当列表组件是含有状态变化的，会导致状态变化时，渲染异常，比如多个组件渲染状态错误

使用 key:

1. 当数据更新时，会触发销毁和创建的逻辑，准确渲染组件
2. 有时候我们可以根据这个逻辑，实现组件刷新的逻辑，比如设置 key 为一个时间戳的值，将该 key 赋值到某个需要的组件，当我需要强制刷新组件的时候，修改这个 key 的值就会自动销毁并创建组件

















