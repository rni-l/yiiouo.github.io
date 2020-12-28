---
title: vue 2.x 源码阅读总结
date: 2020-01-10
tags: [“js", “vue"]
categories: ["记录"]
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

















