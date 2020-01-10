---
title: 如何编写高效的vue组件
date: 2017-06-27 00:00:00
tags: ["js", "vue"]
categories: ["分享"]
draft: true
---


# vue-cli3 单元测试 + 覆盖率

单元测试， cli3 自带有，而且官方教程也有说明如何使用，这里说下 Mocha 的覆盖率配置

参考资料

1. [vue-test-util 教程](https://vue-test-utils.vuejs.org/zh/guides)
2. [mocha-webpack coverage 配置](https://github.com/zinserjan/mocha-webpack/blob/master/docs/guides/code-coverage.md)
3. [issue - 配置覆盖率](<https://github.com/vuejs/vue-cli/issues/1363>)
4. [nyc 配置](<https://www.npmjs.com/package/nyc>)

## 配置

`npm i nyc babel-plugin-istanbul istanbul-instrumenter-loader -D`

vue.config.js

```js
chainWebpack: config => {
    if (isTesting) {
      config.devtool('eval');
      config.module
        .rule('istanbul')
        .test(/\.(js|vue)$/)
        .enforce('post')
        .include.add(path.resolve(__dirname, '/src'))
        .end()
        .use('istanbul-instrumenter-loader')
        .loader('istanbul-instrumenter-loader')
        .options({ esModules: true });
    }
  }
```

package.json

```json
{
  "script": {
    "test:unit": "vue-cli-service test:unit",
    "coverage:unit": "nyc vue-cli-service test:unit"
  },
  "nyc": {
    "check-coverage": true,
    "per-file": true,
    "instrument": false,
    "sourceMap": false,
    "reporter": [
      "lcov",
      "text",
      "text-summary"
    ],
    "include": [
      "packages/**/*.{js,vue}"
    ]
  }
}
```

babel.config.js

```js
{
  plugins: ['istanbul']
}
```

这样就可以了。执行 `npm run coverage:unit` 后，会在项目根目录生成 .nyc_output 和 coverage 文件夹（记得忽略掉），可以看到每个文件的覆盖率

# 如何编写高效的 vue 组件

> 这篇文章主要有两个部分，一个是总结组件的日常使用，另一个是编写组件的技巧。

## 基础

```javascript
// 在全局声明一个叫 `testComponent` 的组件
Vue.component('testComponent', {
  template: `<div>{{ a }}</div>`,
  data() {
    return { a: 'this is a test component' }
  }
})

// html
<test-component />

```

```javascript
// 单文件组件 - test.vue
<template>
  <div>

  </div>
</template>

<script>
  export default {
    
  }
</script>

<style scoped>

</style>

// use
<template>
  <div>
    <test />
  </div>
</template>

import test from './test'
export default {
  components: {
    test
  }
}
```

上面是最常用的两种方式，可以通过纯 js 的方式，在 `Vue` 内声明一个全局组件或者子组件，或者使用单文件组件的模式。如果组件有大量的 html 或者有 css 的话，建议使用单文件组件。

### 动态组件

以下使用的是官网例子：

```vue
<template>
<div id="dynamic-component-demo" class="demo">
  <button
    v-for="tab in tabs"
    v-bind:key="tab"
    v-bind:class="['tab-button', { active: currentTab === tab }]"
    v-on:click="currentTab = tab"
  >{{ tab }}</button>

  <component
    v-bind:is="currentTabComponent"
    class="tab"
  ></component>
</div>
</template>

<script>
// js
Vue.component('tab-home', { 
	template: '<div>Home component</div>' 
})
Vue.component('tab-posts', { 
	template: '<div>Posts component</div>' 
})
Vue.component('tab-archive', { 
	template: '<div>Archive component</div>' 
})

new Vue({
  el: '#dynamic-component-demo',
  data: {
    currentTab: 'Home',
    tabs: ['Home', 'Posts', 'Archive']
  },
  computed: {
    currentTabComponent: function () {
      return 'tab-' + this.currentTab.toLowerCase()
    }
  }
})
</script>
```

```vue
// 核心：
<component :is='currentComponent' />
```

`currentComponent` 指向一个组件对象，通过修改这个变量，从而渲染不同的组件。这样就不需要通过 `if` 去判断渲染哪个组件。



### props

组件可以通过 `props` 进行传参。因为组件之间的数据流，是单向的，从父到子。所以组件不能更改 `props` 的数据。`props` 接受两种数据格式，数组和对象

* `props: ['test', 'test-b', 'test-c']`
* `props: { a: { ... }, b: { ... } }`

第一种数组就不用说明了，只是简单地声明有这个属性而已。主要说下第二种，因为可配置的更多。

* type: 类型 `Number|String|Boolean|Object|Array|Date|Function|Symbol`
* default: 默认值, 如果是数组或者对象，要从工厂函数返回默认值
* required: 是否必须
* validator: 自定义验证

```javascript
props: {
    parentD: {
      validator: (value) => {
        // 如果参数的长度小于等于 5 就会报错
        return value.length > 5
      }
    }
  }
```

### 添加原生属性

```javascript
<test-component parentD='122323' data='123' />
```

`data` 是没有在 `props` 声明的，然后我们就会在 dom 看到 `<div data='123'>` `data` 属性。利用这个特性，可以添加一些静态原生的属性，不需要再在 `props` 声明。这对于一些表单组件，会非常有用。

```javascript
// component
{ template: `
  <input class='input' type='number' />
` }

// use
<u-input class='test' type='tel' />
  
// dom
<input class='test input' type='tel' />
```

如果有重复定义的属性，后者会覆盖。而 `class` 却会叠加，非常人性化。



### 自定义事件

自定义事件，这个用得比较多：

```javascript
this.$emit('test')

<u-test @test='handle' />
```

这也是一个父子通信比较常用的方法。

#### v-model

```javascript
Vue.component('textInput', {
  model: {
    // 设置的属性名
    prop: 'data',
    // 在什么方法更改后，进行修改
    event: 'change'
  },
  props: {
    data: String
  },
  template: `
    <input
      type="text"
      :value='data'
      // 向父组件更新
      v-on:change="$emit('change', $event.target.value)"
    >
  `,
  methods: {
  }
})

// use
<test-input v-model='testData' />
  
```

如果像子组件修改 `props` 的数据，就使用 `v-model`。父组件调用子组件，定义 `v-model` 并传入一个属性。在子组件内部，定义 `model` 和 `props`。

不了解的话，可以看下[这篇文章](https://segmentfault.com/a/1190000009492595)

#### .sync

```javascript
Vue.component('uInput', {
  template: `
    <input class='input' v-model='t2' type='number' />
  `,
  props: ['t'],
  data() {
    return {
      t2: this.t
    }
  },
  watch: {
    t2() {
      console.log(this.t2)
      this.$emit('update:t', this.t2)
    }
  }
})

// use
<u-input :t.sync='d2' />
```

`.sync` 可以使 `props` 双向绑定。当子组件 `t2` 属性发生变化，使用 `emit` 通知 `update:[propname]` 父组件更新属性值（内部自动处理）。

`.sync` 是一个语法糖，就是下面代码的简写：

```javascript
// .sync
<u-input :t.sync='d2' />
  
// native 
<u-input :t='d2' @update:t='d2 = $event' />
```

`.sync` 和 `v-model` 的功能类似，可以使子组件修改 `props` 的数据。而 `.sync` 支持多个属性，`v-model` 只能一个。



### 插槽

```javascript
Vue.component('testComponent', {
  template: `
    <div>
      <p>{{ a }}</p>
      <div>{{ parentD }}</div>
      <slot></slot>
    </div>
  `,
})

<test-component >
  <h1>{{ d3 }}</h1>
</test-component>
```

通过定义 `slot`，可以直接在组件包裹内，定义内容。

下面引用 vue 官方文档的说明

> 父组件模板的所有东西都会在父级作用域内编译；子组件模板的所有东西都会在子级作用域内编译。

`testComponent` 的 `a` 属性是子组件作用域的， `d3` 属性是父组件作用域的。

插槽支持具体和默认值

```javascript
// 子组件
<div>
  <slot name='footer'>sdfsdfdf</slot>
</div>

// 父组件
<test>
  <div slot='footer'>hah</div>
</test>
```

插槽作用域，子组件可以传递属性，以供父组件调用时使用

```javascript
// 子组件
<div>
  <slot name='footer' :descData='data'>sdfsdfdf</slot>
</div>

// 父组件
<test>
  <div slot='footer' slot-scope='slotProps'>{{ slotProps.descData }}</div>
</test>
```

### 依赖注入

```vue
// 父组件，声明暴露什么接口给子组件
{
  provide: function () {
    return {
      getMap: this.getMap
    }
  }
}

// 子组件，接受父组件的接口
{
  inject: ['getMap']
}

```



## 编写组件

### 一次性定义所有的 `prop`

```vue
Vue.component('Test', {
  props: ['a1', 'a2', 'a3', 'a4', 'a5', 'a6', 'a7'],
  template: `
    <input v-bind='$props' >
  `
})

// 调用
<test :a1='data_a1' :a2='data_a2' ..... />

```

通过 `v-bind='$props'` 这一步操作，把 `props` 里面所有的属性都绑定了。这种方法很适合二次封装第三方的组件，我在使用 `ElementUi` 的时候，二次封装 `el-select` 、`el-table` 这些常用的组件的时候，就会用到这个方法。



待续......