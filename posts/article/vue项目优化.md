---
title: vue项目优化
date: 2018-06-28 00:00:00
tags: ["js", "vue", "优化"]
categories: ["分享"]
---

>主要说的是，我在项目中，自己遇到的一些小问题和解决方案

## 图片 base64 问题

```
// 有一个 test 的组件
<template>
  <div class='icon'></div>
</template>
<style>
  .icon {
    background: url(../assets/test.png);
  }
</style>

// 然后有三个页面，引入了 test 组件
// h1.vue, h2.vue, h3.vue 分别都引入 test 组件
<template>
  <test></test>
</template>
<script>
import test from '@/components/test.vue'
// ...
</script>

```

第一次打包文件之后，三个页面的 js 文件，都会出现相同的一个 base64 图片

![](http://oxnbdd4i9.bkt.clouddn.com/vueOptimizationpic1.jpg)

后面第二次打包使用 `import` 进行引用，或者直接在标签写死，就不会出现上面这种情况

```
import a from 'a.jpg'
<div style='backgroundImage: url(a.jpg)'></div>
```

## 如何缓存特定页面

使用 `keep-alive` 对某些页面进行缓存

```
// app.vue
<keep-alive :include="['a', 'b', 'c']">
  <router-view></router-view>
</keep-alive>
```

在 `router-view` 父级，加上一层 `keep-alive`，再使用 `include` ，告诉 vue 有哪些页面需要缓存，就行了。

其中 a, b, c 是 `.vue` 文件的 `name` 属性值

`keep-alive` 标签，支持两个属性

* include: 只有匹配的组件会被缓存
* exclude: 任何匹配的组件都不会被缓存

会被缓存的页面（其实就是一个 vue 组件），离开时，vue 会在内部缓存当前组件的状态，下次再次进入这个页面，就会显示离开时的状态

如果有些需求，需要你在特定得状态下，才需要缓存，而在其他时间，进入页面都是保持最初的状态时，我们可以使用 vue 的一个方法，去销毁该组件，达到刷新的效果

在 `activated` 或 `deactivated` 钩子函数事件中，使用 `this.$destroy()` 这个方法进行销毁组件

~~使用 keepa-alive 的好处，缓存组件，的确是很好，我在项目体验中，更重要是缓存页面离开时的状态，这个真得非常棒。~~

~~比如我在一个支付页面，有 M 种状态，然后要用户去另外一个页面去设置密码后，才能支付（特定需求），这时不能用一个遮罩去挡住，而是真正得跳转另一个页面，然后用户在另外一个页面操作完后，跳回去支付页面，此时支付页面还是维持离开时的状态，大大减少了很多工作（可以用 vuex 实现此效果，不过有点麻烦）~~

~~而且配上钩子函数和 `$destroy` 销毁组件的方法，可以处理好各种情况~~

**2018.01.15 更新**

使用 $destroy 对组件进行销毁，会有个 BUG。

有三个页面 A，B，C。B 页面是要进行缓存的。B 页面，配置了 keep-alive

A -> B

B -> C（B 到 C 要进行缓存，所以不销毁）

C -> B  (B 维持跳去 C 时的状态)

B -> A (不缓存)

A -> B

B -> C（B 到 C 要进行缓存，所以不销毁）

C -> B  (这时候就会出问题了！，会触发 activated 和 created 两个钩子函数。在 vue 的 issue 中有人提出过这个问题，尤大也说了不支持。[这里是链接](https://github.com/vuejs/vue/issues/6961))

解决方案：

我现在是把页面加进去 keep-alive 里面，然后离开时，判断下次进入要不要刷新页面。如果要刷新页面，手动把 state 更新…..超级麻烦，不优雅

**2018.01.17 更新**

有一个新的解决方案，动态的修改 include，达到更新的效果

比如，t1 组件是要缓存的

```javascript
<keep-alive :include='array'>
  <router-view></router-view>
</keep-alive>
// ...
watch: {
  $route(to) {
    if (to.name === 't1') {
      this.array = 't1'
    }
  }
}
```

每次要去到 t1 页面，对页面设置为缓存。然后在 t1 页面要跳去其他页面时候，判断是否需要缓存页面，如果不需要，修改 array 这个值。而这个 array 值，应该存在 vuex 中，这样可以更好地处理它

## 代码分割

在 `router.js` 文件里

```
{ path: '/home', component: resolve => require(['@/module/home'], resolve) },
{ path: '/h1', component: resolve => require(['@/module/h1'], resolve) },
{ path: '/h2', component: resolve => require(['@/module/h2'], resolve) }
```

这里使用了 `vue-router` 的懒加载，结合 webpack 的代码分割，将代码按页面进行分割，达到懒加载效果。进入某个页面，会按需加载 js 文件，有效地降低首屏文件大小

但这里有个问题，如果某页面的 js 文件很大，用户在跳转页面时，会出现暂时性的白屏或者无法点击的情况，影响用户体验

这里我觉得就要根据项目情况去考虑了。在某个项目中，有5个页面是没用代码分割的，而这5个页面，浏览数会比较多，并且文件的大小相对会比较大。所以取消使用了代码分隔后，当用户首次点入这几个页面，就不会出现延迟的情况

当你这个页面使用了很多 icon ，经过 base64 转换后，这个 js 文件也会相对很大，是否不应该在页面的 js 文件出现 base64 ?提前把 base64 的图片加载了？

上面的操作都是为了提高用户的体验，这个要看情况而定了

使用代码分割，文件的大小，会稍微比不用的时候大一点

## 优化依赖文件

当我们 `npm run build` 后，通常可以看到一个依赖文件，有好几百 KB，甚至 1、2MB 大，而这个文件是所有依赖文件的集合，像 `vue.js, vue-router.js, axios.js, mint-ui.js` 等

这里我们可以使用 webpack 的 `externals` 进行优化，webpack 文档是这样说 `externals`: "防止将某些 import 的包(package)打包到 bundle 中，而是在运行时(runtime)再去从外部获取这些扩展依赖"

`externals` 其实很多大神的优化贴都有写了，根据自己得体验，这里只是简单得说明

我们在 `index.html` 引入两个 js

```
<script src="//cdn.bootcss.com/vue/2.2.2/vue.min.js"></script>
<script src='//cdn.bootcss.com/mint-ui/2.2.3/index.js'></script>
```

在 `webpack.base.conf.js` 里，添加一代代码

```
externals: {
  'vue': 'Vue',
  'mint-ui': 'MINT'
}
```

这样就能无缝切换，使用 cdn 去加载比较大的依赖文件，减少打包后的 bundle 文件大小

### 打包文件大小对比

没有使用 externals 的情况， vendor.js 文件是 307KB

![](http://oxnbdd4i9.bkt.clouddn.com/vueOptimizationpic3.jpg)

使用 externals 的情况， vendor.js 文件是 107KB

![](http://oxnbdd4i9.bkt.clouddn.com/vueOptimizationpic4.jpg)

这里面是去掉了 vue 和 mint-ui 两个 js 文件，而这两个 js 文件通过外链 cdn 进行引用，两个 js 文件大约都是 31KB 左右，这里面节省了 140KB 左右的大小

这样做，虽然加多了 HTTP 请求，但是不会影响并发的数量，而且大大减少了单个文件的大小，性能更好

在 ngrok 里运行时，会运行得更快（测试过）

## 使用 v-once

在 vue 文档中，是这样提示的：对低开销的静态组件使用 v-once

使用了 v-once ，除了第一次的渲染，后面就不会再次渲染了

```
<div v-once>{{ a }}</div>

<script>
// ...
data() {
  return {
    a: 1
  }
},
created() {
  this.a = 2
}
</script>
```

最后 div 还是显示 1。这可以用于优化更新性能

## 动态加载 js

有时候，某些页面才需要的功能，需要引入特定的库去使用。比如我在一个项目要用到阿里的 OSS ，进行图片上传。前端要引入 OSS-JS-SDK 的文件才行，虽然这个 js 不大，几十 KB 而已，但只有两个页面才用到，感觉没必要。所以就使用动态加载 js，在使用某个功能时，才去加载它

```javascript
function loadJS(opts) {
  if (!(opts.obj && opts.obj !== 0)) {
      throw new Error('请传入全局对象名')
    }
    if (!opts.url) {
      throw new Error('请传入js资源地址')
    }
    if (window[opts.obj]) {
      return new Promise(resolve => {
        resolve()
      })
    }
    const oScript = document.createElement('script')
    oScript.src = opts.url
    document.body.appendChild(oScript)
    return new Promise((resolve) => {
      oScript.onload = () => {
        resolve()
      }
    })
}
```

比如我要上传图片了，就会在 change 的回调事件做事情

```javascript
<input type='file' @change='upload'>
// ...
methods: {
  upload(e) {
    loadJS({
      obj: 'OSS',
      url: 'http://xx.xxx.com',
      callback: () => {
        // js 加载完毕
      }
    })
  }
}
```

每次就调用  loadJS 这个方法。这个方法会先判断你要的全局对象是否存在，如果存在就直接执行回调。如果页面从打开到关闭都没有使用上传图片这个功能，那就不会加载这个 js

## mixin 的使用

使用 mixin，可以把相同的代码抽出来，弄出一个代码块，进行公用，不用说弄个组件。

### mixin 注意点

```javascript
const mixin = {
  data() {
    return {
      test: 123
    }
  },
  created() {
    console.log('first')
  },
  methods: {
    mixintest() {
      console.log('mixin test')
    }
  }
}

// 在 a 组件使用
export default {
  // 有一个 mixins 的属性使用，接受一个数组，说明可以放多个 mixin
  mixins: [mixin],
  data() {
    return {
      test2: 222
    }
  },
  created() {
    console.log('second', this.test)
    this.test = 'ddd'
    console.log(this.test)
  },
  mounted() {
    this.mixiintest()
  }
}
```

我们来看下上面的代码块。定义了一个 mixin 的对象，里面结构和我们平时写的组件基本一样的，然后运行了只会，代码执行的顺序是： first, second 123, ddd, mixintest

a 组件就像是继承了 mixin 里面的属性和方法，可以去读写 mixin 的属性方法。如果 mixin 和使用的组件，有属性或者方法同名的话，会取组件对象的。

### 使用

可以把一些常用的方法抽取出来


## router.beforeEach 里使用 replace

### 使用 replace

```vue
router.beforeEach(async (to, from, next) => {
  const loginStatus = store.state.test
  // 验证登录状态
  if (loginStatus) {
    if (to.path === '/login') {
      return router.replace({ name: 'home' })
    }
  } else {
    if (to.path !== '/login') {
      return router.replace({ name: 'login' })
    }
  }
  next()
})
```

上面是一个路由钩子，在进入路由前，判断是否有登录。

没登录，且不在登录页的， replace 到登录页

没登录，在登录页的， next

登录了，不在登录页的，next

登录了，在登录页，replace 到 home

#### 问题

正常走下去是没问题的。但有种情况会出现无法后退。路由是变了，但是内容没有变。

但第一次进入页面时，触发了 replace 的话，后面无论进了多少个页面，都是无法后退的。就算把 replace 换成 push 也是一样。

#### 解决方案

使用 next 方法就可以了。

```vue
next()
next({ name: 'login' })
next({ name: 'home', replace: true })
```



## 总结

以上的一些问题，在 PC 端好像影响不大，移动端的话，还是比较严重的。很多优化点都是根据实际情况入手，上面这几个，都是我在做项目时，感觉不合适而进行优化的，后面会持续补充下去~
