## globalThis

JavaScript引擎在执行代码之前，会在堆内存中创建一个全局对象（Global Object，GO）



这个全局对象具有以下特点：

- 全局访问: 所有的作用域（scope）都可以访问这个全局对象。

- 全局属性和方法: 该对象中存在全局的属性和方法，如 `Date`、`Array`、`String`、`Number`、`setTimeout`、`setInterval` 等。

- 存在指向自身的自引用，理论上可以无限递归下去

  

不同宿主环境中的全局对象式不同的

1. 浏览器是window
2. node是global
3. worker是self

为了方便代码的跨平台，提供了globalThis，其会根据不同平台，返回对应的全局对象



## 执行上下文

执行代码是以栈的方式运行的，这个栈叫执行上下文栈(Execution Context Stack，简称ECS)

任意一段代码在被执行之前都会创建对应的执行上下文(Execution Contexts)后再入栈，执行完再出栈

最顶层的那个执行上下文栈被称之为活跃执行上下文



全局代码执行会创建Global Execution Context(GEC) 被压入ECS中，GEC是最早入栈的执行上下文



对于任意一个EC，其内部都会初始化三个属性

1. 作用域链(scope chain)
2. VO (Variable Object)
3. this



全局VO就是GO

默认情况下，this指向代码对应的执行上下文，对于全局执行上下文对应的就是globalThis



在执行代码之前，会创建对应执行上下文，在此过程中，会将定义的变量、函数等加入到VO中

+ 函数被解析并被挂载到VO 「 有值 」
+ 解析变量挂载到VO 「 值为undefined，如果变量和函数同名，变量解析默认失效 」
+ 这个过程被称之为变量的作用域提升(hoisting) 或被称之为预解析

```js
foo()

var foo = 'foo'

function foo() {
  console.log('function foo')
}

console.log(foo) // => foo
```



创建函数时，会创建对应函数执行上下文Functional Execution Context，简称FEC)

并创建AO(Activation Object， 活跃对象)作为执行上下文的VO

AO上会挂载函数的参数，arguments，函数内部定义的变量和函数等



执行上下文上存在一个私有属性`[[scope]]`，其是一个伪数组，用于表示变量查找的作用域链(Scope Chain)

和this不同，作用域是在函数被定义的那一刻就已经被确定下来，保存在函数内部一个私有属性`[[scopes]]`中，也就是说JS的作用域是词法作用域，而不是动态作用域



在ES6中，对对应的术语进行了调整，但是整体思路是一致的，

其中比较大的变化是ES5中执行上下文是由VO, this和 [[ scope ]]组成的

但是在ES6开始，执行上下文由词法环境和this组成



词法环境是ES6中运行代码时会生成的一个数据结构，主要由以下两部分组成

+ 一个词法环境是由环境记录（Environment Record）和一个可能为空的外部词法环境（o*ute;r* Lexical Environment）组成 「 全局环境记录中，外部词法环境的值就是null 」
+ 词法环境就是ES5中的VO

也就是说ES5中`执行上下文 = VO + this + [[ scope ]]`

ES6中的 执行上下文 = `词法环境 + this`



为了兼容老版本的语言，环境记录也被划分为了两类

| 类型           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| 声明式环境记录 | 存储 `let`、`const` 声明的变量<br />存在暂时性死区（TDZ），即在声明之前访问这些变量会导致引用错误<br />这些变量在作用域内是块级作用域。 |
| 对象式环境记录 | 存储 `var` 声明的变量和 `function` 声明的函数 <br />这些变量在整个函数或全局作用域内可用，不存在块级作用域<br />函数声明会被提升到作用域的顶部，这意味着函数可以在声明之前调用 |

> 为了保持一致性和向后兼容性，`function` 声明仍然存储在对象式环境记录中。