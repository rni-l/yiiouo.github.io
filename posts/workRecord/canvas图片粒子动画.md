---
title: canvas图片粒子动画
date: 2017-06-27 00:00:00
tags: ["js", "cavans"]
categories: ["工作记录"]
---

# canvas图片粒子化效果的实现

>这次分享的是 `cavans` ，通过 `js` 控制图形、动画的实现。而这次主要是分享粒子动画的实现原理，而不是介绍 `canvas` 的 `api` ， `api` 大家去 `MDN` 看看就好了

## 关键词

* [canvas](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Basic_usage)
* [ImageData](https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Pixel_manipulation_with_canvas)
* [requestAnimationFrame](http://www.zhangxinxu.com/wordpress/2013/09/css3-animation-requestanimationframe-tween-%E5%8A%A8%E7%94%BB%E7%AE%97%E6%B3%95/)

## 相关基础要点

### 兼容性

`canvas` 的兼容性是支持 `ie9` ， `requestAnimationFrame` 的兼容性支持是 `ie10` 不过 `requestAnimationFrame` 可以使用 `setTimeout` 代替

### canvas 的坐标系统

    var canvas = document.querySeletor('canvas')
    canvas.width = 200
    canvas.height = 200

每个 `canavs` 的坐标起始点，都是 `(0, 0)` ，起始点是**左上角**

![](/images/canvas_png1.png)

### 获取随机范围值

这个获取随机范围值，在后面会用到，实现的算法也有多个，这里凑合下

    function random(max, min) {
      return Math.ceil(Math.random()*(max-min+1)+min)
    }

## 如何使用 canvas

    // 获取 canvas 的上下文
    var canvas = document.querySeleteror('canvas')
    var ctx = canvas.getContext('2d')

`.getContext(2d)` 这个方法获取一个 `2d` 的 `canvas` 的上下文对象，后面的绘制和渲染都要靠 `ctx` 执行

然后我们就可以使用 `ctx` 这个上下文进行操作

    // 获取一个图片实例
    var img = new Image()
    // 图片加载完毕后，绘制在 canvas 
    img.onload = function() {
      ctx.drawImage(img, 0, 0)
    }
    // 赋值图片地址
    img.src = './icon/set.png'

上面的代码执行后，就会在 `canvas` 上出现一张图片了。绘制图片的话，一定要等图片加载完毕后，才能执行 `canvas` 的绘制，不然什么都没有


## 实现粒子动画

这里就大概说说实现粒子动画的原理

通俗点，就是多个粒子，在起始位置，去到终点位置，有一个持续性的效果。

我们先要获取两个数据，起始位置和终点位置

而每个粒子都有属于自己的起始位置，终点位置，速度，颜色，大小等属性，也就是说每个粒子都是一个独立的个体

伪代码：

    function Create(...){
      this.firstX = x
      this.firstY = y
      this.lastX = lastX
      this.lastY = lastY
      this.rgba = rgba
    }

    for (var i=0; i< 100; i++) {
      // 生成100个粒子
      new Create()
    }

为了效果好看，每个粒子的属性都应该是所有区别的

### 起始位置

起始位置，这个看需求而定，我这里的效果是从下到上显示的动画效果

    function Create() {
      // 水平中间点
      this.x = cW / 2
      // 垂直方向，有一个随机的分布
      this.y = random(cH + 100, cH)
    }

### 终点位置

实现这个粒子动画，最关键的一点就是如何获取每个粒子的终点位置

这里就要使用 `ctx.getImageData` 这个方法获取图片的**像素数据**

#### ImageData

`ImageData` 有以下三个**只读**属性

    console.log(ctx.getImageData('2d'))
    // ... 打印数据
    {
      width: ,
      height: ,
      data: []
    }

而 `data` 这个数组不是普通的数组，是一个 [Uint8ClampedArray](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Uint8ClampedArray#兼容性注意事项) 8位无符号整型固定数组，里面的每一个值都是在 [0 - 255] 之间的

使用 `getImageData` 获取到的数据，其实就是每一个像素点的 `rgba` 值

    var rgba1 = `rgba(${data[0]}, ${data[1]}, ${data[2]}, ${data[3]})`
    var rgba2 = `rgba(${data[4]}, ${data[5]}, ${data[6]}, ${data[7]})`
    ......

也就是说，每4位就组成一个像素点的值

而在一个 `200 * 200` 的 `canvas` ，通过 `getImageData` 获取的数据是：
 
    {
      width: 200,
      height: 200,
      data: [160000]
    }

`data` 的长度就会有16万长，是这样算的： 200 * 200 * 4

那么我们究竟如何获取到需要的终点位置？

我们要明确一点，这是像素级的操作，每一个点，代表1像素 

![](/images/canvas_png2.png)

上面一个圆圈代表1像素，总共有4个像素那么每个像素点的坐标是不是这样：`[0, 0], [0, 1], [1, 0], [1, 1]`

在 `200 * 200` 的 画布里，就会有4万个像素点

    for (var x = 0; x < width; x++) {
      for (var y = 0; y < height; y++) {
        // 这样就能获取到所有像素点的位置
      }
    }

如果我们让4万个粒子同时在画布里动起来，那肯定会卡死的，我们肯定要进行筛选，获取对我们“有用”的像素点

#### 筛选有用的像素点

![](/images/canvas_png3.jpg)

先看看这张图片，我是在 PS 上进行截图的，除了黑色线，其余都是透明的，我们获取后的 `ImageData` ，透明的点表现的数值是：(0, 0, 0, 0) 这样的，这对我们根本没用，所以我们可以根据透明度进行筛选像素点

因为 `ImageData` 是 `Uint8ClampedArray` ，里面所有的值都是 [0 - 255] 之间的，我们只需要筛选透明度值是 128 以上的像素点就行了

    for (var x = 0; x < width; x++) {
      for (var y = 0; y < height; y++) {
        // 筛选
        var i = 4*((y - 1) * width + x)
        if (data[i + 3] >= 128) {
          // ... 获取到坐标和像素值
          {
            x: x,
            y: y,
            rgba: ...
          }
        }
      }
    }

上面的 `var i = 4*((y - 1) * width + x)` 这步骤，就是获取这个像素点的索引值

这样获取后的像素点会减少了很多，如果想减少更多的话：

    for (var x = 0; x < width; x+= 4) {
      for (var y = 0; y < height; y+= 4) {
      }
    }

### 动画效果

粒子需要的数据都获取到了，然后就是实现动画效果，这里就不详细说了，贴上主要的代码：

    // 计算粒子的坐标
    function compute(v) {
      if (frames > 120) {
        v.x = v.maxX
        v.y = v.maxY
      }
      return v
    }
    // 绘制粒子
    function drawPoint() {
      // 擦除画布
      ctx.clearRect(0, 0, w, h)
      for(var i=0;i<list.length;i++){
        var v = list[i]
        // 计算
        var _v = compute(v)
        if (_v) {
          v.x = _v.x
          v.y = _v.y
        }
        // 开始路径
        ctx.beginPath()
        ctx.fillStyle = v.rgba
        ctx.arc(v.x, v.y, 1, 0, Math.PI*2, true)
        ctx.fill()
        // 关闭路径
        ctx.closePath()
      }
    }
    // 运行
    function animate() {
      drawPoint()
      if (frames > 120) {
        return false
      }
      frames++
      requestAnimationFrame(animate)
    }
    animate()

### requestAnimationFrame

`requestAnimationFrame` ，现在的浏览器支持的是60帧，也就是说1秒要运行60次，这是最佳的效果。但是当我们`js`有复杂的运算的话，会达不到这效果，而 `requestAnimationFrame` 当回调函数完成后，才会触发下一个回调函数，这就是 `requestAnimationFrame` 优于 `setTimeout` 的地方。 `setTimeout` 无论你的回调函数是否在指定时间内完成，只要过了设置的时间，都会执行下一步的回调，从而造成了卡顿的效果

## 总结

这次的分享，主要是介绍下 `canvas` 和普通动画的实现。通过 `ImageData` 获取到像素点的属性位置等等属性，从而实现粒子动画。 `canvas` 的功能远远不止这些，能做的事情有很多。
