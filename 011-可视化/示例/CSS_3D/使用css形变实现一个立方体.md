需求使用纯css的3D形变搭建一个类似如下图的立方体

![image.png](https://s2.loli.net/2022/09/25/TesnIbx1wiLPO7K.png)

思路：

1. 定义一个舞台，所有的需要形变的元素都在舞台上进行对应的形变，也就是避免形变元素对标准流中的布局元素产生影响

2. 对舞台进行整体的形变，从而修改整个舞台中形变元素的参考坐标系，方便子元素进行调整
3. 在对整个舞台进行形变的过程中，不需要设置透视距离，也就是不应该出现近大远小的效果



## 基础结构搭建

`html`

```html
<div class="container">
  <!-- 定义舞台 - 也就是需要执行形变元素的父元素 -->
  <div class="stage">
    <!--
			这里的文本div和子元素的文本1~6的作用是一致的
			也就是当元素形变后，可以通过文本内容迅速找到对应的元素左上角
			因为只要元素发生形变，对应的坐标轴就会发生改变
			所以可以通过该文本，顺序识别形变后的坐标轴，方便调试
		-->
    <span>div</span>
    <!-- 立方体的六个面 -->
    <div class="slide top">1</div>
    <div class="slide bottom">2</div>
    <div class="slide front">3</div>
    <div class="slide back">4</div>
    <div class="slide left">5</div>
    <div class="slide right">6</div>
  </div>
</div>
```



`css`

```css
body {
  margin: 0;
  padding: 0;
}

.container {
  width: 600px;
  height: 600px;
  margin: 100px auto;
  display: flex;
  align-items: center;
  justify-content: center;
  background: url(./grid.png);
}

/* 所有的动效都是在舞台中执行的，和外部元素无关，这有点类似于开启BFC的元素的作用 */
.stage {
  position: relative;
  width: 200px;
  height: 200px;
  background-color: red;
}

/*
子元素和父元素一样大，并使用定位和父元素左上角重合
可以有效的避免元素因为形变而对标准流中的元素产生影响
同时当执行动效的时候，绝对定位元素会单独使用一个合成图层进行渲染
可以提升渲染速度
*/
.slide {
  position: absolute;
  top: 0;
  left: 0;
  width: 100px;
  height: 100px;
}
```



## 绘制侧视图

```css
.stage {
  position: relative;
  width: 200px;
  height: 200px;
  background-color: red;
  /* 为子元素开启3D空间 */
  transform-style: preserve-3d;
  /* 形成切面图，因为不需要近大远小的效果，所以不需要设置透视距离 */
  transform: rotateX(-33.5deg) rotateY(45deg);
}
```

![image.png](https://s2.loli.net/2022/09/25/JKjM3N6aOAxuZnC.png)



## 根据侧视图，绘制立方体的六个面

```css
.top {
  background-color: rgba(255, 0, 0, 0.4);
  transform: rotateX(90deg) translateZ(100px);
}

.bottom {
  background-color: rgba(0, 255, 0, 0.4);
  transform: rotateX(-90deg) translateZ(100px);
}

.front {
  background-color: rgba(0, 0, 255, 0.4);
  transform: translateZ(100px);
}

.back {
  background-color: rgba(255, 255, 0, 0.4);
  transform: rotateX(180deg) translateZ(100px);
}

.left {
  background-color: rgba(255, 0, 255, 0.4);
  transform: rotateY(-90deg) translateZ(100px);
}

.right {
  background-color: rgba(125, 100, 144, 0.4);
  transform: rotateY(90deg) translateZ(100px);
}
```

[纯css实现立方体代码演示](https://code.juejin.cn/pen/7147173824562200584)



### 对立方体的一些其它调整

`如果对某个元素进行形变，那么其中的子元素也会产生对应的形变效果`

所以如果我们对舞台进行缩放的时候，对应的子元素也会实现缩放效果，从而实现整个立方体的缩放

```css
/* 将Y轴放大2倍，此时对应的正方体将会被转变为立方体 */
transform: rotateX(-33.5deg) rotateY(45deg) scaleY(2);
```

![image.png](https://s2.loli.net/2022/09/25/Snbakyzs9C2W6LR.png)



```css
/*
	scale(2) 本质上 是将舞台的x轴和y轴 同时扩大2倍
	所以如果此时同时将z轴也扩大2倍
	就可以实现将整个立方体放大2倍的效果
*/
transform: rotateX(-33.5deg) rotateY(45deg) scale(2) scaleZ(2);
```

![image.png](https://s2.loli.net/2022/09/25/6PFDhCgIAMwlNE4.png)