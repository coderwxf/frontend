[Sass (Syntactically Awesome StyleShefrts)](https://sass-lang.com/) 是于2007年发布的基于Ruby语言编写的一款CSS预处理语言

最初它是 为了配合HAML (一种缩进式HTML预编译器)而设计的，因此有着和HTML一样的缩进式风格，后缀名为sass

因此默认情况下sass是不兼容css的

但是sass在后期受到less的影响，改变了其默认的语法风格，并将后缀名修改为scss

scss文件默认是完全兼容css文件的，也就是说只要直接将css文件的后缀直接修改为scss后，其就是一个合法的scss文件

现如今，`一般说的sass指代的就是scss风格的sass`



`sass风格`

```scss
nav
  ul
    margin: 0
    padding: 0
    list-style: none

  li
    display: inline-block

  a
    display: block
    padding: 6px 12px
    text-decoration: none
```



`scss风格`

```css
nav {
  ul {
    margin: 0;
    padding: 0;
    list-style: none;
  }

  li { display: inline-block; }

  a {
    display: block;
    padding: 6px 12px;
    text-decoration: none;
  }
}
```



## 安装

sass虽然可以完全兼容css语法，但是其本质依旧是一个css预处理器

浏览器并不可以直接解析识别sass语法，所以其需要经过编译工具的编译和转换，将sass文件为css文件后，才可以被浏览器解析和使用

常见的sass的编辑器有很多，比较常见的有`node-sass`和 `sass`



### node-sass

`安装`

```shell
$ npm i node-sass
```



`使用`

```shell
# 在编译的时候，如果文件夹或文件不存在，会主动去创建对应的文件或文件夹

# 编译 - 文件
$ node-sass <scss文件路径> <css输出文件路径>

# 编译 - 文件夹
$ node-sass <scss文件夹路径> -o <css输出文件夹路径>

# 编译 - 实时监听 - 文件
$ node-sass -w <scss文件路径> <css输出文件路径>

# 编译 - 实时监听 - 文件夹
$ node-sass -w <scss文件夹路径> -o <css输出文件夹路径>

# 可以使用outputStyle来指定输出模式
$ node-sass <scss文件路径> <css输出文件路径> --outputStyle=<输出模式>
# 上述命令也等价于
$ node-sass <scss文件路径> <css输出文件路径> --outputStyle <输出模式>
```



### sass

`安装`

```shell
$ npm i sass
```



`使用`

```shell
# 在编译的时候，如果文件夹或文件不存在，会主动去创建对应的文件或文件夹

# 编译 - 文件
$ sass <scss文件路径> <css输出文件路径>
# 或
$ sass <scss文件路径>:<css输出文件路径> # 冒号的两边不要有空格

# 编译 - 文件夹
$ sass <scss文件夹路径>:<css输出文件夹路径>

# 编译 - 实时监听 - 文件
$ sass -w <scss文件路径>:<css输出文件路径>

# 编译 - 实时监听 - 文件夹
$ sass -w <scss文件夹路径>:<css输出文件夹路径>

# 可以通过style来指定输出模式
$ sass <scss文件路径> <css输出文件路径> --style=<输出模式>
# 上述命令也等价于
$ sass <scss文件路径> <css输出文件路径> --style <输出模式>
```



### sass vs node-sass

| 类别      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| sass      | 前身为dart-sass，目前是sass官方推荐的编译工具<br />底层使用dart进行编写，性能相比node-sass而言，更为的优秀 |
| node-sass | 底层使用node来进行实现<br />是早期的sass编译工具             |

> 所以在开发中，推荐使用`sass`来替换`node-sass`



## 输出模式

原始的scss文件内容如下

```scss
.container {
  width: 300px;
  height: 300px;

  .box {
    width: 100px;
    height: 100px;
  }
}
```



1. 嵌套模式 --- nested --- 默认输出方式

```js
/* 输出示例 */
.container {
  width: 300px;
  height: 300px; }
  .container .box {
    width: 100px;
    height: 100px; }
```



2. 展开模式 --- expanded --- 测试环境推荐使用

```css
 .container {
  width: 300px;
  height: 300px;
}

.container .box {
  width: 100px;
  height: 200px;
}
```



3. 紧凑模式 --- compact

```css
.container { width: 300px; height: 300px; }

.container .box { width: 100px; height: 200px; }
```



4. 压缩模式 --- compressed --- 生产环境推荐使用

```css
.container{width:300px;height:300px}.container .box{width:100px;height:200px}
```



## 基本语法

### 注释

```scss
// scss中的单行注释 --- 不会出现在编译后的css中

/*
	scss中的多行注释 --- 会存在编译后的未压缩的css中
*/

/*!  
	scss中的强制注释
		--- 会存在于编译后的css中，无论是压缩的还是未压缩的
		--- 但是在压缩格式的css中添加注释多数情况下是没有意义的，所以这种注释很少使用						
*/
```



### 变量

```scss
// scss中使用$定义变量
// scss变量的命名规则是任意的
// 推荐:
// 1. 如果该变量是用于类或css属性，推荐其命名规则参考css对应命名规则
// 2. 如果该变量是用于scss扩展功能(如mixin,include等)或简单的逻辑运算，那么其命名规则推荐参考js对应命名规则
$color: red;
$fz: font-size;

.box {
  width: 200px;
  height: 200px;
  // 使用变量
  // scss和less一样，一个选择器的大括号就是一个作用域
  // scss在查找变量的时候，会在当前作用域中查找
  // 在去其上层作用域中查找，依次类推，如果全局也没有找到就报错
  background-color: $color;
  // 如果变量是属性key，变量需要使用#{}进行包裹
  // 类似于js中的计算属性需要使用中括号包裹是一样的
  #{$fz}: 40px;
  color: #fff;
}
```

```scss
$border: 1px solid red;
$w: 300px;

.box {
  width: $w;
  height: $w;
  border: $border;
}
```



### 嵌套

```scss
.dialog {
  // 在嵌套中，&表示的是引用父级选择器
  // 在这里& === .dialog
  &-container {
    .box {
      width: 200px;
      height: 200px;
      // 属性的嵌套
      border: {
        left: {
          color: skyblue; // -> border-left-color
          style: solid; // -> border-left-style
          width: 5px; // -> border-left-width
        }
      }

      // &结合伪类一起使用
      &:hover {
        // 属性的嵌套
        background: {
          color: red; // -> background-color
        }
      }
    }

    // &-title === .dialog-container-title
    &-title {
      font-size: 20px;
    }
  }
}
```



### mixin

```scss
// 定义mixin
// 和less不一样，scss中mixin的名称不可以以点开头
@mixin text-overflow {
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}

.box {
  // 和less一样，再使用mixin的时候，如果mixin没有参数，小括号可以省略，但是不推荐
  // mixin和include在解析的时候, 和css的解析规则是一致的
  // 所以如果include之后还有内容的时候，分号是不可以省略的
  @include text-overflow();
  // div.box自己的样式
  color: #f0f;
}
```

```scss
// $n: 2
// $n -> 变量
// 2 -> $n的默认值
@mixin line-clamp($n: 2) {
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box;
  -webkit-line-clamp: $n;
  -webkit-box-orient: vertical;
}

.box {
  // 使用mixin 并传参
  @include line-clamp(3);
}
```

```scss
@mixin box($w: 200px, $color: skyblue) {
  width: $w;
  height: $w;
  background-color: $color;
}

.box {
  // 在传参的时候，顺序不一定需要和mixin中定义的顺序一致
  // 也可以自主决定
  @include box($color: red, $w: 300px)
}
```



### 继承

```scss
.container {
  width: 300px;
  height: 300px;

  .box {
    width: 200px;
    height: 200px;
    background-color: skyblue;
  }
}

.foo {
  // .foo的样式继承自.container
  @extend .container;
  
  // .foo自己的样式
  color: orange
}
```

`编译后`

```css
/* 继承的实现本质是选择器的分组  */
.container, .foo {
  width: 300px;
  height: 300px;
}

/* 不单单父选择器会被继承，如果父选择器中有子选择器，那么子选择器也会被继承  */
.container .box, .foo .box {
  width: 200px;
  height: 200px;
  background-color: skyblue;
}

.foo {
  color: orange;
}
```



### import

```scss
// 引入另一个scss文件 --- scss后缀默认可以省略
@import './variable'
```

```scss
body {
  margin: 0;
  padding: 0;
}

// 和原生的import不同，scss中的 import 不需要在第一行进行导入
// 但是如果在当前文件中需要使用引入文件中的内容的时候
// 那么引入文件必须位于使用前，也就是必须先引入后使用
@import './mixin.scss';

.container {
  // mixin来自于mixin.scss
  // 所以需要在import后才可以使用对应的mixin
  @include textOverflow();
}
```



### 运算

scss中支持四则运算，且支持包括`min, max, percentage, floor, ceil, round, abs`等在内的一系列数学函数

```scss
.box {
  width: 10px + 20px; // -> 30px
  height: 40 - 10px; // -> 30px

  // scss中不支持不同单位之间的运算
  // 所以以下运算在解析的时候，会报错
  // width: 20em + 10px;
  // height: 20% + 30px;
}
```

```scss
// scss中乘法运算的一些注意事项
.box {
  // 20px * 10px -> 200pxpx => pxpx不存在，解析报错
  // width: 20px * 10px;

  width: 20px * 10; // 20px * 10 -> 200px
}
```

```scss
// scss中除法的一些注意事项
.box {
  // 300px / 100px -> 300px / 100px
  // 因为在css中，某些值中存在 /
  // 例如 background-position/background-size, font-size/line-height等
  // 所以如果 被除数或除数有单位 scss就不会将/识别为除法
  // width: 300px / 100px;

  // 使用小括号将除法包裹起来，就相当于显示告诉scss编译器，这里是一个除法运算
  width: (300px / 100); // 3px
}
```

```scss
// scss中除法的一些注意事项
.box {
  // 300 / 100px -> 3 * (1/px) --- 1/px 整体会被识别为单位 因此会报错
  // width: (300 / 100px);

  // 300px / 100px -> 3 --- 被除数上的px和除数上的px被消除了
  width: (300px / 100px);
}
```



scss中的字符串和字符串的加法会被识别为字符串拼接, 且拼接后的字符串会使用双引号进行包裹

```scss
.box {
  // percentage 将表达式的结果转换为百分比形式的值
  // 所以percentage函数参数的运算结果必须是一个纯数字类型的值
  width: percentage(3 / 100); // -> 3%
  height: 300px;
  background-color: skyblue;
  font: 20px / 300 'sans-' + 'sarif'; // -> font: 20px/300 "sans-sarif";
}
```



```scss
// scss也可以对颜色值进行运算
// css中的颜色是分为R,G,B三个色道的
// 所以scss中的颜色运算是针对三个色道单独进行计算的
// 因此如果计算结果值大于等于255，就取255，不会产生颜色进位的情况
.box {
  background-color: #f00 + #0f0; // -> #ff0
  color: #400 * 2; // -> #800
}
```



### 插值语法

+ scss中的插值使用 `#{}`

+ scss中的插值可以使用在注释，选择器，属性名, 属性值中

```scss
$color: red;
$c: color;
$variable: '插值';

/* 在注释中使用#{$variable} */

// 在属性名,属性值和选择器中使用插值
.#{$color} {
  background-#{$c}: #{$color};
}
```

`编译后`

```css
@charset "UTF-8";
/* 在注释中使用插值 */
.red {
  background-color: red;
}
```