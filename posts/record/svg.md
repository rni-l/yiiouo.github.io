---
title: svg
date: 2019-05-09 17:59:02
tags: ["css", "svg"]
categories: ["记录"]
---

## svg

### 参考文章

1. [张鑫旭 svg smil 介绍](<https://www.zhangxinxu.com/wordpress/2014/08/so-powerful-svg-smil-animation/>)
2. [mdn](<developer.mozilla.org/zh-CN/docs/Web/SVG>)

### 使用

```html
<svg version="1.1" baseProfile="full" width="300" height="200" xmlns="http://www.w3.org/2000/svg">
  
  <rect width="100%" height="100%" fill="red" />

  <circle cx="150" cy="100" r="80" fill="green" />

  <text x="150" y="125" font-size="60" text-anchor="middle" fill="white">SVG</text>

</svg>
```

使用时，要先声明 `svg` 标签，做为一个声明空间。而 `xmlns` 这个属性就声明这个声明空间是什么

`version` 和 `baseProfile` 也是必须

### 注意点

1. svg 其实就是一种标签，通过配置属性显示相关的内容
2. 可以使用 iframe 引入
3. SVG也能使用相对大小，只需给出数字，不标明单位，输出时就会采用用户的单位

### 常用标签

#### rect

```html
<rect x="10" y="10" width="30" height="30"/>
```

#### circle

```html
<circle cx="25" cy="75" r="20"/>
```

#### ellipse

```html
<ellipse cx="75" cy="75" rx="20" ry="5"/>
```

cx, cy 表示椭圆圆心坐标， rx 表示 x 轴的半径， ry 表示 y 轴的半径

#### line

```html
<line x1="10" x2="50" y1="110" y2="150"/>
```

#### polyline

```html
<polyline points="60 110, 65 120, 70 115, 75 130, 80 125, 85 140, 90 135, 95 150, 100 145"/>
```

#### polygon

```html
<polygon points="50 160, 55 180, 70 180, 60 190, 65 205, 50 195, 35 205, 40 190, 30 180, 45 180"/>
```

#### path

```html
<path d="M 20 230 Q 40 205, 50 230 T 90230"/>
```

`d` 声明这个 `path` 内容显示

规则：

1. M 表示 `move to` ，移动到某个坐标上
   1. 大写字母表示绝对定位，小写字母表示相对定位（相对于上一个点）
   2. M命令仅仅是移动画笔，但不画线。

2. `L` 表示 `line to`，需要两个参数：`x` 和 `y`

3. `H` 绘制水平线

   1. 只带一个参数，标明在x轴移动到的位置

4. `V` 绘制垂直线

   1. 只带一个参数，标明在y轴移动到的位置

5. `Z` 声明闭合路径命令，不是必须

6. `C` 三次贝塞尔曲线命令

   1. ```html
      C x1 y1, x2 y2, x y (or c dx1 dy1, dx2 dy2, dx dy)
      ```

   2. 最后一个坐标(x,y)表示的是曲线的终点，另外两个坐标是控制点，(x1,y1)是起点的控制点，(x2,y2)是终点的控制点



### 动画

动画元素放在形状元素的内部

#### set

元素可以用来设定一个属性值，在某个时间后赋予一个新值

```html
<set attributeName="x" attributeType="XML" to="60" begin="3s" />
```

3 秒后，x 轴数值变为 60

#### animate

用来定义一个元素的某个属性如何踩着时点改变。在指定持续时间里，属性从开始值变成结束值

```html
<animate attributeName="x" from="160" to="60" begin="0s" dur="3s" repeatCount="indefinite" />
```

在 3 秒内，x 轴从 160 变位 60，循环播放

#### animateColor

已废弃

#### animateMotion

引用的元素沿着运动路径移动

```html
<animateMotion path="M10,80 q100,120 120,20 q140,-50 160,0" begin="0s" dur="3s" repeatCount="indefinite"/>
```

#### animationTransform

和 `animate` 类似，不过允许动画控制转换、缩放、旋转或斜切

```html
<animateTransform attributeName="transform" begin="0s" dur="3s"  type="scale" from="1" to="1.5" repeatCount="indefinite"/>
```



#### 动画属性介绍

1. **attributeName** 声明动画的属性

2. **attributeType** 声明属性类型，

   1. xml, css, auto 三个值可选，让浏览器知道根据哪个类型的属性，实现动画

3. **from, to, by, values**

   1. 张鑫旭大神总结：`from-to`动画、`from-by`动画、`to`动画、`by`动画以及`values`动画。

4. **begin, end** 开始、结束时间

   1. beigin, end 这两个值除了可以设置数值外，还可以设置另外一些特殊值

   2. **offset-value** 就是一些数值

   3. **syncbase-value** 基于另一个动画的开始或结束

      ```html
      <animate id="x" attributeName="x" to="60" begin="0s" dur="3s" fill="freeze" />
      <!-- 基于 id 为 x 的结束时间，开始 -->
      <animate attributeName="y" to="100" begin="x.end" dur="3s" fill="freeze" />
      ```

   4. **event-value** 基于事件

      1. focusin、 focusout、 `activate`、 `click`、 `mousedown`、 `mouseup`、 `mouseover`、 `mousemove`、 `mouseout`、 `DOMSubtreeModified`、 `DOMNodeInserted`、 `DOMNodeRemoved`、 `DOMNodeRemovedFromDocument`、 `DOMNodeInsertedIntoDocument`、 `DOMAttrModified`、 `DOMCharacterDataModified`、 `SVGLoad`、 `SVGUnload`、 `SVGAbort`、 `SVGError`、 `SVGResize`、 `SVGScroll`、 `SVGZoom`、 `beginEvent`、 `endEvent`和`repeatEvent`
      2. 不一定所有元素都支持上面的事件

   5. **repeat-value** 事件发生了指定次数的时间点，被定义为元素动画的开始时间点

      ```html
      <!-- x 重复了两次后才开始 -->
      <animate attributeName="y" to="100" begin="x.repeat(2)" dur="3s" fill="freeze" />
      ```

   6. **accessKey-value** 当用户按下指定的键时，元素动画就开始了（注意兼容性）

      ```html
      <!-- accessKey(s) 表示按下 s 后就执行 -->
      <animate attributeName="x" to="60" begin="accessKey(s)" dur="3s" repeatCount="indefinite" />
      ```

   7. **wallclock-sync-value** 是真实世界钟的时点

   8. 通过 js/svg 触发

      ```html
      <animate id="animate" attributeName="x" to="60" begin="indefinite" dur="3s" />
      
      <a xlink:href="#animate">
          <text x="10" y="20" fill="#cd0000" font-size="30">点击我</text>
      </a>
      
      var animate = document.getElementsByTagName("animate")[0];
      if (animate) {
          document.getElementById("svg").onclick = function() {
              animate.beginElement();
          };
      }
      ```

      

5. **dur** 动画时间

6. **calcMode** 控制动画速度

   1. discrete: from 直接跳到 to
   2. linear: 匀速
   3. paced: 
   4. spline: 贝塞尔曲线

7. **keyTimes** 设置多个值，进行分隔

8. **keySplines** 

9. **repeatCount** 执行次数

10. **repeatDur** 重复动画的总时间

11. **fill** 动画间隙填充方式

    1. freeze: 动画结束了保持状态
    2. remove: 动画结束后回到开始地方

12. **accumulate** 累积

    1. none
    2. sum 动画结束后的位置，为下次动画的起始位置

13. **additive** 动画是否附加

    1. replace
    2. sum 动画的基础知识会附加到其他低优先级的动画上



循环动画

```html
<svg width="320" height="200" xmlns="http://www.w3.org/2000/svg">
    <path d="M10 100 C80 50, 150 50, 220 100" stroke="#4fd" fill="transparent" >
      <animate attributeName="d" dur="1s"  repeatCount=indefinite
        values="M10 100 C80 50, 150 50, 220 100; M10 100 C80 140, 150 140, 220 100; M10 100 C80 50, 150 50, 220 100"
      />
    </path>
  </svg>
```



### 渐变

```html
<svg>
  <def>
    <linearGradient x1="0" x2="0" y1="0" y2="1" id="bg">
          <stop stop-color="#031336" offset="0%"></stop>
          <stop stop-color="#031337" offset="80%"></stop>
          <stop stop-color="#0D1A36" offset="100%"></stop>
    </linearGradient>
   </def>
   <rect fill="url(#bg)" />
</svg>
```



### viewbox 的使用

```html
<svg viewbox="{min-x} {min-y} {width} {height}">
  <rect x=1 y=1 width="100" height="100" />
</svg>
```

上面是一个矩形的 svg

假如viewbox 我设置成 “0, 0, 200, 200”，svg 宽高为：200, 200，则矩形就会在 svg 左上角为初始点， (1, 1) 开始绘制

假如viewbox 我设置成 “0, 0, 200, 200”，svg 宽高为：400，400，则矩形就会在 svg 左上角为初始点， (2, 2) 开始绘制

假如viewbox 我设置成 “0, 0, vw, vh”，svg 宽高为 sw，sh，则矩形就会在 svg 左上角为初始点， (1 * (sw / vw), 1 * (sh / vh)) 开始绘制

简单来说，设置了 viewbox 后，该 svg 里面的标签的具体坐标、宽高数值等，是根据 viewbox 进行定位

然后 svg 进行缩放， svg 里面的标签数值，就会根据 svg 的宽高和 viewbox 的比例进行一个转换

相当于移动端的 rem 缩放效果

#### preserveAspectRatio

[详情看文档](https://developer.mozilla.org/zh-CN/docs/Web/SVG/Attribute/preserveAspectRatio)

默认是按当前 svg 等比缩放，如果设置了 none,则变位强制缩放









