## 基本结构搭建

`html`

```html
<div class="container">
  <div class="cube inner">
    <div class="slide top">1</div>
    <div class="slide bottom">2</div>
    <div class="slide left">3</div>
    <div class="slide right">4</div>
    <div class="slide front">5</div>
    <div class="slide back">6</div>
  </div>

  <div class="cube outer">
    <div class="slide top">1</div>
    <div class="slide bottom">2</div>
    <div class="slide left">3</div>
    <div class="slide right">4</div>
    <div class="slide front">5</div>
    <div class="slide back">6</div>
  </div>
</div>
```



css

```css
html,
body {
	padding: 0;
	margin: 0;
	width: 100%;
	height: 100%;
	background-color: #2b3a42;
}

.container {
	width: 100%;
	height: 100%;
	/*
		需要使用flex布局进行居中显示
		不要使用transform来进行跳转
		因为使用transform后，transform-origin会被改变，对应的坐标轴也会改变
	*/
	display: flex;
	justify-content: center;
	align-items: center;
}


.cube {
	position: absolute;
  /* 设置立方体的侧视图 */
  transform: rotateX(-33.5deg) rotateY(45deg);
  transform-style: preserve-3d;
}

.inner {
	width: 100px;
	height: 100px;
}

.outer {
	width: 200px;
	height: 200px;
}

.slide {
	position: absolute;
	top: 0;
	left: 0;
	width: 100%;
	height: 100%;
  border: 1px solid #fff;
}

.inner .slide {
  background-color: #175d96;
}

.outer .slide {
	background-color: rgba(141, 214, 249, 0.5);
}
```



## 绘制立方体

```css
.inner .slide {
  background-color: #175d96;
	--offset: 50px;
}

.outer .slide {
	background-color: rgba(141, 214, 249, 0.5);
	--offset: 100px
}

.top {
	transform: rotateX(90deg) translateZ(var(--offset));
}

.bottom {
	transform: rotateX(-90deg) translateZ(var(--offset));
}

.front {
	transform: translateZ(var(--offset));
}

.back {
	transform: translateZ(calc(-1 * var(--offset)));
}

.left {
	transform: rotateY(90deg) translateZ(var(--offset));
}

.right {
	transform: rotateY(-90deg) translateZ(var(--offset));
}
```

![image.png](https://s2.loli.net/2022/09/25/w4jqmdQ1X78nNHy.png)



## 添加旋转动画

如果需要让整个立方体进行旋转，只要让立方体所在的舞台进行旋转即可，其内部的所有的元素也就会相应的产生对应的旋转

在绘制立方体的时候，对舞台进行了transform转换，所以此时对应的舞台的坐标轴为如下结构

![image.png](https://s2.loli.net/2022/09/25/Otz31Q5wuaS9Up6.png)



```css
.inner {
	width: 100px;
	height: 100px;
	animation: innerLoop 6s ease-in-out infinite;
}

.outer {
	width: 200px;
	height: 200px;
	animation: outerLoop 6s ease-in-out infinite;
}

@keyframes innerLoop {
  /*
  	0% ~ 50% 的时候 transform: rotateX(-33.5deg) rotateY(45deg);
  	50% ~ 100% 的时候 保持不动
    也就是说 使用3s旋转一圈后 停留3s不动
  */

	50%, 100% {
		transform: rotateX(-33.5deg) rotateY(45deg) rotateY(360deg);
	}
}

/*
	这里不能使用animation的backwards属性
	backwards属性的值是 动画执行轨迹从 100% 反向往 0% 进行执行
	而这里只是旋转方向不一致，并不是要动画从100%往0%进行旋转
*/
@keyframes outerLoop {
	50%, 100% {
		transform: rotateX(-33.5deg) rotateY(45deg) rotateY(-360deg);
	}
}
```



[webpack logo最终实现效果样式](https://code.juejin.cn/pen/7147207854376615967)