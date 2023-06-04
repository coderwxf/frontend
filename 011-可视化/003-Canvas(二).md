## colors

fillStyle = color: 设置图形的填充颜色，需在 fill() 函数前调用

strokeStyle = color: 设置图形轮廓的颜色，需在 stroke() 函数前调用



color 可以是表示 CSS 颜色值的字符串，支持:关键字、十六进制、rgb、rgba格式

默认情况下，线条和填充颜色都是黑色(CSS 颜色值 #000000)



一旦设置了 strokeStyle 或者 fillStyle 的值，那么这个新值就会成为新绘制的图形的默认值

如果要给图形上不同的颜色，你需要重新设置 fillStyle 或 strokeStyle 的值

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.beginPath()
  ctx.fillStyle = 'red'
  ctx.rect(100, 100, 100, 100)
  ctx.fill()


  ctx.beginPath()
  ctx.strokeStyle = '#f0f'
  ctx.arc(400, 200, 100, 0, Math.PI)
  ctx.stroke()
}
```



## transparent

除了可以绘制实色图形，我们还可以用 canvas 来绘制半透明的图形



`方式一  --- strokeStyle 和 fillStyle属性结合RGBA`

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.beginPath()
  ctx.fillStyle = 'rgba(255, 0, 0, 0.4)'
  ctx.rect(100, 100, 100, 100)
  ctx.fill()
}
```



`方式二 --- globalAlpha 属性`

globalAlpha = 0 ~ 1

+ 这个属性影响到 canvas 里所有图形的透明度
+ 有效的值范围是 0.0(完全透明)到 1.0(完全不透明)，默认是 1.0

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.globalAlpha = 0.4

  ctx.beginPath()
  ctx.fillStyle = 'red'
  ctx.rect(100, 100, 100, 100)
  ctx.fill()
}
```





## line styles

调用lineTo()函数绘制的线条，是可以通过一系列属性来设置线的样式

| 属性              | 说明                         |
| ----------------- | ---------------------------- |
| lineWidth = value | 设置线条宽度                 |
| lineCap = type    | 设置线条末端样式             |
| lineJoin = type   | 设定线条与线条间接合处的样式 |



### line-width

设置线条宽度的属性值必须为正数。默认值是 1px，不需单位。( 零、负数、Infinity和NaN值将被忽略)

线宽是指给定路径的中心到两边的粗细。换句话说就是在路径的两边各绘制线宽的一半



如果想要绘制一条从 (3,1) 到 (3,5)，宽度是 1px 的线条

![image.png](https://s2.loli.net/2022/10/01/GEhJzit5NcRsWdm.png)

+ 路径的两边个各延伸半个像素填充并渲染出1px像素的线条(深蓝色部分)
+ 两边剩下的半个像素又会以实际画笔颜色一半色调来填充(浅蓝部分)
+ 实际画出线条的区域为(浅蓝和深蓝的部分)，填充色大于1px了，这就是为何宽度为 1px 的线经常并不准确的原因



要解决这个问题，必须对路径精确的控制

如，1px的线条会在路径两边各延伸半像素

那么从 (3.5 ,1) 到 (3.5, 5) 的线条，其边缘正好落在像素边界，填充出来就是准确的宽为 1.0 的线条

![image.png](https://s2.loli.net/2022/10/01/g9TZoDLp2ne64Cr.png)



### line-clap

lineCap: 属性的值决定了线段端点显示的样子。它可以为下面的三种的其中之一

+ butt 截断，默认是 butt
+ round 圆形
+ square 正方形

![image.png](https://s2.loli.net/2022/10/01/6lsktxNEeK2bJZO.png)



### line-join

lineJoin: 属性的值决定了图形中线段连接处所显示的样子。它可以是这三种之一

+ round 圆形
+ bevel 斜角
+ miter 斜槽规，默认是 miter 其实就是尖角

![image.png](https://s2.loli.net/2022/10/01/BkS2Tu71lP4aAqi.png)



## 绘制文本

canvas 提供了两种方法来渲染文本

| 方法                                | 说明                                                    |
| ----------------------------------- | ------------------------------------------------------- |
| fillText(text, x, y [, maxWidth])   | 在 (x,y) 位置，填充指定的文本<br />绘制的最大宽度(可选) |
| strokeText(text, x, y [, maxWidth]) | 在 (x,y) 位置，绘制文本边框<br />绘制的最大宽度(可选)   |

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  // font = value: 当前绘制文本的样式。和 CSS font 属性相同的语法。默认的字体是:10px sans-serif
  ctx.font = '60px sans-serif'

  // textAlign = value:文本对齐选项。可选的值包括:start, end, left, right or center. 默认值是 start
  ctx.textAlign = 'center'

  // textBaseline = value:基线对齐选项。可选的值包括:top, hanging, middle, alphabetic, ideographic, bottom
  // 默认值是 alphabetic
  ctx.textBaseline = 'middle'
  ctx.strokeStyle = 'red'

  ctx.strokeText('Hello', 100, 100)
}
```



## 绘制图片

绘制图片，可以使用 drawImage 方法将它渲染到 canvas 里。drawImage 方法有三种形态

| 方法                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| drawImage(image, x, y)                                       | 其中 image 是 图片(image, svg), 视频(当前播放的那一帧)，canvas元素<br />x 和 y 是其在目标 canvas 里的起始坐标 |
| drawImage(image, x, y, width, height)                        | 这个方法多了 2 个参数: width 和 height                       |
| drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight) | sx, sy, sWidth, sHeight - 裁剪部分<br />dx, dy, dWidrh, dHeifht - 绘图部分 |

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  const image = new Image()
  image.src = './images/backdrop.png'

  image.onload = () => {
    ctx.drawImage(image, 0, 0)
  }
}
```



## 绘画状态

Canvas绘画状态是当前绘画时所产生的样式和变形的一个快照

Canvas在绘画时，会产生相应的绘画状态，其实我们是可以将某些绘画的状态存储在栈中来为以后复用

Canvas 绘画状态的可以调用 save 和 restore 方法是用来保存和恢复，这两个方法都没有参数，并且它们是成对存在的

| 方法      | 说明                             |
| --------- | -------------------------------- |
| save()    | 保存画布 (canvas) 的所有绘画状态 |
| restore() | 恢复画布 (canvas) 的所有绘画状态 |

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.fillStyle = 'red'
  ctx.save()
  ctx.fillRect(100, 100, 50, 50)

  ctx.fillStyle = 'green'
  ctx.save()
  ctx.fillRect(200, 100, 50, 50)

  ctx.fillStyle = 'blue'
  ctx.save()
  ctx.fillRect(300, 100, 50, 50)

  ctx.restore()
  ctx.fillRect(300, 170, 50, 140)

  ctx.restore()
  ctx.fillRect(200, 170, 50, 140)

  ctx.restore()
  ctx.fillRect(100, 170, 50, 140)
}
```



## 形变

Canvas和CSS3一样也是支持变形，形变是一种更强大的方法，可以将坐标原点移动到另一点、形变可以对网格进行旋转和缩放。

| 方法            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| translate(x, y) | 用来移动 canvas 和它的原点到一个不同的位置                   |
| rotate(angle)   | 用于以原点为中心旋转 canvas，即沿着z轴旋转<br />注意: 单位是弧度 不是角度 |
| scale(x, y)     | 用来增减图形在 canvas 中像素数目，对图形进行缩小或放大<br />注意: 缩放 会改变原本canvas中像素的大小 |

> 在做变形之前先调用 save 方法保存状态是一个很好的习惯
>
> 因为形变必然会导致canvas中坐标轴体系的改变，这会导致后期的绘图或者坐标轴恢复十分麻烦
>
> 所以保存初始坐标系，可以保证我们的坐标系一直是以canvas的左上角为初始原点

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  // 保存状态
  ctx.save()

  // 位移
  ctx.translate(200, 200)

  // 顺时针 旋转45deg
  ctx.rotate(Math.PI / 180 * 45)

  // 缩放 x轴和y轴同时缩放 - 所以现在的1px 等价于 原来的 2px
  ctx.scale(2, 2)

  // 绘制矩形
  ctx.fillRect(0, 0, 100, 100)

  // 恢复状态
  ctx.restore()
}
```



## 动画

canvas最大的限制就是图像一旦绘制出来，它就是一直保持那样了， 在canvas中没有图层的概念

如需要执行动画，不得不对画布上所有图形进行一帧一帧的重绘

如果需要绘制出流畅的动画，一秒需要绘制60帧，或者说是60个canvas快照

为了实现动画，我们需要使用setInterval 、 setTimeout 和 requestAnimationFrame 这三种方法来定期执行指定函数进行重绘



### 使用定时器绘制动画

```js
// 需求: 模拟时钟的秒针
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  let i = 0

  function draw() {
    i++

    if (i >= 60) i = 0

    // 在每次绘制之前 清空画布
    ctx.clearRect(0, 0, 300, 300)

    // 保存状态
    ctx.save()

    // 旋转画布
    ctx.translate(100, 100)
    ctx.rotate(Math.PI * 2 / 60 * i)

    ctx.beginPath()
    ctx.moveTo(0, 0)
    ctx.lineTo(0, -80)

    ctx.strokeStyle = 'red'
    ctx.lineWidth = 4
    ctx.lineCap = 'round'

    ctx.stroke()

    // 恢复状态
    ctx.restore()
  }

  draw()
  setInterval(() => {
    draw()
  }, 1000)
}
```



### 使用requestAnimationFrame绘制动画

```js
// 需求: 模拟时钟的秒针
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  function draw() {

    ctx.clearRect(0, 0, 300, 300)

    ctx.save()

    ctx.translate(100, 100)

    // 通过当前时间的秒数 控制秒针需要旋转的角度
    ctx.rotate(Math.PI * 2 / 60 * (new Date().getSeconds()))

    ctx.beginPath()
    ctx.moveTo(0, 0)
    ctx.lineTo(0, -80)

    ctx.strokeStyle = 'red'
    ctx.lineWidth = 4
    ctx.lineCap = 'round'

    ctx.stroke()

    ctx.restore()
    requestAnimationFrame(draw)
  }

  draw()
  requestAnimationFrame(draw)
}
```



### 综合案例

1. [太阳系](./%E7%A4%BA%E4%BE%8B/canvas/%E5%A4%AA%E9%98%B3%E7%B3%BB.md)

2. [时钟动画]()
