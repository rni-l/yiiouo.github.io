---
title: fabricjs的学习使用
date: 2017-06-27 00:00:00
tags: ["js", "canvas", "fabricjs"]
categories: ["记录"]
---
# fabric.js 使用记录

> 有个项目需要用到 canvas，有大量的 canvas 基础操作，所以就选择了这个 canvas 的对象模型库

## 安装

```
npm i fabric.js -S
```

就是用 import 导入，fabric 对象貌似也会存在全局中。。。

## 介绍

它里面有很多模块: Object, rect, path, image 等等，我们可以很轻易实例化一个对象出来，`new fabric.rect({...})` 。除了这些之外，还有监听事件，动画，组等等（TODO）

## 使用

### canvas

```javascript
var canvas = new fabric.Canvas('test', { ... })
```

第一个参数是 canvas 标签的 id ，第二个参数是配置参数。实例化， canvas 节点会有所变化，多了一个父级 div 嵌套，和多一个 canvas。这样方便优化~

#### canvas 的配置参数（TODO）

* selection, default -> true, 是否可以选择

#### methods

* add, canvas.add(obj)
  * 添加对象


* remove, canvas.remove(obj)
  * 移除 canvas 里某个对象
* renderAll, canvas.renderAll()
* requestRenderAll, canvas.requestRenderAll()（FOCUS）
  * 重绘全部内容
* dispose
  * 清除所有内容，包括方法和 canvas 节点
* removeListeners
  * 清除所有方法
* clear
  * 清除所有对象，background，但是 canvas 不会删除
* setActiveObject, canvas.setActiveObject(obj)
  * 选中目标
* getPointer, canvas.getPointer(e)
  * 传入事件对象，获取相对于 canvas 左上角的坐标（这个真的很实用）


#### 回调事件

* canvas.on('before:render')
  * 渲染前的回调（调用 canvas.renderAll()）
* canvas.on('after:render')
  * 渲染后的回调


### Object

```javascript
var obj = new fabric.Object({ ... })
```

下面介绍的，其它对象都会有的属性和方法

#### 参数

* lockScalingFlip
  * 禁止 scale 值为负

#### methods

*  get, obj.get(key)
  * 获取对象属性的值
*  set, obj.set({ ..  })
  * 接受一个对象参数，设置对象的属性
*  getBoundingRect, obj.getBoundingRect()
   *  获取对象，边界值，left top width height（真实的，就算是经过缩放的，也会计算后给你）

### Rect

```javascript
new fabric.Rect({
  width,
  height,
  angle: 0,
  fill: 'rgba(255,0,0,0.5)',
  hasControls: true
})
```

生成一个 Rect 对象，其它属性，和其它形状对象基本类似，很多我们看单词的意思就知道它是干嘛了，所以就单独挑一些我不会的~~

#### 参数

* width, height, top, left, angle, fill,stroke,
* hasControls, default -> true, 是否有控制器


###  Text

```javascript
new fabric.Text(this.targetObject.name, {
  fontSize: 16,
  textBackgroundColor: 'black',
  fill: 'white',
  left: 2
})
```

#### 参数

* fontSize, fontStyle, fontWeight 等等，都和 canvas 设置文本一样
* textBackgroundColor 背景颜色

#### 方法

* setText
  * 设置文本



### Group

Group 这个是非常好用的对象。我可以把多个 rect 组合成一个 Group，最后添加到 canvas 里，这样在拖动缩放时，会整体一起变化。而且要单独改变 Group 里面的对象也是没问题的。

```javascript
const group = new fabric.Group([text, rect], {
  left: this.origX,
  top: this.origY
})
canvas.add(group)
```

在 Group 对象里，还有 _objects 属性，可以修改 Group 内对象的属性

## 问题

1. 使用 remove 移除对象，添加后，再移除，再添加的对象，会无法移除（项目的过滤模式）。现才用 clear() 再  add 添加对象的方法解决
2. 对象的事件，没有返回当前的对象?
3. font-size, strokewidth 等属性，会随着 scale 值而变化！拉动 pointer，改变的是 scale，而不是宽高（艹）
   1. 解决，根据 strokeWidthUnscaled 和 scale 值解决，但是，只能解决单对象，如果是 group 效果就不怎么好了。
   2. group，要修改里面的子对象，但是会出现错位，group 的框会比子对象的 rect 宽高大