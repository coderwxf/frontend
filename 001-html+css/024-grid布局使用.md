Flex 布局是轴线布局，只能指定“项目”针对轴线的位置，可以看作是一维布局

[Grid](https://css-tricks.com/snippets/css/complete-guide-grid/) 布局则是将容器划分 成“行”和“列”，产生单元格，然后指定“项目所在”的单元格，可以看作是二维布局



## 基本概念

和Flex布局一样，grid布局同样被划分为`grid container`和`grid items`

![LxNJmD.png](https://s6.jpg.cm/2022/05/03/LxNJmD.png) 

![LxNuO4.png](https://s6.jpg.cm/2022/05/03/LxNuO4.png) 

![LxNSd6.jpg](https://s6.jpg.cm/2022/05/03/LxNSd6.jpg) 

每个grid布局，有隐藏的网格线，用来帮助定位

![LxNrCE.png](https://s6.jpg.cm/2022/05/03/LxNrCE.png) 

## 容器属性

### grid-template-*

![LxNF1h.png](https://s6.jpg.cm/2022/05/03/LxNF1h.png) 

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

    只设置了grid-template-columns，表示列数根据columns个数自动排布，且高度默认值为stretch
    只设置了grid-template-rows,表示行数根据rows个数自动排布，且宽度默认值为stretch
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
/* repeat()，第一个参数是重复的次数，第二个参数是所要重复的值 */
/*
  grid-template-columns: 100px 100px 100px;
  等价于
	grid-template-columns: repeat(3, 100px);
*/
grid-template-columns: repeat(3, 100px);
```



#### auto-fill

单元格的大小是固定的，但是容器的大小不确定的时候，可以使用这个属性来让浏览器自主决定放置多少个

浏览器会尽可能的进行放置，放不下的时候，会自动换行

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
grid-template-columns: repeat(2, 100px) minmax(100px, auto);
```



#### auto

auto，表示由浏览器自己决定长度

```css
/* 此时auto的作用和fr类似 */
grid-template-columns: repeat(3, auto);

/* 此时auto表示宽度为元素内容的宽度，剩余的宽度由1fr的那个元素占据 */
grid-template-columns: auto auto 1fr;
```



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

  当column-gap的值和row-gap的值一样的时候
  可以简写成一个 -> gap: 10px;
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

![LxN2H8.png](https://s6.jpg.cm/2022/05/03/LxN2H8.png) 

![LxNCZk.png](https://s6.jpg.cm/2022/05/03/LxNCZk.png) 



### justify-items/align-items

设置单元格内容的水平和垂直的对齐方式

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

![LxNTe5.png](https://s6.jpg.cm/2022/05/03/LxNTe5.png) 



### grid-auto-columns / grid-auto-rows 

用来设置多出来的项目宽和高

```css
/* 
	设置行超出元素的高度 默认为stretch
	宽度为column的宽度
*/
grid-auto-rows: 50px;

/* 
	设置列超出元素的宽度 默认为item内容的宽度
	高度为row的高度
*/
grid-auto-columns: 80px;
```

![LxNiKt.png](https://s6.jpg.cm/2022/05/03/LxNiKt.png) 

 

## 项目属性

### grid-column-start / grid-column-end / grid-row-start / grid-row-end

grid-column-start / grid-column-end / grid-row-start / grid-row-end用来指定item的具体位置是根据哪几根网格线来进行定位的

![LxSnKT.png](https://s6.jpg.cm/2022/05/03/LxSnKT.png) 

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

![LxSYiy.png](https://s6.jpg.cm/2022/05/03/LxSYiy.png) 

```css
grid-column-start: 1;
grid-column-end: 3;
grid-row-start: 1;
grid-row-end: 3

/* 可以简写为 */
grid-area: 1 / 1 / 3 / 3;
```



### justify-self / align-self / place-self

justify-self属性设置单元格内容的水平位置（左中右），跟justify-items属性的用法完全一致， 但只作用于单个项目 (水平方向) align-self属性设置单元格内容的垂直位置（上中下），跟align-items属性的用法完全一致， 也是只作用于单个项目 (垂直方向)

```shell
justify-self: start | end | center | stretch;

align-self: start | end | center | stretch;

# place-self属性是align-self属性和justify-self属性的合并简写形式 
# 同样第一个值是设置垂直方向上的值，第二个值才是设置水平方向上的值
place-self: <align-self> <justify-self>;
```

