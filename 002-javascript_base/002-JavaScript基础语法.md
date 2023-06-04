## 编写位置

### HTML元素上

```html
<!--
  在onclick中编写JS代码
  1. 自定义的JS代码会优先于元素的默认行为(在这里就是跳转去google)
  2. JS的onxxxx事件定义事件的时候，其名称都是小写的，不是小驼峰
  3. alert中的参数使用单引号，是因为onclick使用了双引号，单引号和双引号不可以混合使用
-->
<a href="https://www.google.com" onclick="alert('Hello World')">click me</a>
```

```html
<!--
  在a元素的href属性中，也可以编写对应的JS代码
  但是此时JS代码之前需要加上javascript: 前缀
	结尾分号可以省略
-->
<a href="javascript: alert('Hello World')">click me</a>
```

将JS代码编写在HTML元素上是不被推荐的

在HTML元素上编写对应的JS代码，其只能编写简单的代码，如果代码过多不仅不方便进行书写，而且可读性差，不好维护



### script标签中

```html
<a href="#" id="foo">click me</a>

<!-- 
	script元素内部用了编写js代码
	页面中可以存在多个script标签元素
-->
<script>
  // 当浏览器解析到script标签后，浏览器会自动加载和解析script标签中的JS代码
  const aEl = document.getElementById('foo')
  aEl.onclick = () => alert('Hello World')
</script>
```



### 外部的script文件

```html
<!-- 将js文件编写到外部的js文件中，随后通过script标签的src属性来进行引入 -->
<script src="./index.js"></script>
```



## noscript元素

在某些情况下，script脚本可能无法被正常解析

+ 早期的不支持JS的浏览器
+ 浏览器的JS功能被关闭了

所以我们解决这种情况，浏览器需要一个页面优雅降级的处理方案，也就是`<noscript>`

```html
<!-- 当支持js的时候，会执行script元素中的内容，不支持noscript中的内容 -->
<script>
  alert('Hello World')
</script>

<!-- 浏览器不支持JS代码的时候，会自动执行noscript元素中的内容  -->
<noscript>
  <p>浏览器不支持JS代码或已关闭了JS功能</p>
</noscript>
```



## JS编写的注意事项

1.   script元素不推荐写成单标签

在外联式引用js文件时，script标签中不可以写JavaScript代码，并且script标签不推荐写成单标签

```html
<!--
  1. 以单标签形式去使用script标签是不被推荐的，且必须使用/来闭合标签 -- 兼容性不好
  2. 推荐使用双标签的形式来运行js代码
-->
<script src="./index.js" />

<!-- 推荐写法 -->
<script src="./index.js"></script>

<!--
  当script标签上存在src属性来加载外部的js文件的时候
  script标签内部编写的js代码会被忽略
-->
<script src="./index.js">
  // 这里的代码会被忽略
  alert('Hello JavaScript')
</script>
```



2. text/javascript

在早期的代码中，`<script>`标签中会使用` type=“text/javascript”`

但是这个属性现在默认可以省略，因为所有现代浏览器以及 HTML5 中的默认脚本语言都是JavaScript

所以现在，`script标签`的`type属性`的默认值就是`text/javascript`



3. JavaScript代码严格区分大小写

HTML元素和CSS属性不区分大小写，但是在JavaScript中严格区分大小写



4. JS的解析顺序

`script`标签本质上就是一个普通的HTML元素，所以JavaScript默认遵循HTML文档的加载顺序，即自上而下的加载顺序

而JS中通常会涉及到操作DOM和BOM的部分，如果JS脚本过早的执行，可能导致JS无法获取到对应的元素

所以推荐将JavaScript代码的编写位置放在body子元素的最后一行

虽然script脚本放置在body元素后或者html元素后，浏览器也可以正常加载和执行对应的JS代码

但其本质是浏览器对其进行了容错处理，script标签作为普通HTML标签，是页面的一部分

所以最为推荐的JS编写位置是放在body子元素的最后一行



## JS和浏览器的交互方式

| 交互方式       | 功能                             | 参数个数 |
| -------------- | -------------------------------- | -------- |
| alert          | 弹窗查看 --- 函数                | 一个     |
| console.log    | 在浏览器控制台查看 --- 方法      | 多个     |
| prompt         | 在浏览器接受用户输入 --- 函数    | 一个     |
| document.write | 将参数内容追加到页面中  --- 方法 | 多个     |

```js
alert('Hello World')

console.log('Hello World')

// console.log可以传入多个参数
// 参数和参数之间在展示的时候，会使用空格进行分割
console.log('Hello', 'World') // => Hello World

/*
  参数为输入弹框的提示信息
  prompt函数会将用户的输入结果以字符串的形式作为该函数的返回值进行返回
  如果用户没有任何的输入，直接点击了取消，那么该函数的返回值为null
*/
const res = prompt('请输入内容')

// 插入新内容，且插入的新内容后无换行
document.write('Hello World')

// 插入新内容，且插入的新内容后存在换行
// 但是这个换行在HTML解析的时候，会被转换为空格
// 如果需要显示这个空格，需要将输出的内容包裹在pre标签中
document.writeln('Hello World')

// document.write可以传入多个参数
// 参数和参数之间并不会使用空格进行分割
// 而是会紧挨在一起
document.write('Hello', 'World') // => HelloWorld
```

 

## 语句和分号

语句(Statements)是向浏览器发出的指令，通常表达的是一个操作或者行为(Action)

例如`console.log('Hello World')`就是一条语句



通常每条语句的后面我们会添加一个分号(semicolon)，表示语句的结束

但当存在换行符(line break)时，在大多数情况下可以省略分号

 JavaScript 将换行符理解成“隐式”的分号，这也被称之为自动插入分号(an automatic semicolon)

所以我们在编写JS的时候，可以省略语句结束的那个分号



但是如果一条语句以括号、方括号、花括号作为语句开头的时候或者多行JS语句在一行编写的时候(例如代码压缩后)

前一条语句的结尾必须加上分号，否则该语句有可能会和之前的语句作为整体一起进行解析，从而解析失败



```js
// 此时第一条打印语句结尾的分号不可以省略
// 否则第一条打印语句会和IIFE作为一条语句进行解析
// 从而报错
console.log('Hello World')

(function(){
  console.log('Hello JavaScript')
})()
```

```js
console.log('Hello World')

// 除了通过在第一条语句的结尾加上分号，以将console.log和IIFE进行划分
// 我们也可以在IIFE前，加上分号或者叹号来进行两条语句之间的划分
;(function(){
  console.log('Hello JavaScript')
})()
```



## 注释

```js
// 在JS中，注释不支持嵌套

// 单行注释

/*
	多行注释或者叫块级注释
*/

// 文档注释 一般用于对类和函数进行注释的说明
// 常用于工具类或工具函数中
// 在vscode的html格式文件中，对文档注释的支持度并不高
// 所以推荐在vscode的单独的js文件中，为对应的类或函数添加文档注释

/**
 * 一个简单的加法运算函数 --- 用于演示文档注释的使用
 * @param {number} num1 参数加法运算的第一个参数
 * @param {number} num2 参加加法运算的第二个参数
 * @returns 两数之和
 */
function sum(num1, num2) {
  return num1 + num2
}
```