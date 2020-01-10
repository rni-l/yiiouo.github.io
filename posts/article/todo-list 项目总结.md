---
title: todo-list 项目总结
date: 2018-06-27 00:00:00
tags: ["js", "react"]
categories: ["分享"]
---
# todo-list 项目总结

> 这个 todo-list 项目，很简单，前端使用 react，后端 nodejs 使用 koa2 进行开发。数据库使用 Mysql。之所以要选择这些框架、数据库，是因为我都不会这些技术，为了学习，所以就使用这些技术进行开发。

在这次的小项目开发中，大概用了1个月的时间吧，平时有空就开始开发或者学习。

而这篇文章，主要是记录了在学习开发过程，遇到的一些问题，是如何解决的。

## 使用的框架、库

### 前端

项目脚手架是用 create-react-app 生成的

* react
* react-router
* redux

### 后端

后端用的是 nodejs

* koa2
* mysql



## 前端开发遇到的问题

### create-react-app

#### css module 问题

用 create-react-app 生成的项目后，再用 npm run eject 把 webpack 配置文件生成出来（因为要修改配置）。原本打算使用 css module + scss 进行开发的。但是我在配置完 webpack 之后，css module 有个问题解决不了

*通过 composes 引用的类，在源文件使用了变量或者 @include 都是没有效果的，还有使用了 postcss-px2rem 这个插件，也是无法转成 rem 的。*

不知道是我配置哪里出现了问题，所以就暂时使用 scss 进行开发，后面再找下有没有现成的配置，看看是如何做的。

#### 其他添加的配置

只是简单得添加了 scss-loader 和 postcss-px2rem 插件进行使用。

使用 postcss-px2rem，在 css 中写 px，会根据配置参数，转译后变成 rem。方便移动端开发。

其他暂时没有什么大的问题，create-react-app 和 vue-cli 相比，功能也挺完整的。但是开发和打包的是两个配置文件，其中公共部分没有抽出来，感觉这点就比 vue-cli 不好了。

### react

因为我有 vue 的基础，所以感觉学习 react 的时候并不觉得很难，当然还没真正在工作项目中使用 react 开发，所以很多问题都没遇到。

react 是用 jsx 进行编写模板的。我自己从 vue 的 template 转到 react 的 jsx 感觉还是没什么问题。

react 自带的 api 并不多，很多都要去看看有什么库可以用，而选择库的时候，要看看其他开发者的介绍、评价等，再去选择使用，还有 github 的 star 数。

#### 动画库选择

像要实现动画效果时，vue 可以直接自带的 `<transition>` 的组件就好了，很容易实现：

```vue
<transition name='className'>
  <div v-if='isShow'>test</div>
</transition>
```



再编写 css 动画类就好了。而 react 可以找一些库去实现，没有什么约束，选择更多。

我这里就使用了 `react-motion` 这个动画库。可以实现更多物理效果的的动画。

```react
import {Motion, spring} from 'react-motion';
// In your render...
<Motion defaultStyle={{x: 0}} style={{x: spring(10)}}>
  {value => <div>{value.x}</div>}
</Motion>
```

官方 demo 看起来也感觉挺简单的。用起来感觉还行。

当动画结束后，有一个钩子函数可以使用 `onRest`

其他一些功能组件，都是自己尝试去编写的，像日历组件、picker 组件、toast 组件等。因为项目简单，所以需要的组件不多。

#### react-router

刚开始看的时候，知道 react-router (后面直接说 router 了) v4 版本和之前的版本区别有点大。然后选择了 v4 版本。v4 版本的 router，是没有任何的钩子函数，我就感觉懵逼了。。。拦截器功能要怎么实现。。。路由权限要怎么实现。一个 2.8W star 数的库，肯定有解决方法。

[关于 React Router 4 的一切](https://juejin.im/post/5995a2506fb9a0249975a1a4)

大家可以看下上面这篇文章，帮助很大。

##### 实现拦截器和路由权限功能

这个其实挺简单的，想一下就可以了，在每个 `Route` 组件，包多一层父级组件，每次匹配 Route 时，都要先经过那层父级组件，判断要输出什么。

```react
const AuthRoute = ({ component: Component, redirect, loginStatus, ...rest }) => (
  <Route {...rest} render={props => (
    loginStatus ? (<Component {...props} />) : (
      <Redirect to={redirect || {
        pathname: '/home'
      }} />
    )
  )} />
)
```

根据传进去的 loginStatus ，判断是要重定向还是输出原组件。这种做法就是“高阶组件”，接受一个组件参数，返回一个组件。

使用的时候就是这样：

```react
<AuthRoute path='/group/:id' component={GroupPage} loginStatus={loginStatus} />
```

然而麻烦的是，只要涉及到要权限判断的，就要用这层组件包裹。。。没用到的就不需要。然而大部分页面都是需要的。

##### 如何在 js 里进行路由调整？

react-router-dom 库是有依赖 history 这个库的。在组件里，用 `withRouter` （这个组件是在 react-router-dom 里面的）包住当前组件，就会在 `props` 出现 histroy、location、match 这三个对象。history 拥有 push、replace、go 等等多种路由操作的方法。

```react
import React, { Component } from 'react'
import { withRouter } from 'react-router-dom'

class Com extends Component {
  constructor(props) {
    super(props)
    // this.props.history....
  }
  
  render() {
    return (
      <div>
        
      </div>
    )
  }
}

export default withRouter(Com)

```



*如果我在纯 js 里面，要如何调整？*

请求 api 我是用 axios，我想在 axios 拦截器里，当判断后端返回你没有登录，就要自动跳去登录页。

网上有个教程是，使用 history 库，实例化一个 hisotry 对象，就可以 push 、replace 等。但是那个好像是在 v3 之前的版本才有效。在 v4 中，路由的 url 是变化了，但是内容没有变。

[react-router v4 使用 history 控制路由跳转](https://github.com/brickspert/blog/issues/3)

这个大家可以看下这位前辈写的方法。前两种是在组件里使用的。第三种方法是：

```react
import { Router, Link, Route } from 'react-router-dom'
import history from './history'

ReactDOM.render(
  <Provider store={store}>
    <Router history={history}>
      ...
    </Router>
  </Provider>,
  document.getElementById('root'),
)
```

把 history 传到 Router 组件里。然后其他地方就可以使用了。但这里有个问题，Router 组件是无法用 BrowserRouter 其他的一些功能，像 basename。这个感觉还是很必要，Router 组件是没有这个属性的选项的。我现在做的项目，基本上每个都要配置 basename（用的是 vue 开发的项目）。虽然可以用其他一些方法去添加这个 basename，但觉得很麻烦。

然后我就看了第四种方法。。。直接从 react-router-dom 里面的 BrowserRouter 源码里 copy 下面，自己封装了一个组件：

```react
import React from "react"
import { setHistory } from '@/utils/history'
import { createBrowserHistory as createHistory } from "history"
import { Router } from 'react-router-dom'

class BrowserRouter extends React.Component {

  history = createHistory(this.props)

  render() {
    setHistory(this.history)
    return <Router history={this.history} children={this.props.children} />
  }
}

export default BrowserRouter
```

其实 Router 组件，加上 history 对象和 children ，就是一个 BrowserRouter。在里面生成 history 对象，我自己就缓存起来。哪里要用，我就直接调用它就好了。

然后使用这个“假的” BrowserRouter，其他功能和“真的”是一样的。在里面加 basename 也是可以的。

这样我在 axios 里就可以跳转路由

```react
import axios from 'axios'
import { getHistory } from '@/uitls/history'

axios('http://xxxx.com/api').then(res => {
    if (res.code === 500) {
        getHistory().push('/login')
    }
})

```

上面就是我在 react-router 暂时遇到的问题。



#### redux

redux 也没什么好说的，教程网上搜搜或者看文档就好了。 redux 刚开始用的时候，感觉有别扭。。。转不过去，只能照搬复制粘贴用。可能用着用着，才明白，额。。原来是这么回事，然后再去刷文档。。。

主要是如何在 axios 里，更新 store 的数据。这个也很简单

```javascript
store.dispatch(setToast({ status: true, title: '请重新登录' }))
```

#### react 和 vue 开发时的区别

感觉做同样的事情，react 需要写的代码稍微会多点，感觉也麻烦点，不知道是不是还没熟悉，用 vue 感觉写起来很爽。

vue 提供的功能比较全面，像 computed, watch 这些，react 的话好像是没有提供的？react 更多需要找现成的库。不过自从学了 react，es6 也要更加熟练了。。。

#### 前端总结

整个前端开发起来，其实还好。一些公共处理的问题，在后面都解决到了。但还是要看看其他项目， 开发是如何做好约束，组件是如何分类等等。

## 后端开发遇到的问题

相对前端，后端基础等于没有，靠看文档、搜索、问同事，从而解决一些问题。

后端主要要做的是提供接口，进行简单的增删改查。数据库使用 Mysql。nodejs 框架就使用 koa2（因为没学过，就决定用它了）。

项目使用 koa2-generator 生成的。开箱既用。再自己添加一些库就好了。

连接 mysql 的话，用 npm 的一个 mysql 包就好了。

其他的话，后续更新吧~~

最后附上 [项目 github 地址](https://github.com/rni-l/todo-list)

[还有体验地址](http://www.rni-l.com)

