---
title: canvas-colorPicker
date: 2017-06-27 00:00:00
tags: ["js", "canvas"]
categories: ["分享"]
---

# 使用canvas制作的移动端color picker


> 我在另一个[demo](https://github.com/yiiouo/canvas-Image-processing)中，需要用到color picker，但是找不到我需要的移动端color picker，很多都是pc的，然后发现input[type='color']这个东西存在，发现安卓没问题，ios却不支持，但是我看安卓那个color picker貌似很简单就能实现，刚好另一个demo也是用canvas实现的，所以就打算自己写一个。


![color picker](/images/color.png)

效果大概就是这样。外层选择颜色，内层就是混合的。

## 怎么做？

这里有两部分，一个是外环，一个是正方形，两个形状内都有两个点，这两个点表示当前选择的颜色。外环的选择的颜色，决定正方形显示的颜色，正方形内的点，决定最终输出的颜色。

所以我先制作外环和正方形的颜色，然后给画布事件获取鼠标位置。再使用canvas的getImageData，我们只需要`ctx.getImageData(x,y,1,1)`就可以获取到rgba颜色了。

## 制作外环

制作这个外环，一开始想到是用渐变来处理的，但无论线性还是径向，都好像搞不出来，然后我google了下，使用线和hsl就可以搞出来了。我们这里制作的外环，是一个圆形，这里就可以用循环360，根据每次循环的度数，去制作线。而颜色就是`hsl(i , 100% , 50%)`这样就可以得到颜色。

### hsl

我们看看hsl是怎么组成的。
H：Hue(色调)。0(或360)表示红色，120表示绿色，240表示蓝色，也可取其他数值来指定颜色。取值为：0 - 360
S：Saturation(饱和度)。取值为：0.0% - 100.0%
L：Lightness(亮度)。取值为：0.0% - 100.0%

我们只需改变H(色调)就可以制造出常用的颜色出来。

### 获取坐标，制作线

canvas中的mveTo和lineTo，需要x,y坐标，才能制作出一条线，再通过hsl颜色，进行绘制。这里获取x,y坐标，用到了三角函数，`Math.sin,Math.cos`。要注意的是，它们接受的值是弧度，而不是角度，所以我们要先把角度转成弧度先`rad = i * (2 * Math.PI) / 360`。

![计算x,y坐标](/images/math.png)

看这图，应该能大概知道了x,y坐标的获取，不过这里获取的x,y是相对于圆心的距离，而不是相对于(0,0)这个点，所以我们还要加上半径r，才是最终的x,y坐标。

    for (var i = 0; i < 360; i += .1) {
    	//获取度数
    	var rad = i * (2 * Math.PI) / 360;
      //计算x,y坐标
      x = r + Math.cos(rad) * r;
      y = r + Math.sin(rad) * r;
      //然后连接线
      ctx.strokeStyle = "hsl(" + i + ", 100%, 50%)";
      ctx.beginPath();
      ctx.moveTo(r,r);
      ctx.lineTo(x,y);
      ctx.stroke();
      ctx.closePath();
    }

这样就可以制作出渐变色的圆形，但却不是圆环，因为中间是透明，没有颜色的(也不能是白色)。原本我打算再绘制一个白色圆来覆盖中间部分的，但我想要的是透明的效果，这样看起来会好很多。所以继续在上面循环做文章了。

既然终点可以计算，那起点也是同样的道理，只不过加的长度不同而已。

    for (var i = 0; i < 360; i += .1) {
    	//获取度数
    	var rad = i * (2 * Math.PI) / 360，
    	 angleX = Math.cos(rad),
    	 angleY = Math.sin(rad),
    	 lineW = 20;//圆环的厚度

      //然后连接线
      ctx.strokeStyle = "hsl(" + i + ", 100%, 50%)";
      ctx.beginPath();

      //计算坐标
      ctx.moveTo(r + angleX * (r - lineW),r + angleY * (r - lineW));
      ctx.lineTo(r + angleX * r,r + angleY * r);
      ctx.stroke();
      ctx.closePath();
    }

这样就可以制作出圆环的效果，而且中间的圆形是透明的。

## 制作正方形颜色

这里是用了三种颜色叠加起来的。首先是从外环获取到的颜色，加上从左到右的白色渐变色，加上从下到上的黑色渐变色。每当外环的颜色改变后，要重新生成正方形的颜色。

## 判断手指移动的坐标

接着，就要给canvas添加事件了，来判断手指移动的坐标，是否在圆环或者正方形内，然后根据这个坐标去生成颜色。

这里就主要说下，怎么判断移动时，是否在圆环内。

移动时，通过事件对象event，可以获取到x,y坐标

    var t = event.touches[0],
      x = t.pageX - oCan_left,
      y = t.pageY - oCan_top;

这里获取到x,y坐标还要减去画布到可视区的左，上的距离。因为我们计算的时候，是根据画布的坐标计算的，(0,0)坐标就是画布的左上角。

接着我们来计算，这个x,y坐标到圆心的距离`d = Math.sqrt(Math.pow((x - r),2) , Math.pow((y - r),2))`。这里用的是知道两点坐标，得到两点之间的距离，公式是这样的`d = √(x1 - x2)² + (y1 - y2)²`

![坐标图](/images/circle.png)

如上图，画布宽高其实就是外圆的直径，r1*2。开始点(0,0)在左上角。我们移动的坐标，到圆心的距离d，小于r1且大于r2的话，就是在圆环内了。
`d >= r2 && d <= r1`

## 获取颜色

这个简单，得到了x,y坐标后就可以直接获取颜色了，直接上代码

    var pixel = this.ctx.getImageData(x, y, 1, 1),
        data = pixel.data,
        rgb = 'rgb(' + data[0] + ',' + data[1] + ',' + data[2] + ')';
    return rgb

## 如何使用

我已经把代码上传到[github](https://github.com/yiiouo/canvas-colorPicker)了，大家可以下载使用。
只要把colorPicker.js和colorPicker.css导入就好了。按下面代码调用即可:

    //html结构
    <div class='colorPickerbox'>
      <canvas id='colorPicker'></canvas>
      <div class="colorPickerBtnWrap"></div>
    </div>

    //调用代码
    new ColorPicker({
      oBox: document.querySelector('.colorPickerbox'),//最外层
      oBtnWrap: document.querySelector('.btnWrap'),//按钮外层
      oCan: document.querySelector('#colorPicker'),//画布
      width: 200,//画布宽高
      height: 200,
      callback:function(color){//回调函数
        //color就是获取到的颜色
      }
    }).init()//初始化


# 总结

这个color picker我用了一个下午左右的时间，大概完成了。主要是生成颜色区域，判断手指移动所在的区域，这两块花了多点的时间。而且用到了三角函数，还有一些坐标计算啊，数学都已经还给了老师，要多写下这种demo，顺便来复习下数学。
可以用canvas来处理的东西太多了，看来是要花更多的时间在canva这部分，因为确实挺有趣的 :)
