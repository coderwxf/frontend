## 行盒

在css中，盒子是由其内部内容的高度来撑开的

![LT49qr.png](https://s6.jpg.cm/2022/04/21/LT49qr.png) 

但是我们可以看到盒子和文本之间有一定的间隙，而这些间隙本质上是由于行高导致的

所以内容为文本的盒子本质上是由文本的行高撑起来的

但如果有多行文本的是，浏览器是怎么布局的呢？

本质上，浏览器在布局行内级元素和行内块级元素的时候，会将他们放置到一个虚拟的盒子中进行排列

而`这个虚拟的盒子就被称之为行盒(line box)`

`行盒的作用是将当前行里面所有的内容全部包裹在内`

![LT4TH5.png](https://s6.jpg.cm/2022/04/21/LT4TH5.png) 

在多行文本中每一行，在浏览器渲染的时候，就会存在一个行盒，行盒会将其中所有行内级元素和行内块级元素包裹在内



默认情况下，行内级元素和行内块级元素在行盒中水平排列的时候，是按照`基线对齐`的方式进行排列的

1. 对于文本，其基线位于小写字母x的底部
2. 对于行内替换元素和没有文本的行内块级元素，其基线位于元素的margin-bottom的底部
3. 对于有内容的行内块级元素，其基线位于最后一行内容的基线所在的位置（无论该内容是文本还是其它元素）



```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    .box {
      margin: 100px auto;
      width: 300px;
      background-color: #eee;
    }

    .inner {
      width: 100px;
      height: 100px;
      display: inline-block;
      background-color: skyblue;
    }
  </style>
</head>
<body>
  <div class="box">
    bbbxxxpppggg
    <div class="inner"></div>
  </div>
</body>
</html>
```

![LTBRqD.png](https://s6.jpg.cm/2022/04/21/LTBRqD.png) 

因为在行盒中，多个元素的对齐方式默认是基线对齐

所以当文本元素和行内替换元素对齐或者是文本元素和行内块级元素对齐的时候

会发现行内替换元素或行内块级元素在底部会存在一定像素的间隙

这是因为文本元素的基线位于小写x的底部，而类似于字母p或g之类的字母会在下方有一部分内容会低于小写x的底部

所以需要为其预留一定的空间，因此此时行内块级元素或行内替换元素在进行排列的时候

在其底部会存在一定像素的间隙



![LTBVK2.png](https://s6.jpg.cm/2022/04/21/LTBVK2.png) 

此时即使没有文本和行内替换元素或行内级元素对齐，其也会为以后可能存在的文本对齐情况，在元素的底部预留一定的间隙



## vertical-align

vertical-align是用来设置行内块级元素 在一个 行盒 中垂直方向的位置

也就是说`vertical-align`是设置在行内级或行内块级元素上的

| 值             | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| baseline       | 基线对齐 --- 默认值                                          |
| top            | 行内级盒子的顶部跟line boxes顶部对齐                         |
| bottom         | 行内级盒子的底部跟line box底部对齐                           |
| middle         | 行内级盒子的中心点与小写字母x的中心线对齐                    |
| `<percentage>` | 把行内级盒子提升或者下降一段距离(距离相对于line-height计算\元素高度)<br />0%意味着同baseline一 样 |
| `<number>`     | 把行内级盒子提升或者下降一段距离，0cm意味着同baseline一样    |



所以解决之前行内替换元素和行内块级元素的下边缘的间隙方法是:

1. 设置`vertical-align`的值为`top/middle/bottom`
2. 将元素的`display`属性设置为`block`



### 注意点

1. `vetical-align:middle`不能实现垂直居中对齐

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    .container {
      width: 300px;
      height: 300px;
      /* 将行盒的高度设置为和div.container的高度一致 */
      line-height: 300px;
      background-color: gray;
    }

    .box {
      width: 100px;
      height: 100px;
      background-color: red;
      display: inline-block;
      /* 设置行内块级元素div.box在父级盒子中的对齐方式 */
      vertical-align: middle;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="box"></div>
  </div>
</body>
</html>
```

![LTyEXe.png](https://s6.jpg.cm/2022/04/21/LTyEXe.png) 

注意:

上述代码`并不能使div.box在div.container中在垂直方向上真正居中对齐`

因为`vertical-align: middle本质是和小写字母的x的中心点对齐`

而在实际中，`大写X的中心点才是正好和中线重合，小写x的中心点是略低于中线的`

![](https://iknow-pic.cdn.bcebos.com/b3119313b07eca80a3ad98e29c2397dda144830f) 

而且绝大部分的字体在实际设计的时候，为了美观，可能会对字体进行略微的下沉处理

因此上述代码不能够真正意义上使div.box在div.container中垂直居中对齐

只能做到近似的垂直居中对齐



2. 使用`vertical-align`实现垂直居中

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    .container {
      width: 300px;
      /* div.container中的文本垂直居中对齐  */
      height: 300px;
      line-height: 300px;
      background-color: gray;
    }

    .box {
      width: 100px;
      /* div.box中的文本垂直居中对齐 */
      height: 100px;
      line-height: 100px;
      background-color: skyblue;
      display: inline-block;
    }
  </style>
</head>
<body>
  <div class="container">
    containerxxxx
    <div class="box">loremxxxxx</div>
  </div>
</body>
</html>
```

![LT9mUS.png](https://s6.jpg.cm/2022/04/21/LT9mUS.png)  

此时div.box在div.container中是真正意义上垂直居中对齐的

因为此时div.container中的字体大小和div.box中的字体大小是一致的

且在div.box中 line-height === height

在div.container中 line-height === height

所以此时div.box在div.container中是真正意义上垂直居中对齐



但是这么做的局限性很大

1. 父盒子和子盒子中都必须满足height等于line-height
2. 父盒子和子盒子中必须存在单行文本且只能是单行文本
3. 子盒子中的字体大小必须要等于父盒子中的字体大小



![LTT6MW.png](https://s6.jpg.cm/2022/04/21/LTT6MW.png) 

如果div.box和div.container中字体大小不一致的时候

为了保证这两个元素是基线对齐，所以div.box无法实现在父元素中垂直居中对齐

例如上图中div.box的font-size大于div.container中的font-size

所以div.box中的字体会上浮，最终div.box距离上边的距离会小于其距离下边的距离



3. `当line-height大于height的时候，内容会下沉，反之内容会上浮`

  ```html
  <!DOCTYPE html>
  <html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
    <style>
      .container {
        width: 300px;
        height: 300px;
        background-color: gray;
      }
  
      .box {
        width: 100px;
        /*
          line-height > height内容下沉
          并导致内容抛出div.box的内容区域
          也就是行盒的高度远大于div.box的高度
  
          此时跑出去的文本在标准流中依旧是占据一定位置的
          所以会将div.foo往下挤
        */
        height: 100px;
        line-height: 400px;
        background-color: skyblue;
        display: inline-block;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="box">lorem</div>
      <div class="foo">Lorem ipsum dolor sit amet consectetur adipisicing elit. Nemo, dolore.</div>
    </div>
  </body>
  </html>
  ```

