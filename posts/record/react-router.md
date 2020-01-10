---
title: react-router
date: 2017-06-27 00:00:00
tags: ["js", "react"]
categories: ["记录"]
draft: true
---
# react-router

> 使用的是 react-router@4.x.x 版本

## 官方例子

```jsx
import React from 'react'
// 引入 react-router-dom(为什么不是 react-router ...我也不知道)
import {
  BrowserRouter as Router,
  Route,
  Link
} from 'react-router-dom'

// 定义四个组件
const Home = () => (
  <div>
    <h2>Home</h2>
  </div>
)

const About = () => (
  <div>
    <h2>About</h2>
  </div>
)

// Topics 的子组件，根据 match 参数渲染内容
const Topic = ({ match }) => (
  <div>
    <h3>{match.params.topicId}</h3>
  </div>
)

const Topics = ({ match }) => (
  <div>
    <h2>Topics</h2>
    <ul>
      <li>
        <!- 相当于 vue-router 的 router-link，路由跳转入口; to 是要去的目标路径 ->
        <Link to={`${match.url}/rendering`}>
          Rendering with React
        </Link>
      </li>
      <li>
        <Link to={`${match.url}/components`}>
          Components
        </Link>
      </li>
      <li>
        <Link to={`${match.url}/props-v-state`}>
          Props v. State
        </Link>
      </li>
    </ul>
    <Route path={`${match.url}/:topicId`} component={Topic}/>
    <Route exact path={match.url} render={() => (
      <h3>Please select a topic.</h3>
    )}/>
  </div>
)

const App = () => (
  <Router>
    <div>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/about">About</Link></li>
        <li><Link to="/topics">Topics</Link></li>
      </ul>

      <hr/>
      <!- 根据当前路由，显示匹配的 Route  ->
      <Route exact path="/" component={Home}/>
      <Route path="/about" component={About}/>
      <Route path="/topics" component={Topics}/>
    </div>
  </Router>
)
export default App;

```



### 解析

看了上面的官方例子后，感觉喝 vue-router 的配置有很大的区别。 vue-router 的配置，是先定义好，然后再赋值给 vue。这是静态的，定死了。

而 react-router，是动态的，你想在哪显示，就在哪配置 Route 组件。

因为现在才开始学习 react，对 vue 比较熟悉，还是比较倾向于 vue-router 的定义方式（好像旧版本的 react-router 也是静态的）。集中在一处地方显示，更加清晰知道路由的配置等。

我们再看下 react-router 的一些组件和配置方式。

### Link

没什么特别的一个组件

```jsx
<Link to='/'>生成的 DOM，这是一个 a 标签</Link>
```

从 react-router-dom 引入，相当于 vue 的 router-link 标签，`:to` 的值就是要跳转的路径

可以在 to 里面进行传参

```jsx
<Link to={{
  pathname: '/',
  state: {
    d: 'd'
  }
}} />
```

然后在路由组件，通过 location 可以获取到相应的参数

### exact

`<Route />`组件的属性，布尔值。 true 的话，匹配路径最后一个名字

```jsx
<Route path='/'  render={() => this.test1()} />
<Route path='/test2' render={() => this.test2()} />
```

上面的例子，在 /test2 路径下，是不会渲染 test2 的组件。

### Redirect

```jsx
<Redirect to='/' />
```

如果没有匹配到路径，就重定向指定位置

### switch

匹配标签内的 route，之渲染第一个匹配中的 route

### 通过 js 控制路由

>在网上找到的方法，[通过 js 控制路由](http://react-china.org/t/react-router-v4/12318)

class 声明的组件，在 props 属性内含有 history 对象，就可以做到 js 控制路由。

```markdown
history 对象通常会具有以下属性和方法：

length -（ number 类型）指的是 history 堆栈的数量。
action -（ string 类型）指的是当前的动作（action），例如 PUSH，REPLACE 以及 POP 。
location -（ object类型）是指当前的位置（location），location 会具有如下属性：
pathname -（ string 类型）URL路径。
search -（ string 类型）URL中的查询字符串（query string）。
hash -（ string 类型）URL的 hash 分段。
state -（ string 类型）是指 location 中的状态，例如在 push(path, state) 时，state会描述什么时候 location 被放置到堆栈中等信息。这个 state 只会出现在 browser history 和 memory history 的环境里。
push(path, [state]) -（ function 类型）在 hisotry 堆栈顶加入一个新的条目。
replace(path, [state]) -（ function 类型）替换在 history 堆栈中的当前条目。
go(n) -（ function 类型）将 history 对战中的指针向前移动 n 。
goBack() -（ function 类型）等同于 go(-1) 。
goForward() -（ function 类型）等同于 go(1) 。
block(prompt) -（ function 类型）阻止跳转
```



## DEMO

做一个 DEMO，功能如下：

* 多个页面
* 拥有子页面
* 点击按钮，跳到某个页面
* replace、back、go 等效果如何实现
* 如何做拦截器
* 路由跳转，如何传参