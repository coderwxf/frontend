Flex 布局是轴线布局，只能指定“项目”针对轴线的位置，可以看作是一维布局

[Grid](https://css-tricks.com/snippets/css/complete-guide-grid/) 布局则是将容器划分 成“行”和“列”，产生单元格，然后指定“项目所在”的单元格，可以看作是二维布局



## 基本概念

和Flex布局一样，[grid布局](https://enen.me/wp-content/uploads/2021/11/grid.pdf)同样被划分为`grid container`和`grid items`

`grid items`依旧是`grid container`的直接子元素

![LxNJmD.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5be2e3f664564b5ba9363432c735945a~tplv-k3u1fbpfcp-zoom-1.image) 

![LxNuO4.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/783d36b8bd9f46e9bef690ac6865717f~tplv-k3u1fbpfcp-zoom-1.image) 

![LxNSd6.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/78bb303315f5469398c34ec20ab28f8b~tplv-k3u1fbpfcp-zoom-1.image) 

每个grid布局，有隐藏的网格线，用来帮助定位

![LxNrCE.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ef9bd20d4da446a8d4d184af2c55c67~tplv-k3u1fbpfcp-zoom-1.image) 

## 容器属性

### grid-template-*

![LxNF1h.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/59c62b7f5eb5422587611c9b60e079cf~tplv-k3u1fbpfcp-zoom-1.image) 

```css
.container {
  /*
  	开启grid
    grid --- 块级元素
  	inline-grid --- 行内级元素
  */
  display: grid;

  /*
    单单设置display: grid是不会起作用的
    需要结合grid-template-columns或grid-template-rows一起使用

    只设置了grid-template-columns --- 设置网格有多少列，及其对应的排列方式
    只设置了grid-template-rows --- 设置网格有多少行，及其对应的排列方式
  */

  /* 表示三列 每一列的宽度为100px */
  grid-template-columns: 100px 100px 100px;
  
  /* 表示有四行 每一行的高度为100px */
  grid-template-rows: 100px 100px 100px 100px;
  width: 600px;
  height: 600px;
  background-color: skyblue;
}

.item {
  width: 100px;
}
```



#### repeat函数

```css
/* repeat() --- 第一个参数是重复的次数，第二个参数是所要重复的值 */
/*
  grid-template-columns: 100px 100px 100px;
  等价于
	grid-template-columns: repeat(3, 100px);
*/
grid-template-columns: repeat(3, 100px);
```



#### auto-fill

auto-fill 由浏览器自主决定可以划分成多少个单元格

```css
grid-template-columns: repeat(auto-fill, 100px);
```



####  fr

fr，为了方便表示比例关系，网格布局提供了fr关键字（fraction 的缩写，意为"片段"）

```css
/* 宽度平均分成3份 */
grid-template-columns: repeat(3, 1fr);
```



#### minmax()

 minmax()，函数产生一个长度范围，表示长度就在这个范围之中，它接受两个参数，分别为最小值和最大值

```css
grid-template-columns: repeat(2, minmax(150x, 1fr))
```

![image-20230514145631326](https://s2.loli.net/2023/05/14/biCHsMKAlTGhBZD.png) 



#### auto

auto，表示由浏览器自己决定长度

```css
/* 此时auto的作用和fr类似 */
grid-template-columns: repeat(3, auto);

/* 此时auto表示宽度为元素内容的宽度，剩余的宽度由1fr的那个元素占据 */
grid-template-columns: auto auto 1fr;
```

![image-20230514145700246](https://s2.loli.net/2023/05/14/bVXnkpr9RfI8LPD.png) 



#### 网格线

用于定义网格线的名称

```css
/* 三个单元格 --> 四条网格线 */
grid-template-columns: [c1] 100px [c2] 100px [c3] 100px [c4];
```



### gap

gap用于设置item（项目）相互之间的距离

| 名称         | 说明                              |
| ------------ | --------------------------------- |
| `column-gap` | 列单元格与列单元格之间的间距      |
| `row-gap`    | 行单元格与行单元格之间的间距      |
| `gap`        | `column-gap`和`row-gap`的简写属性 |

```css
/*
  column-gap: 10px;
  row-gap: 10px;

  等价于 gap: 10px 10px;

  当column-gap的值和row-gap的值一样的时候可以简写成一个 -> gap: 10px;
*/
gap: 10px;
```



### grid-template-areas

一个区域由单个或多个单元格组成，一个区域中可以有多个单元格组成，也可以仅仅只有一个单元格组成

区域的命名会影响到网格线，每个区域的起始网格线， 会自动命名为区域名-start，终止网格线自动命名为区 域名-end

```css
/* container是一个 3 * 3 的grid容器 */

/* 每一个单元格一个区域 */
grid-template-areas: 'a b c' 'd e f' 'g h i';

/* 为了可读性 一般写成如下形式 */
grid-template-areas:
  'a b c'
  'd e f'
  'g h i';
```

```css
/* 一行为一个区域 */
grid-template-areas:
  'a a a'
  'b b b'
  'c c c';
```

```css
/* 点表示 该单元格暂时不划分到任何一个area中，先行忽略 */
grid-template-areas:
  'a . a'
  'b . b'
  'c . c';
```



### grid-auto-flow

grid-auto-flow用于设置子元素的排放顺序

```css
grid-template-columns: repeat(3, 1fr);
grid-template-rows: repeat(3, 1fr);

/* grid-auto-flow 默认值为 row */
grid-auto-flow: column;
```

![LxN2H8.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f7e3e7d118914ed58ff806e83b629fa3~tplv-k3u1fbpfcp-zoom-1.image) 

![LxNCZk.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/eb86c4b92dc242e5bb551c920b36435a~tplv-k3u1fbpfcp-zoom-1.image) 



### justify-items/align-items

用于设置所有单元格中元素的排列方式

```shell
# 默认值为 stretch
justify-items: start | end | center | stretch;

# 默认值为 stretch
align-items: start | end | center | stretch;
```



place-items属性是align-items属性和justify-items属性的合并简写形式

```shell
# 注意: place-items在设置的时候，第一个值是设置垂直方向的值，第二个值才是设置水平方向上的值
place-items: <align-items> <justify-items>;
```

```css
/* item在水平和垂直方向上全部居中对齐 */
place-items: center;
```



### justify-content/align-content

设置整个内容区域的水平和垂直的对齐方式

```shell
# 默认值 start
justify-content: start | end | center | stretch | space-around | space-between | space-evenly;

# 默认值 start
align-content: start | end | center | stretch | space-around | space-between | space-evenly;
```

![LxNTe5.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/efc680c61a7149c0be2e25af4a7d37f7~tplv-k3u1fbpfcp-zoom-1.image) 



### grid-auto-columns / grid-auto-rows 

用来设置多出来的项目宽和高

```css
/* 设置行超出元素的高度 */
grid-auto-rows: 50px;

/* 设置列超出元素的宽度 */
grid-auto-columns: 80px;
```

![LxNiKt.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/743e2c121ec04b78a0a84d4c7324c3b7~tplv-k3u1fbpfcp-zoom-1.image) 

 

## 项目属性

### grid-column-start / grid-column-end / grid-row-start / grid-row-end

grid-column-start / grid-column-end / grid-row-start / grid-row-end 用来指定item的具体位置是根据哪几根网格线来进行定位的

![LxSnKT.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b6c24703f0d04f16819c30ffaf86437b~tplv-k3u1fbpfcp-zoom-1.image) 

```css
grid-column-start: 1;
grid-column-end: 3;

/* 可以简写为 */
grid-column: 1 / 3;

/* 等价于 */
grid-column-start: span 2;

/* 等价于 */
grid-column-end: span 2;

/* 同理 */
grid-row-start: 1;
grid-row-end: 3;

/* 可以简写为 */
grid-row: 1 / 3;

/* 等价于 */
grid-row-start: span 2;

/* 等价于 */
grid-row-end: span 2;

/* 如果给网格线起了别名 那么在这里也可以使用网格线的别名来进行标识 */
grid-column-start: c1;
grid-column-end: c3;
```



### grid-area

指定项目放在哪一个区域

![LxSYiy.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/e47905144892431ab453e422750905d9~tplv-k3u1fbpfcp-zoom-1.image) 

```css
grid-column-start: 1;
grid-column-end: 3;
grid-row-start: 1;
grid-row-end: 3

/* 可以简写为 */
grid-area: 1 / 1 / 3 / 3;
```



### justify-self / align-self / place-self

justify-self永远设置单个单元格中内容的排列方式。可取值与justify-items的可取值完全一致

```shell
justify-self: start | end | center | stretch;

align-self: start | end | center | stretch;

# place-self属性是align-self属性和justify-self属性的合并简写形式 
# 同样第一个值是设置垂直方向上的值，第二个值才是设置水平方向上的值
place-self: <align-self> <justify-self>;
```
