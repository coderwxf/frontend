函数在调用时，JavaScript会默认给this绑定一个`对象值, 也就是this`

this的绑定和定义的位置(编写的位置)没有关系， 而是和调用方式以及调用的位置有关系

所以this是在运行时被绑定的



## 基本绑定规则

### 默认绑定

`默认绑定又被称之为独立函数调用`

对于默认绑定，函数内部的this存在以下情况:

+ 在严格模式中 this指向 globalThis
+ 在非严格模式  this指向 undefined

```js
const bar = {
  username: 'Klaus',
  baz() {
    console.log(this) // => globalThis
    console.log(this.username) // => undefined
  }
}

// let/const 定义的变量存放在VE中，不会存放于GO中
const username = 'Alex'

const foo = fn => fn()

foo(bar.baz)
```



### 隐式绑定

隐式绑定就意味着该函数的调用发起方式是通过某个对象的调用而发起的

也就意味着在调用的对象内部有一个对函数的引用(常见的类型为属性)



### new绑定

JavaScript中的任意一个函数都可以当做一个类的构造函数来使用，也就是使用new关键字来进行调用

使用new关键字来调用函数是，会执行如下的操作:

1. 创建一个全新的对象
2. 这个新对象会被执行prototype连接
3. 这个新对象会绑定到函数调用的this上
4. 如果函数没有返回对象，表达式会返回这个新对象

也就是说当一个函数使用new来进行调用的时候，其内部的this指向的就是那个生成的实例对象

```js
function sum(num1, num2) {
  console.log(this)
}

// 使用new方式调用函数的时候
// 内部的this指向的是当前函数所创建出的那个实例对象
// 实例对象的类型为函数名
new sum(10, 20) // => sum {}
```



### 显示绑定

所谓显示绑定就是手动主动对函数中的this进行绑定，常见的用于显示this绑定的方法有call，apply和bind

```js
function foo() {
  console.log(this)
}

// this需要是一个对象类型的值
// 所以如果在进行this的显示绑定的时候，如果传入的值是基本数据类型

// 在非严格模式下 call/apply/bind会自动尝试将传入的值转换为对应的包装类类型值
// 在严格模式下 call/apply/bind会使用传入的值作为this对应的值，不进行任何形式的转换
foo.call(124) // => [Number: 124]
foo.call('Klaus') // => [String: 'Klaus']

// 而对于undefined和null 因为他们没有对应的包装类类型
// 所以call/apply/bind在进行绑定的时候，无法找到对应的包装类类型
// 因此就会使用默认绑定规则对其进行this的显示绑定

// 在非严格模式下 传入null/undefined 函数内部的this的值 皆为globalThis
// 在严格模式下 传入null，函数内部的this值为 null。 传入undefined的时候，函数内部this值为undefined
foo.call(undefined)
foo.call(null)
```



#### call/apply

```shell
# 对于call方法
# 参数1 - 需要显示绑定的那个this对象
# 后续参数 - 函数在调用的时候需要传入的其它参数
#         - 以参数列表的形式依次进行传入
function call(thisArg, arg1, arg2, ...)

# 对于apply方法
# 参数1 - 需要显示绑定的那个this对象
# 后续参数 - 函数在调用的时候需要传入的其它参数
#         - 以参数数组的形式进行传入 
function apply(thisArg. [argsArray])
```



#### bind

对于call/apply方法在每次绑定完this的时候，就会立即对函数进行调用，

有的时候，我们可能希望修改为this指向后，不立即对函数进行调用

同时在每次调用的时候, 我们都需要每次都显示的对函数内部的this进行显示绑定，这就会导致存在重复的代码

这个时候，我们就可以使用bind方法

```shell
# bind方法的第一个参数用于显示绑定this
# 后续可以传入调用函数所需要的参数 --- 可以是全部的参数，也可以是部分的参数
function bind(thisArgs[, arg1[, arg2[...]])
```



使用bind方法，bind() 方法创建一个新的`绑定函数(bound function，BF)`

该绑定函数是一个 `exotic function object(怪异函数对象)`

这是因为调用绑定函数的时候，其调用形式明明是默认函数调用，但是其内部的this确实我们显示绑定的那个对象

并且调用函数时候所需要的实参既可以在bind方法定义的时候传入也可以在绑定函数调用的时候被传入

这些行为是十分怪异的，所以bind方法返回的绑定函数，也被称之为怪异函数对象

```js
function sum(num1, num2, num3, num4) {
  console.log(this)
  console.log(num1, num2, num3,  num4)
}

const user = {
  name: 'Klaus'
}

// 函数的实参可以在bind函数绑定this的时进行传入
// 也可以在绑定函数被调用的时候进行传入
// 在本例中bind函数传入了部分实参，绑定函数调用的时候也传入的部分实参
// 在实际被调用的函数中，其会以[...bind绑定时传入的实参，...绑定函数调用时传入的实参]的方式
// 依次将对应的函数实参进行传入并进行使用
const sumBind = sum.bind(user, 10, 20)

// 虽然这里的调用形式是默认函数调用
// 但是bind方法绑定的this其优先级高于call/apply 高于默认函数调用
// 所以函数内部的this指向的是user对象
sumBind(30, 40)
/*
  => { name: 'Klaus' }
  => 10 20 30 40
*/
```



## 内置函数中的this

1. 定时器函数中的this 无论在严格模式还是非严格模式下 指向的都是globalThis

2. 对于DOM事件，内部callback中的this指向的都是 唤起事件的那个dom对象

3. 对于大部分JavaScript内置高阶函数，如果指定了thisArg，那么this指向的就是thisArg

   如果没有指定thisArg，那么此时内部callback中的this指向的 

   ​	在非严格模式下 就是globalThis

   ​    在严格模式下 就是undefined

```js
const arr = ['Klaus', 'Alex']

arr.forEach(function() {
  console.log(this) // => { name: 'Klaus' }
}, { name: 'Klaus' })
```



## 规则优先级

`new > bind > apply/call > 隐式绑定 > 默认绑定`

1. new绑定和call、apply是不允许同时使用的，所以不存在谁的优先级更高

2. call 和 apply 也不可能同时使用，所以也不存在谁的优先级更高 



## 特殊规则

1. 如果在显示绑定中，我们传入一个null或者undefined，那么这个显示绑定会被忽略，使用默认规则
2. 创建一个函数的 间接引用，这种情况使用默认绑定规则

```js
function foo() {
  console.log(this)
}

const bar = {
  foo
}

const baz = {}

// console.log(baz.foo = bar.foo) -> foo函数对象
;(baz.foo = bar.foo)() // => globalThis
```

3. 箭头函数内部不会去绑定this，如果在箭头函数内部使用this，其会按照作用域链规则依次进行查找



## 箭头函数

箭头函数是ES6之后增加的一种编写函数的方法

1. 箭头函数不会绑定this、arguments属性

   如果需要在箭头函数中使用arguments，推荐使用剩余参数进行替代

   如果在箭头函数内部使用this，箭头函数内部不会绑定对应的this

   所以其会沿着作用域链来查找对应的this所指向的值

   `默认情况下，在全局环境中，存在对应的this，其指向的是globalThis`

   所以如果一直没有找到对应的this值的时候，最后对应的this值一定是globalThis

2. 箭头函数不能作为构造函数来使用

   + 在早期的JS中，任意一个函数都可以作为普通函数调用，也可以作为构造函数进行调用
   + 而在ES6开始，将函数的这两种行为进行了拆分
   + 箭头函数，因为内部没有原型对象，所以其只能被作为普通函数进行调用
   + 而ES6中的类 只能通过new方式来进行调用，而不可以作为普通函数进行调用



### 编写规则

1. 如果只有一个参数()可以省略

2. 如果函数执行体中只有一行代码, 那么可以省略大括号,

   并且这行代码的返回值会作为整个函数的返回值被返回

   所以 此时如果存在return关键字的时候，需要将return关键字一起省略

3. 如果函数执行体只有返回一个对象, 那么需要给这个对象加上()

   以表示这是一个对象整体，而不是对应的函数代码体



## 综合练习

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

// 箭头函数内部没有this，其会沿着作用域链依次进行查找
// 并且其是在函数定义的上层作用域进行查找，而不是在函数运行处的上层作用域进行查找
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
person1.obj.foo2().call(person2) // 上层作用域查找: obj(隐式绑定)
```



