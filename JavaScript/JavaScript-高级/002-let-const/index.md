ES6 新增了`let`用来声明变量，`const`用来声明常量

`let`和`const`的用法和`var`一模一样



## 基本用法

### 块级作用域

`let` 和 `const`有块级作用域，而`var`没有

块级作用域 --- `{}`中的部分就是一个单独的块级作用域

```ts
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

```ts
// for循环中的{} 也可以看成是一个块级作用域
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined
```



#### 示例

var定义的变量没有块级作用域 所以只有一个全局的`i`

```js
var a = [];
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10
```



在下述例子中 每一次循环代码中的`i`都是独立的

```js
var a = [];
for (let i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 6
```



另外，`for`循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域

```js
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc
// abc
// abc
```



### 不存在变量提升

`var`命令会发生“变量提升”现象，即变量可以在声明之前使用，值为`undefined`

即脚本开始运行时，变量已经存在了，但是没有值

但按照一般的逻辑，变量应该在声明语句之后才可以使用

`let`命令改变了语法行为，它所声明的变量一定要在声明后使用，否则报错。

```js
// var 的情况
console.log(foo); // 输出undefined
var foo = 2;

// let 的情况
console.log(bar); // 报错ReferenceError
let bar = 2;
```



### 暂时性死区 

let/const 声明的变量 只在自己作用域内生效，不再受外部的影响。

```js
var tmp = 123;

// 在let命令声明变量tmp之前，都属于变量tmp的“死区”
if (true) {
  //  TDZ开始
  tmp = 'abc'; // ReferenceError
  let tmp; //  TDZ结束
}
```

如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

总之，在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。



暂时性死区的本质就是，只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的那一行代码出现，才可以获取和使用该变量。

简单来说 在实际声明变量之前 都属于该变量的TDZ 该变量存在 但不可以使用

```js
// 不报错
var x = x;

// 报错
let x = x;
// ReferenceError: x is not defined
```



在没有`let`之前，`typeof`运算符是百分之百安全的，永远不会报错

```ts
typeof undeclared_variable // "undefined"
```

typeof 一个没有声明过的值 返回值是undefined

但是在TDZ中 typeof一个没有声明的值 会直接报错

```js
typeof x; // ReferenceError
let x;
```



### 不允许重复声明

`let`不允许在相同作用域内，重复声明同一个变量。

```js
// 报错
function func() {
  let a = 10;
  var a = 1; // var 可以重复声明
}

// 报错
function func() {
  let a = 10;
  let a = 1; // let 不可以重复声明
}
```



## 块级作用域

ES5 只有全局作用域和函数作用域，没有块级作用域，这带来很多不合理的场景。

1. 内层变量可能会覆盖外层变量

   ```ts
   var tmp = new Date();
   
   function f() {
     console.log(tmp); // 变量提升
     if (false) {
       var tmp = 'hello world';
     }
   }
   
   f(); // undefined
   ```

2. 用来计数的循环变量泄露为全局变量

   ​	变量`i`只用来控制循环，但是循环结束后，它并没有消失，泄露成了全局变量。

   ```ts
   var s = 'hello';
   
   for (var i = 0; i < s.length; i++) {
     console.log(s[i]);
   }
   
   console.log(i); // 5
   ```



一对花括号中就是一个块级作用域

```js
function f1() {
  let n = 5;
  if (true) {
    let n = 10;
  }
  console.log(n); // 5
}
```

ES6 允许块级作用域的任意嵌套

每一层作用域都是独立的

```js
{{{{
  let insane = 'Hello World';
  {let insane = 'Hello World'}
}}}};
```



块级作用域的出现，实际上使得获得广泛应用的匿名立即执行函数表达式（匿名 IIFE）不再必要了。

```ts
// IIFE 写法 -- 为了避免变量冲突 单独开一个函数作用域
(function () {
  var tmp = ...;
  ...
}());

// 块级作用域写法 -- 块级作用域本身就是独立的
{
  let tmp = ...;
  ...
}
```



### 块级作用域与函数声明

ES6 规定，块级作用域之中，函数声明语句的行为类似于`let`，在块级作用域之外不可引用

```ts
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
```

```ts
// ES5 环境
function f() { console.log('I am outside!'); }

(function () {
  function f() { console.log('I am inside!'); }
  if (false) {
  }
  f();
}());
```

在ES6中 为了兼容老版本浏览器，函数形式和ES5保持一致

- 允许在块级作用域内声明函数。
- 函数声明类似于`var`，即会提升到全局作用域或函数作用域的头部。
- 同时，函数声明还会提升到所在的块级作用域的头部

所以上述代码会被转换为

```ts
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

所以推荐在块级作用域中使用函数表达式来声明函数 而不是函数声明的形式来声明函数



ES6 的块级作用域必须有大括号，如果没有大括号，JavaScript 引擎就认为不存在块级作用域。

```js
// 第一种写法，报错
if (true) let x = 1;

// 第二种写法，不报错
if (true) {
  let x = 1;
}
```



严格模式下，函数只能声明在当前作用域的顶层

```ts
// 不报错
'use strict';
if (true) {
  function f() {}
}

// 报错
'use strict';

// 没有括号 没有块级作用域
if (true)
  function f() {}
```



## const

`const`声明一个只读的常量。一旦声明，常量的值就不能改变

这意味着，`const`一旦声明变量，就必须立即初始化，不能留到以后赋值。

`const`的作用域与`let`命令相同：只在声明所在的块级作用域内有效

`const`命令声明的常量也是不提升，同样存在暂时性死区，只能在声明的位置后面使用。

`const`声明的常量，也与`let`一样不可重复声明



`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址所保存的数据不得改动

```js
const foo = {};

// 为 foo 添加一个属性，可以成功
foo.prop = 123;
foo.prop // 123

// 将 foo 指向另一个对象，就会报错
foo = {}; // TypeError: "foo" is read-only
```



如果真的想将对象冻结，应该使用`Object.freeze`方法。

在JavaScript中，冻结对象是指禁止对对象的属性进行增删改查操作

```js
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

除了将对象本身冻结，对象的属性也应该冻结。下面是一个将对象彻底冻结的函数。

```js
var constantize = (obj) => {
  Object.freeze(obj);
  Object.keys(obj).forEach( (key, i) => {
    if ( typeof obj[key] === 'object' ) {
      constantize( obj[key] );
    }
  });
};
```



## ES6 声明变量的六种方法

ES5 只有两种声明变量的方法：`var`命令和`function`命令。ES6 除了添加`let`和`const`命令，后面章节还会提到，另外两种声明变量的方法：`import`命令和`class`命令。所以，ES6 一共有 6 种声明变量的方法



## 顶层对象的属性

顶层对象，在浏览器环境指的是`window`对象，在 Node 指的是`global`对象



ES5 之中，顶层对象的属性与全局变量是等价的。

顶层对象的属性赋值与全局变量的赋值，是同一件事

```js
window.a = 1;
a // 1

a = 2;
window.a // 2
```



1. 没法在编译时就报出变量未声明的错误，只有运行时才能知道（因为全局变量可能是顶层对象的属性创造的，而属性的创造是动态的）
2. 顶层对象的属性是到处可以读写的，这非常不利于模块化编程
3. `window`对象有实体含义，指的是浏览器的窗口对象，顶层对象是一个有实体含义的对象，也是不合适的



ES6 为了改变这一点，

1. 一方面规定，为了保持兼容性，`var`命令和`function`命令声明的全局变量，依旧是顶层对象的属性
2. 另一方面规定，`let`命令、`const`命令、`class`命令声明的全局变量，不属于顶层对象的属性。



## globalThis

JavaScript 语言存在一个顶层对象，它提供全局环境（即全局作用域），所有代码都是在这个环境中运行。

1. 浏览器里面，顶层对象是`window`
2.  Web Worker 里面，`self`也指向顶层对象
3. Node 里面，顶层对象是`global`，

同一段代码为了能够在各种环境，都能取到顶层对象，现在一般是使用`this`关键字，但是有局限性。

也就是说，任何环境下，`globalThis`都是存在的，都可以从它拿到顶层对象，指向全局环境下的`this`。
