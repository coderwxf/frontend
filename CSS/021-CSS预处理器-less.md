CSS作为一种样式语言, 本身用来给HTML元素添加样式是没有问题的

但是目前前端项目已经越来越复杂, 不再是简简单单的几行CSS就可以搞定的

当CSS变的越来越多的时候，我们管理和维护CSS就变得越来越不容易了

1. css中存在大量的重复代码, 虽然可以用类来勉强管理和抽取, 但是使用起来依然不方便
2. css中无法定义变量，如果一个值被修改, 那么需要修改大量代码, 可维护性很差 --- 最新的CSS已支持定义变量
3. css中没有专门的作用域和嵌套, 需要定义大量的id/class来保证选择器的准确性, 避免样式混淆 --- 最新的CSS规则中已经存在原生嵌套写法，但只是个stage2的版本，并不是recommend的版本

于是社区为了解决CSS面临的大量问题, 出现了一系列的CSS预处理器(CSS_preprocessor)

所以CSS预处理本质上只是对原生的CSS进行了语法上的扩展，其最后依旧需要通过编译转回CSS的

因为浏览器只认识HTML/CSS/JavaScript, 浏览器是无法直接解析CSS预处理的



## 常见的CSS预处理器

1. **Sass/Scss**:
   + 2007年诞生，最早也是最成熟的CSS预处理器，拥有ruby社区的支持，是属于Haml(一种模板系统)的一部分
   + 最早的sass语法是类似于ruby的语法，使用缩进来进行区分，没有大括号和分号，所以对于前端开发者而言其学习成本很高，且sass并不完全兼容css语法
   + 所以早期sass并不流行，但是随着less的出现，sass写出现了全新的写法SCSS, SCSS的写法完全兼容原生CSS
2. **Less**:
   + 2009年出现，受SASS的影响较大，但又使用CSS的语法，让大部分开发者更容易上手
   + 比起SASS来，可编程功能不够，不过优点是使用方式简单、便捷，兼容CSS，并且已经足够使用
3. **Stylus**:
   +  2010年产生，来自Node.js社区，主要用来给Node项目进行CSS预处理支持
   + 语法偏向于Python, 使用率相对于Sass/Less少很多



## less

 Less (Leaner Style Sheets 的缩写) 是一门CSS 扩展语言, 并且兼容CSS

+ Less增加了很多相比于CSS更好用的特性
+ 比如定义变量、混入、嵌套、计算等等
+ Less最终需要被编译成CSS, 才可以运行在浏览器中



### less的使用方式

1. 使用npm 下载less，使用lessc工具对less进行编译

```shell
# lessc <less文件> <编译后需要将编译内容存放到那个css文件中>
$ npx less index.less index.css 

# 或者

$ npm install less -g
$ lessc index.less index.css
```

2. 使用npm，结合webpack，vite等自动化打包工具进行自动编译

3. 通过VSCode插件（Easy Less）来编译成CSS或者[在线编译](https://lesscss.org/less-preview/)

4. 引入CDN的less编译代码，对less进行实时的处理

```html
<!-- 修改rel属性，标识该资源引入的是less文件，需要进行编译 -->
<link rel="stylesheet/less" href="./index.less">

<!--
	引入less脚本，对less文件进行转换
	ps: 1. 使用这种方式进行less编译的时候，需要使用live server之类的本地服务进行打开
			不可以使用file协议来进行调试
			
			2. 需要在引入的less文件后使用，才可以正常进行解析
-->
<script src="https://cdn.jsdelivr.net/npm/less@4"></script>
```



### 基础语法

#### 基本语法

less是完成兼容css的。所以我们可以在Less文件中编写所有的CSS代码，只需要将扩展名换成less即可

```less
.container {
  width: 300px;
  height: 300px;
  background: gray;
}

.container .box {
  width: 100px;
  height: 100px;
  background: skyblue
}
```



#### 变量(Variables)

我们可以将会经常使用的css值，或者可能需要频繁修改的css值 定义为less变量，比如我们使用到的主题颜色值

```css
/* less中定义变量的方式为 @变量名: 变量值; */
@themeColor: #f3c258;

.box {
  width: 200px;
  height: 200px;
  background: @themeColor;
}
```



#### 嵌套(nesting)

```less
.container {
  width: 300px;
  height: 300px;
  background-color: red;

  // 在less中支持 双斜杠这种单行注释
  .box {
    width: 100px;
    height: 100px;
    background-color: skyblue;

    // &表示的是当前选择器的父级选择器
    // 在这里&就等价于.box
    &:hover {
      background-color: orange;
    }

    // 兄弟选择器的使用 + .box 等价于 & + .box
    + .box {
      background-color: green;
    }

    // 后代选择器的使用 > .line 等价于 & > .line
    > .line {
      text-decoration: line-through;
    }
  }
}
```



#### 运算

在Less中，算术运算符 +、-、*、/ 可以对任何数字、颜色或变量进行运算

```less
.container {
  width: 300px;
  height: 300px;
  background-color: red;

  .box {
    // less在进行不同单位运算的时候
    // 多个单位进行运算的时候，计算的结果以最左侧操作数的单位类型为准
    // 其余的运算符会被抛弃
    // 10% + 70px --> 10% + 70% -> 80%
    width: 10% + 70px;
    height: 100px;
    background-color: skyblue;
  }
}
```

```less
.box {
  width: 300px;
  height: 300px;
  // less中的计算 可以运用在颜色，数字和变量上
  background-color: #f00 + #0f0;
}
```

```less
.box {
  // 在css值中可以有斜杠的存在，如background-position/background-size
  // 所以除法在计算的时候，需要使用小括号进行包裹，以表示是数学运算
  
  // 在less的除法中，等式两个运算元中任意一个有单位即可
  // 如果都有单位以第一个单位也就是左边那个单位为准
  // 如果都没有单位，那么计算结果中也不存在对应单位
  width: (200px / 100px); // 200px / 100px -> 2px
  height: 300px;
  background-color: red;
}
```



#### 混入（mixin）

在原来的CSS编写过程中，多个选择器中可能会有大量相同的代码

我们希望可以将这些代码进行抽取到一个独立的地方，任何选择器都可以进行复用

在less中提供了混入(Mixins)来帮助我们完成这样的操作

`混合(Mixin)是一种将一组属性从一个规则集混入(或插入)到另一个规则集的方法`

```less
// 定义混入代码
.text_ellipsis {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.box {
  width: 300px;
  height: 300px;
  background-color: skyblue;

  // 使用混入代码 --- 使用方式和函数一致
  // 在没有参数的时候，小括号可以省略 --- 不推荐
  .text_ellipsis();
}
```



```less
// 参数定义形式 变量名: 默认值, 变量名: 默认值 。。。
.primary_border(@color: red, @borderWidth: 5px) {
  border: @borderWidth solid @color;
}

.box {
  width: 300px;
  height: 300px;
  background-color: skyblue;

  // 传递对应的参数
  // 第二个参数 使用默认值
  .primary_border(green);
}
```



```less
.box_mixin {
  width: 200px;
}

.box {
  // 如果.box_mixin在这里进行混入
  // 那么div.box的宽度为 100px
  // .box_mixin();

  width: 100px;
  height: 100px;
  background-color: red;

  // 如果.box_mixin在这里进行混入
  // 那么mixin中的值会将div.box原本设置的width值给覆盖掉
  // 所以最终div.box的宽度值为200px
  .box_mixin();
}
```



#### 映射（maps）

```less
.container {
  width: 300px;
  height: 300px;
  background-color: skyblue;
}

.box {
  // .container() => 调用获取对应的对象
  // 使用中括号语法来获取对应的属性值
  width: .container()[width];
  height: 100px;
  background-color: red;
}
```



我们可以结合映射和混入来模拟less中的函数功能

```less
@htmlRootSize: 16;

.px2rem(@px) {
  // * 1rem ---> 目的是为了将单位转换为rem
  res: calc(@px / @htmlRootSize * 1rem);
}


.container {
  width: 300px;
  height: 300px;
  background-color: skyblue;
}

.box {
  width: 100%;
  height: 100px;
  // px -> rem
  font-size: .px2rem(32)[res];
  background-color: red;
}
```



#### 继承(extend)

extend和mixins作用类似，用于复用代码

只不过继承代码最终会转化成并集选择器，而mixin会直接进行代码的插入

`mixin`

```less
.box_mixin {
  background-color: skyblue;
}

.box {
  width: 300px;
  height: 300px;
  .box_mixin();
}
```

转换后

```css
.box_mixin {
  background-color: skyblue;
}

.box {
  width: 300px;
  height: 300px;
  background-color: skyblue;
}
```



`extend`

```less
.box_mixin {
  background-color: skyblue;

  /* 和scss不同，less中被继承的选择器的子选择器的样式并不会被继承 */
  a {
    text-decoration: none;
  }
}

.box {
  width: 300px;
  height: 300px;

  &:extend(.box_mixin);
}
```

转换后

```css
.box_mixin,
.box {
  background-color: skyblue;
}
.box_mixin a {
  text-decoration: none;
}
.box {
  width: 300px;
  height: 300px;
}

```



#### 内置函数

Less 内置了多种[函数](https://lesscss.org/functions/)用于转换颜色、处理字符串、算术运算等

```less
.box {
  width: 300px;
  height: 300px;
  // color函数将颜色关键字 转换为对应的 HexColor
  background-color: color(skyblue);
  // covert函数可以进行单位的转换
  // 但是其只能在绝对单位之间进行转换 如in，px，cm，mm等
  font-size: convert(16px, 'in');
  // 向上取整
  line-height: ceil(12.5px);
  // 向下取整
  border: floor(1.5px) solid red;
}
```



#### 注释 (comments)

```less
// 在less中可以 编写单行注释
/* 在less中可以 编写块级注释 或者叫多行注释 */
```



#### 作用域(scope)

```less
@mainColor: #67c23a;

.box {
  .inner {
    @mainColor: #e6a23c;

    // 在less中，一个大括号就是一个作用域
    // less在查找变量的时候
    // 首先在当前作用域或当前作用域中引入的混入中进行查找
    // 如果不存在，就去上一层作用域和上一层作用域中的混入中进行查找
    // 一层层找，如果在顶层中依旧没有找到，那么就报错
    span {
      color: @mainColor;
    }
  }
}
```



#### 导入 (importing)

```less
// 导入一个 .less 文件，此文件中的所有变量和混入就可以全部使用了
// 和原生import必须放在第一行不同的是，less中的导入只需要在使用前导入即可，并不需要强制放置在第一行
// @import url(./mixin.less);

// 在引入的时候，less文件的后缀名可以省略
@import url(./mixin.less);

.box {
  width: 300px;
  height: 30px;
  background-color: skyblue;
  // 在其它文件中定义的混入函数
  // 需要先导入后，才可以进行使用
  .text_ellipsis();
}
```
