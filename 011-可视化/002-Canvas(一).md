Canvas 最初由Apple于2004 年引入，用于Mac OS X WebKit组件，同时给Safari浏览器等应用程序提供支持

后来，它被 Gecko内核的浏览器(尤其是Mozilla Firefox)，Opera和Chrome实现，并被W3C提议为下一代的标准元素(HTML5新增 元素)

Canvas提供了非常多的JavaScript绘图API，也就是说canvas的绘制是通过命令式的方式进行绘制的

同这些canvas api 可以将2D图形绘制在canvas元素(画布)上

Canvas API 主要用于绘制 2D 图形。当然也可以使用 Canvas 提供给的 WebGL API 来绘制 3D 图形

Canvas可以用于动画、游戏画面、数据可视化、图片编辑以及实时视频处理等方面



操作Canvas只能通过JavaScript脚本，Canvas能够以 .png 或 .jpg 格式保存图像，而Svg 只能保存成以svg作为文件后缀名的文件

在移动端，使用Canvas是很耗性能的操作，如果在移动端使用Canvas过多，会使内存占用超出了手机承受范围，可能会导致浏览器崩溃

Canvas绘制出的图形是由像素点组成的位图，所以Canvas更适合于进行那些像素级别的操作，然而Canvas绘制出来的图片是位图，所以图片放大会失真

Svg（Scalable Vector Graphics 可伸缩矢量图形）严格遵从XML语法，并用文本格式的描述性语言来描述图像内容，因此是一种和图像分辨率无关的矢量图形格式，所以SVG放大并不会失真，而且SVG对文字具有更好的处理能力



因此Canvas可以看成是一个拥有绘图API的HTML5，在 canvas 中，一旦图形被绘制完成，它就不会继续得到浏览器的关注。如果其位置发生变化，那么整个场景也需要重新绘制，包括任何或许已被图形覆盖的对象，因此Canvas非常适合图像密集型的游戏开发，因为游戏开发需要频繁重绘大量的图形对象



SVG 基于 XML， 这意味着 SVG 中的每个元素都是可用，所以对于SVG而言，可以对某个画面元素进行单独操作，如进行事件处理

所以SVG更适合于那些对精度要求比较高的图形绘制，如地图和图表等

而Canvas更适合于那些需要基于动态数据进行动态渲染的或者需要频繁重绘的图形的绘制，因为这些图形需要频繁重绘，就需要操作DOM，而canvas只有画布一个DOM元素，而SVG中的每一个元素都是一个DOM元素，所以Cavans更适合于那些需要频繁重绘的图形



## 基本使用

`<canvas>`除了id, class等公共属性外

`canvas`标签只有两个属性 - width和height( 单位不需要写，默认为px )

canvas是是行内替换元素，没宽高时，canvas 会初始化宽为 300px 和高为 150px

`<canvas>` 元素必须需要结束标签 (`</canvas>`)

如果结束标签不存在，则文档其余部分会被认为是替代内容，将不显示出来

```html
<canvas id="canvas" width="1000" height="1000">
  当canvas元素不被浏览器支持的时候，会显示canvas标签中的内容
  不过现代浏览器都是支持canvas元素的
</canvas>

<script>
  window.onload = () => {
    // 获取canvas元素
    const canvasEl = document.getElementById('canvas')

    // 通过canvas获取对应的上下文(画笔) - ctx (CanvasRenderingContext2D的缩写)
    // 绘图上下文(ctx) 本质上是一个对象，该对象提供了在canvas中用于绘制的属性和方法

    // 当绘制2D图形的时候，getContext的参数为2d
    // 当绘制3D图形的时候，getContext的参数为webgl
    const ctx = canvasEl.getContext('2d')

    // canvas的坐标原点也是在左上角
    // 下述指令的含义是在以(0, 0)作为矩形的左上角
    // 绘制一个宽200px，高200px的矩形
    // ps: 不需要书写对应的单位，单位默认是px
    ctx.fillRect(0, 0, 200, 200)
  }
</script>
```



## 坐标系统

Canvas存在对应网格的概念，网格也可理解为是坐标空间(坐标系)

通常来说网格中的一个单元相当于 canvas 元素中的一像素

网格的原点位于坐标 (0,0) 的左上角(被称为初始坐标系)。所有图形都相对于该原点绘制

移动、旋转、缩放坐标系后，都会修改坐标系，默认所有后续变换都将基于新坐标系的变换

![image.png](https://s2.loli.net/2022/10/01/xXAaMLS74lWeVym.png)



## 绘制矩形

| 方法                            | 说明                                 |
| ------------------------------- | ------------------------------------ |
| fillRect(x, y, width, height)   | 绘制一个填充的矩形                   |
| strokeRect(x, y, width, height) | 绘制一个矩形的边框                   |
| clearRect(x, y, width, height)  | 清除指定矩形区域，让清除部分完全透明 |

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  // 从0,0 点为左上角 绘制一个宽200 高200的填充矩形
  // 默认填充和描边色 皆为黑色
  ctx.fillRect(0, 0, 200, 200)

  // 从300, 300 为矩形的左上角 绘制一个宽200 高200的描边矩形
  // 如果绘制的图形 超出了canvas的部分 那么超出的部分会被裁剪
  ctx.strokeRect(300, 300, 200, 200)

  // 清除矩形区域 --- 类似于画笔的功能
  // 如果宽高和canvas的宽高一致的时候 可以认为是清空整个画布
  ctx.clearRect(0, 0, 1000, 1000)
}
```



## 路径

图形的基本元素是路径。路径是点列表，由线段连接

这些线段可以具有不同形状、弯曲或不弯曲的、连续或不连续的、不同的宽度和颜色

路径是可由很多子路径构成，这些子路径都是在一个列表中，列表中所有子路径(线、弧形等)将构成图形



使用路径绘制图形的步骤:

1. 首先需要创建路径起始点(beginPath)
2. 使用绘图命令去画出路径( arc 、lineTo )
3. 把路径闭合( closePath , 虽然不是必须， 但是推荐加上)
4. 默认路径是透明不可见的，所以一旦路径生成，就需要通过描边(stroke)或填充路径区域(fill)来渲染图形



| 方法      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| beginPath | 新建一条路径，生成之后，图形绘制命令被指向到新的路径上绘图，不会关联到旧的路径<br />在实际绘制的过程中，会存在一个列表记录这之前所有绘制的路径, 这个列表中的路径就被称之为子路径<br />当我们调用beginPath后，会直接清空这个路径列表中的所有子路径 |
| closePath | 闭合路径，即从当前路径绘制一条直线到起始路径                 |
| stroke    | 通过线条来绘制图形描边(针对当前路径图形)                     |
| fill      | 通过填充路径的内容区域生成实心的图形(针对当前路径图形)       |



### 绘制直线

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  // 开启路径的绘制
  ctx.beginPath()

  // 设置画笔的初始点
  // 如果初始化Canvas后, 对应画笔所在位置，为上一个path的结束位置
  // 而对于第一次绘制的时候，并没有上一次path结束位置，所以需要使用moveTo来进行指定
  // 同样我们可以通过移动画笔 来绘制一些不连续的路径
  ctx.moveTo(10, 10)

  // 从10,10 到100,10 之间绘制一条线
  ctx.lineTo(100, 10)

  ctx.closePath()

  // 线是没有内容区域的，对于线而言只有边
  // 所以对于线而言，只能使用stroke，不可以使用fill
  ctx.stroke()
}
```



#### 绘制三角形

![image.png](https://s2.loli.net/2022/10/01/8h9waPpNRzLnbBA.png)

```js
ctx.beginPath()

ctx.moveTo(50, 50)
ctx.lineTo(100, 100)
ctx.lineTo(50, 150)

// stroke并不会主动进行路径闭合
ctx.stroke()
```



![image.png](https://s2.loli.net/2022/10/01/o6I9lnuZ423Vrgm.png)

```js
ctx.beginPath()

ctx.moveTo(50, 50)
ctx.lineTo(100, 100)
ctx.lineTo(50, 150)

ctx.closePath()

ctx.stroke()
```



![image.png](https://s2.loli.net/2022/10/01/tKm7H9BiqozI8D5.png)

```js
ctx.beginPath()

ctx.moveTo(50, 50)
ctx.lineTo(100, 100)
ctx.lineTo(50, 150)

// 在进行填充的时候，fill也不会主动闭合路径
// 所以此时画笔依旧在 50, 150
// 但是因为此时使用了填充，所以浏览器会模拟路径的闭合 并进行颜色的填充
ctx.fill()
```



### 绘制圆弧

| 方法                                                   | 说明                                                         |
| ------------------------------------------------------ | ------------------------------------------------------------ |
| arc(x, y, radius, startAngle, endAngle, anticlockwise) | 1. x、y:圆心坐标<br />2. radius:圆弧半径<br />3. startAngle、endAngle:指定开始 和 结束的弧度。以 x 轴为基准(注意:单位是弧度，不是角度) <br />4. anticlockwise:为一个布尔值。为 true ，是逆时针方向，为false，是顺时针方向，默认为false |



#### 弧度和角度

**弧度**(英语:radian)，是平面角的单位。1单位弧度为: 圆弧长度等于半径时的圆心角，而一个完整的圆的弧度是 Math.PI * 2，半圆弧度是 Math.PI

![image.png](https://s2.loli.net/2022/10/01/rJhsHWiMnw7BczY.png)

角度与弧度的 JS 表达式:弧度 = ( Math.PI / 180 ) * 角度 ，即 1角度= Math.PI / 180 个弧度

比如:  旋转90° -> Math.PI / 2;   旋转180° -> Math.PI ;  旋转360° -> Math.PI * 2;  旋转-90° -> -Math.PI / 2



#### 绘制圆

```js
ctx.beginPath()

ctx.arc(50, 50, 50, 0, 2 * Math.PI, false)

ctx.stroke()
```



#### 绘制圆弧

```js
ctx.beginPath()

ctx.arc(50, 50, 50, 0, Math.PI, false)

ctx.stroke()
```



#### 注意事项

![image.png](https://s2.loli.net/2022/10/01/KiRwQWE3jDpcBqd.png)

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.beginPath()

  ctx.arc(50, 50, 50, 0, 2 * Math.PI, false)

  ctx.stroke()

  // 因为没有重新调用beginPath方法，所以本质上依旧在一个path中
  // 没有执行moveTo方法移动画笔，所以会被认为连续的图形
  // 因此在描边的时候，会将上一个圆的终点和半圆的起点相连
  ctx.arc(150, 150, 50, 0, Math.PI, false)

  ctx.stroke()
}
```



`解决方法 - moveTo`

![image.png](https://s2.loli.net/2022/10/01/C7NDA2Byu4dwV8R.png)

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.beginPath()

  ctx.arc(50, 50, 50, 0, 2 * Math.PI, false)

  ctx.stroke()

  // 将画笔 瞬移到下一个弧度开始的位置
  ctx.moveTo(200, 150)
  ctx.arc(150, 150, 50, 0, Math.PI, false)

  ctx.stroke()
}
```



`解决方法 - beginPath`

![image.png](https://s2.loli.net/2022/10/01/C7NDA2Byu4dwV8R.png)

```js
window.onload = () => {
  const canvasEl = document.getElementById('canvas')

  const ctx = canvasEl.getContext('2d')

  ctx.beginPath()

  ctx.arc(50, 50, 50, 0, 2 * Math.PI)

  ctx.stroke()

  ctx.beginPath()
  ctx.arc(150, 150, 50, 0, Math.PI)

  ctx.stroke()
}
```



#### 绘制矩形

除了可以使用fillRect和stokeRect这类的API来绘制对应的图形，也可以使用路径来绘制对应的图形

此时就需要调用Rect方法，将一个矩形路径增加到当前路径上

rect(x, y, width, height) --> 绘制一个左上角坐标为(x,y)，宽高为 width 、 height 的矩形

当该方法执行的时候，moveTo(x, y) 方法自动设置坐标参数(0,0), 也就是说调用Rect方法前，会默认将画笔重置为0,0 坐标点

```js
ctx.beginPath()

ctx.rect(100, 100, 200, 200)

ctx.stroke()
```