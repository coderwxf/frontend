## 绑定规则

Javascript函数在被调用的时候，会被默认绑定一个名为this的对象

在Javascript中，函数的this关键字指向的对象称为函数的执行上下文（一般为函数的调用者）

函数的执行上下文是在函数被调用时动态创建的，也就是说this的值是在运行时被指定的，和函数的编写位置没有任何关系

在Javascript中， this的值一般可以分为以下四种情况:

1. 默认绑定
2. 隐式绑定
3. 显示绑定
4. new绑定



### 默认绑定

默认绑定又被称之为独立函数调用 即函数没有被绑定到某个对象上进行调用

```js
// 在非严格模式中 this指向 globalThis
function foo() {
  console.log(this)
}

foo()
```

```ts
// 在严格模式中 this指向 undefined
"use strict"

function foo() {
  console.log(this)
}

foo()
```



### 隐式绑定

隐式绑定就意味着该函数的调用发起方式是通过某个对象的调用而发起的

也就是说对应的this是通过JavaScript解析引擎来进行绑定的

```js
const obj = {
  foo() {
    console.log(this)
  }
}

obj.foo()
```



### New 绑定

在JavaScript中，使用function关键字定义的函数，可以作为普通函数进行调用，也可以使用new关键字将其作为构造函数进行调用

使用new关键字来调用函数是，会执行如下的操作:

1. 创建一个全新的对象
2. 构造函数的显示原型会指向新对象的隐式原型
3. 这个新对象会绑定到函数调用的this上
4. 将this所指向的那个对象返回 
   + 如果函数显示的返回了一个对象
   + 那么就会将显示返回的那个对象作为函数的返回值返回
   + 而不是使用this所指向的那个对象

```js
function Person() {
  console.log(this) // Person{}
}

const per = new Person()
```



### 显示绑定

所谓显示绑定就是我们明确告诉JavaScript Engine 调用函数的时候 this所指向的值

常见的用于显示this绑定的方法有call，apply和bind



#### call / apply

call 和 apply 都是 在函数被调用的时候 显示指定this对应的值

call 和 apply 的第一个参数都是this的值

+ 如果是对象，那么this的值就是该对象
+ 如果是原始数据类型，那么this的值就是原始数据类型对应的包装类
+ 如果是null和undefined这类没有包装类的原始数据类型，this的值就是globalthis或undefined

call 和 apply 的 后续参数是 参数列表

+ call 通过 剩余参数的形式 传递参数列表
+ apply 通过 数组的形式 传达参数列表

```js
function foo(name, age) {
  console.log(this, name, age)
}

foo.call({ name: 'thisArg' }, 'Klaus', 23)
foo.apply({ name: 'thisArg' }, ['Klaus', 23])
```



#### bind

对于call/apply方法在每次绑定完this的时候，就会立即对函数进行调用，有的时候，我们可能希望修改为this指向后，不立即对函数进行调用，此时就可以使用bind方法

使用bind方法，bind() 方法创建一个新的`绑定函数(bound function，BF)`

调用绑定函数的时候，其调用形式明明是默认函数调用，但是其内部的this确实我们显示绑定的那个对象

并且调用函数时候所需要的实参既可以在bind方法定义的时候传入也可以在绑定函数调用的时候被传入

这些行为是十分怪异的，所以`bind方法返回的绑定函数，也被称之为怪异函数对象`

```js
function fun(name, age) {
  console.log(this, name, age)
}

const foo = fun.bind({ name: 'thisArg' }, 'Klaus')

// 实际传入的参数为 [...bind传入的除this外的所有参数, ...调用foo传入的参数] 
foo(23) // => { name: 'thisArg' } Klaus 23
```



## 回调函数

在JavaScript中，在使用一些内置函数或者第三方库的时候，参数往往也是一个函数，这类函数被称之为回调函数,  然而回调函数内部的this，往往取决于回调函数是怎么被调用的

常见的内置函数中的this如下:

1. 定时器函数中的this 无论在严格模式还是非严格模式下 指向的都是globalThis

2. 对于DOM事件，内部callback中的this指向的都是 唤起事件的那个dom对象

3. 对于大部分JavaScript内置高阶函数，如果指定了thisArg，那么this指向的就是thisArg

   如果没有指定thisArg，那么此时内部callback中的this指向

   ​	在非严格模式下 就是globalThis

   ​    在严格模式下 就是undefined

```js
const arr = ['Klaus', 'Alex']

arr.forEach(function() {
  console.log(this) // => { name: 'Klaus' }
}, { name: 'Klaus' })
```



## 优先级

`new > bind > apply/call > 隐式绑定 > 默认绑定`

1. `new绑定和call、apply是不允许同时使用`，所以不存在谁的优先级更高

2. `call 和 apply 不可能同时使用`，所以也不存在谁的优先级更高 



## 箭头函数

箭头函数是ES6之后增加的一种编写函数的方法，并且它比函数表达式要更加简洁, 尤其是在作为回调函数被传递的时候

在ES6之前，编写函数主要的方式是`函数表达式`和`函数声明`， 箭头函数可以看成是函数表达式的特殊写法

```js
// 参数列表 => 函数体
const foo = () => {}
```



箭头函数有以下特点

1. 箭头函数没有this 和 arguments属性
2. 箭头函数内部没有原型对象，所以箭头函数不能通过new关键字进行创建，只能作为普通函数进行调用



箭头函数可以进行如下简写:

1. 如果只有一个参数时, `()`可以省略
2. 如果函数执行体中只有一行代码, 那么可以省略大括号
   + 并且这行代码的返回值会作为整个函数的返回值
   + 所以此时不需要return关键字
3. 如果函数执行体只有返回一个对象, 那么需要给这个对象加上`()`， 以区分普通代码块



## 特殊规则

### 箭头函数

箭头函数不符合this的四种标准规则(也就是不绑定this)，而是根据外层作用域来决定this



### 严格模式

在非严格模式下，使用apply或call调用函数的时候，给this赋值的时候，会尽可能的将需要赋予的值转换为对象类型，如果无法转换为对象类型，就会直接使用默认绑定规则

```js
function fun() {
  console.log(this)
}

fun.call({ name: 'klaus' }) // => { name: 'klaus' }
fun.call(123) // => [Number: 123]
fun.call(null) // => GlobalThis
fun.call(undefined) // => GlobalThis
```



在严格模式下，使用apply或call调用函数的时候，会直接对this进行赋值操作，无论这个值是不是对象类型

```js
"use strict"

function fun() {
  console.log(this)
}

fun.call({ name: 'klaus' }) // => { name: 'klaus' }
fun.call(123) // => 123
fun.call(null) // => null
fun.call(undefined) // => undefined
```



### 间接函数引用

创建一个函数的 间接引用，这种情况使用默认绑定规则

```js
const obj = {
  fun() {
    console.log(this)
  }
}

const obj2 = {};

/*
  1. 当一行代码以 小括号，中括号 或者大括号进行调用的时候
     + 在上一行代码结束的时候，以分号结尾，以表示一行的结束 = 推荐
     + 在下一行开头的时候，使用 叹号，加号或减号 以表示该行是一个整体 -- 不推荐
  2. obj2.fun = obj.fun的返回值是 fun函数本身
     所以(obj2.fun = obj.fun)() <=> fun() 
     所以内部的this值是undefined或globalThis
*/
(obj2.fun = obj.fun)()
```



### 综合练习

```js
var name = 'window'

var person1 = {
  name: 'person1',
  foo1: function () {
    console.log(this.name)
  },
  foo2: () => console.log(this.name),
  foo3: function () {
    return function () {
      console.log(this.name)
    }
  },
  foo4: function () {
    return () => {
      console.log(this.name)
    }
  }
}

var person2 = { name: 'person2' }


// 开始题目:
person1.foo1(); // 隐式绑定: person1
person1.foo1.call(person2); // 显式绑定: person2

person1.foo2(); // 上层作用域: window
person1.foo2.call(person2); // 上层作用域: window

person1.foo3()(); // 默认绑定: window
person1.foo3.call(person2)(); // 默认绑定: window
person1.foo3().call(person2); // 显式绑定: person2

// 箭头函数内部没有this，其会沿着作用域链, 一点点的向上层执行上下文(上层的调用者)进行查找
person1.foo4()(); // person1
person1.foo4.call(person2)(); // person2
person1.foo4().call(person2); // person1
```



```js
var name = 'window'

function Person(name) {
  this.name = name
  this.obj = {
    name: 'obj',
    foo1: function () {
      return function () {
        console.log(this.name)
      }
    },
    foo2: function () {
      return () => {
        console.log(this.name)
      }
    }
  }
}

var person1 = new Person('person1')
var person2 = new Person('person2')

person1.obj.foo1()() // 默认绑定: window
person1.obj.foo1.call(person2)() // 默认绑定: window
person1.obj.foo1().call(person2) // 显式绑定: person2

person1.obj.foo2()() // 上层作用域查找: obj(隐式绑定)
person1.obj.foo2.call(person2)() // 上层作用域查找: person2(显式绑定)
// 是通过person1.obj进行调用的
person1.obj.foo2().call(person2) // 上层作用域查找: obj(隐式绑定)
```



