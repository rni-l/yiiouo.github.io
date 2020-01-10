---
title: react使用
date: 2017-06-27 00:00:00
tags: ["js", "react"]
categories: ["记录"]
draft: true
---
# React 学习记录

> 因为有 vue 的基础，所以打算根据 vue  用什么方法能实现什么功能，通过 react 再去实现。

注意：此文章并不是提供学习，只是我个人的学习记录，以供自己日后能快速查阅。

我是先看了下 React 教程，然后再根据功能尝试使用 React 编写功能。

## DEMO

> 应该 DEMO 都是通过 create-react-app 生成的

### tab 选项卡

> 主要有三个 js 文件 index.js，tabs.js，content.js。index.js 是主组件， tabs 和 content 是选项卡按钮和内容的组件。

index.js:

```jsx
import React from 'react'
import Tab from './tabs.js'
import Content from './content.js'

class Tabs extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      tab: 'first',
      content: ''
    }
    this.contentList = {
      first: () => (<div>first</div>),
      second: () => (<div>second</div>)
    }
    this.list = ['first', 'second']
    this.handlSetTab = this.handlSetTab.bind(this)
  }

  componentWillMount() {
    this.setState({
      content: this.contentList.first()
    })
  }

  handlSetTab(v) {
    this.setState({
      tab: v,
      content: this.contentList[v]()
    })
  }

  ListItems() {
    return this.list.map((v) =>
      <li
        key={v}
        onClick={() => this.selectTag(v)}
        className={this.state.tab === v ? 'on' : ''}>
        {v}
      </li>
    )
  }

  render() {
    return <div>
      <Tab list={this.list} tab={this.state.tab} onTagChange={this.handlSetTab} />
      <Content content={this.state.content} />
    </div>
  }
}

export default Tabs

```

tabs.js:

```jsx
import React from 'react'

class Tab extends React.Component {
  // constructor(props) {
  //   super(props)
  // }

  selectTag(v) {
    this.props.onTagChange(v)
  }

  ListItems() {
    console.log(this.props)
    return this.props.list.map((v) =>
      <li
        key={v}
        onClick={() => this.selectTag(v)}
        className={this.props.tab === v ? 'on' : ''}>
        {v}
      </li>
    )
  }

  render() {
    return <ul>
      {this.ListItems()}
    </ul>
  }
  
}

export default Tab

```

content.js

```jsx
import React from 'react'

class Content extends React.Component {
  // constructor(props) {
  //   super(props)
  // }
  
  render() {
    return <div className='content'>
      {this.props.content}
    </div>
  }
}

export default Content

```

一个很普通的选项卡功能，点击某个选项卡，显示对应的内容。

#### 生成 HTML

上面，每个 class 是一个组件，里面有一个 render 的方法，提供渲染 Html。就像vue 中的 template，把 html 写在 template 标签里。

如果要生成组件，首先引入了一个组件 Tab，在 render 函数 `return <div><Tab /></div>` 这样就回渲染 Tab 组件。通过标签形式生成的组件，首字母一定要大写。而且 return 后的代码，好像一定要用一个 html 标签嵌套着（我没用就报错了）。 

另外一种就是通过 Function

```jsx
return <div>
  {() => (<div>html</div>)}
</div>

// build
<div>
  <div>html</div>
</div>
```

最后就会生成这样的标签了。用单个大括号包住的代码，相当于在 js 作用域。

#### state 作用域

React 是单向绑定的。要修改值，要用到 setState。而我在 constructor 声明的属性，用 setState 是没效果的，属性一定要在 state 属性里面声明。

```jsx
constructor(props) {
  super(props)
  this.state = {
      haveResponse: true // 有响应
  }
  this.noResponse: true // 不会响应
}
```

如果直接修改 `this.noResponse` 是不会触发 render() 的

#### 绑定事件

```jsx
<div onClick={click}></div>
```

事件名要驼峰式写法。如果要获取事件对象，在事件的回调函数里，第一个参数就是了。不过打印 event 时，看不到原本那些属性值，但是要获取的时候，是有的。

如果要传其他参数，我感觉就有点麻烦了

```jsx
<div onClick={click.bind(this, 123)}></div>
// or
<div onClick={(e) => click(123, e)}></div>
```

#### HTML 属性

添加 class：

```jsx
<div className='test'></div>
```

好怪。。。

其他的属性值，引号是字符串，单大括号是属性值

#### 父子组件传值

我们可以看上面的例子。 index.js 引入了 tabs.js，然后在 render 函数里，使用 Tab 组件

```jsx
render() {
  const data2 = '123
  return <div>
    <Tab data='hah' data2={data2} />
  </div>
}
// Tab 组件
render() {
  return <div data={props.data} data2={props.data2}></div>
}
```

子组件通过 props 就可以获取父组件传给的值 。

因为 React 规范了，数据是单向的，从父组件 props 获取的数据，在子组件是不能修改的。这里使用了 onTagChange 事件告诉父组件要修改 props 的值了。类似 vue 的 $emit 方法。简单来说，子组件调用父组件定义的方法，传参，触发父组件的方法，进行修改值。

```jsx
handlSetTab(v) {
  this.setState({
    tab: v
  })
}
<Tab tab={this.state.tab} onTagChange={this.handlSetTab} />

// Tab
selectTag(v) {
  this.props.onTagChange(v)
}
render() {
  return <div onClick={() => this.selectTag(123)}></div>
}
```

#### 循环渲染

```jsx
this.list = [123, 234]
ListItems() {
  return this.list.map((v) =>
     <li
      key={v}
      onClick={() => this.selectTag(v)}
      className={this.state.tab === v ? 'on' : ''}>
      {v}
    </li>
  )
}

render() {
  return <ul>
    {this.ListItems()}
  </ul>
}
```



ListItems 方法返回一个数组，里面是循环生成的 li 标签。 key 的用法和 vue 相同。

#### 生命周期

```jsx
componentWillMount() {console.log('componentWillMount')}
componentDidMount() { console.log('componentDidMount') }
componentWillReceiveProps() { console.log('componentWillReceiveProps') }
shouldComponentUpdate() {
  console.log('shouldComponentUpdate')
  // 根据返回值，判断是否会阻止更新
  return true
}s
componentWillUpdate() { console.log('componentWillUpdate') }
componentDidUpdate() { console.log('componentDidUpdate') }
componentWillUnmount() { console.log('componentWillUnmount') }
```

[详细解析看文档](https://doc.react-china.org/docs/react-component.html)

tab 选项卡大概用了上面的这些功能，感觉覆盖到挺多功能了~



### form 表单

>学习使用表单标签

```jsx
import React, { Component } from 'react'

class UForm extends Component {
  constructor(props) {
    super(props)
    this.state = {
      user: '',
      pwd: '',
      sex: 'man',
      select: 2
    }
    this.inputChange = this.inputChange.bind(this)
    this.selectChange = this.selectChange.bind(this)
    this.submit = this.submit.bind(this)
    this.options = [1, 2, 3, 4]
  }
  inputChange(e) {
    this.setState({
      [e.target.name]: e.target.value
    })
  }
  selectChange(e) {
    this.setState({
      select: e.target.value
    })
  }
  submit() {
    const state = this.state
    let msg = ''
    if (!state.user) {
      msg = '请输入用户名'
    } else if (!state.pwd) {
      msg = '请输入密码'
    }
    if (msg) {
      return alert(msg)
    }
    alert('ok')
  }
  getOptions() {
    return this.options.map(v => <option key={v} value={v}>{v}</option>)
  }
  componentDidMount() {
    this.textInput.focus()
  }
  render() {
    return <div className='form'>
      <input
        type="text"
        name='user' value={this.state.user} onChange={this.inputChange} 
        ref={(input) => { this.textInput = input }} />
      <input autoComplete='new-password' type='password' name='pwd' value={this.state.pwd} onChange={this.inputChange} />
      <select value={this.state.select} onChange={this.selectChange}>
        {this.getOptions()}
      </select>
      <div onChange={this.inputChange}>
        <input type='radio' name='sex' value='women' />women
        <input type='radio' name='sex' value='man' />man
      </div>
      <button type='button' onClick={this.submit}>确定</button>
    </div>
  }
}

export default UForm

```

这个 DEMO 很简单，包含三种类型的标签: text, radio, select。

#### 绑定、获取值

```jsx
inputChange(e) {
    this.setState({
        user: e.target.value
    })
}
<input
  type="text"
  name='user' value={this.state.user} onChange={this.inputChange} />
```

首先在 input 标签的 value 里绑定一个值，再绑定 change 事件，触发回调后，修改 state 里的值。上面的例子相当于下面这个 vue 例子

```vue
<input text='text' :value='user' />
```

至于其他的表单标签，都是差不多~

如果不想每个标签都写一个绑定事件，可以根据 name 值，判断要修改那个 state

##### 如何绑定多个相同的事件？

```jsx
<input
  type="text"
  name='user' value={this.state.user} onChange={this.inputChange} onChange={this.inputChange2} />
```

上面这样是不行的。。。。后面的事件会覆盖前面的。

```jsx
<input
  type="text"
  name='user' value={this.state.user} onChange={(e) => {this.inputChange(e); this.inputChange2(e)}} />
```

大概要这样~

#### 获取 DOM

```jsx
<input
    type="text"
    name='user' value={this.state.user} onChange={this.inputChange} 
    ref={(input) => { this.textInput = input }} />
```

官方文档是这样说吗 ref 属性的：

> 当给 HTML 元素添加 `ref` 属性时，`ref` 回调接收了底层的 DOM 元素作为参数

这样就可以获取到 HTML 的 DOM 对象。

#### 非受控组件

```jsx
<input
  type="text"
  defaultValue='ddd'
  ref={(input) => { this.textInput = input }} />
```

下面是根据我现在的理解而写。

受控组件，重绑定值到修改值，全部写一遍，虽然麻烦，但是达到了所有部分都可以控制。

非受控组件，我们可以看下上面这个例子，通过 ref 回调事件，获取到对应的 DOM，然后在需要验证的时候，直接根据 this.textInput.value 去做处理。因为 this.textInput 是 DOM 对象，所有可以这样做。

非受控组件这样的做法，我觉得和双向绑定很相似，只要声明一个属性就行了。但是在 set 这个环节我们是无法做处理的，比如用户输入的值不符合要求，我就不执行 setState，这在非受控组件是做不了。



## 面试









