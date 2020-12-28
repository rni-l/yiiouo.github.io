---
title: redux
date: 2017-06-27 00:00:00
tags: ["js", "react"]
categories: ["记录"]
draft: true
---
# Redux

## 总结 reducx 的第一次使用

redux 流程~

一个项目只有一个 state，通过子对象进行分割模块。

```jsx
import { combineReducers } from 'redux'
const reducer = combineReducers({
  username,
  ago
})
const store = createStore(reducer)
```

通过 reducer，建立 state 模型

```jsx
const username = (
  state = '',
  action
) => {
  switch (action.type) {
    case 'SET_USERNAME':
      return action.username
    default:
      return state
  }
}

```

每个模块的值，可以通过默认值定义好，最后出来的 state 对象大概是这样：

```jsx
state = {
    username: '',
    ago: ''
}
```

reducer 就是定义每个子 state 的值，如何去修改它们。

生成的 store 有几个方法，getState，subscribe，dispatch

getState 是获取整个 state 的值。subscribe接受一个回调函数，监听 state 值的变化。dispatch 就是触发 reducer，修改 state 的值。

![redux 流程图](http://oxnbdd4i9.bkt.clouddn.com/redux-flow.png)

最后用 `<Provider store={store}` 包住原本的根标签。使每个组件都能使用 dispatch 方法。

## 疑惑

### action 的作用

其实不一定要定义 action，直接 dispatch，传入对应的对象值，就可以了。以我现在的理解，定义 action 可以更清晰的知道，要触发的 dispatch 是要干嘛的，要什么参数，触发哪个 reducer。

### state 树

要找下好的例子，看看是如何让 state 树更加清晰得显示，而不是一个个 reducer 去看 state 的默认值是什么。