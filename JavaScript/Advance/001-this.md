this指向的对象称为函数的执行上下文

this的值在运行时被指定，和函数的编写位置没有任何关系

在Javascript中， this的值一般可以分为以下四种情况:

1. 默认绑定 「独立函数调用，直接调用」
   + 在非严格模式中 this指向 globalThis
   + 在严格模式中 this指向 undefined
2. 隐式绑定
3. 显示绑定 「 call，apply和bind 」
4. new绑定



## call / apply

修正this并理解调用函数

call 和 apply 的第一个参数都是this的值

+ 如果是对象，那么this的值就是该对象
+ 如果是原始数据类型 「 除null和undefined外 」
  + 非严格模式，转换为对应包装类
  + 严格模式，直接使用原始数据类型

+ 如果是null和undefined
  + 非严格模式，使用globalThis
  + 严格模式，使用null或undefined


call 和 apply 的 后续参数是 参数列表

+ call 通过 剩余参数的形式 传递参数列表
+ apply 通过 数组的形式 传达参数列表



## bind

修正this，并返回一个新函数，但不立即调用

```js
function fun(name, age) {
  console.log(this, name, age)
}

const foo = fun.bind({ name: 'thisArg' }, 'Klaus')

// 实际传入的参数为 [...bind传入的除this外的所有参数, ...调用foo传入的参数] 
foo(23) // => { name: 'thisArg' } Klaus 23
```

bind返回的那个函数被称之为绑定函数或怪异函数

因为无论返回的绑定函数如何调用，其内部this的值都是bind方法调用时传入的this值



## 回调函数的this

1. 定时器函数中的this 无论在严格模式还是非严格模式下 指是globalThis

2. 对于DOM事件，this的值是 唤起事件的那个dom对象

3. 对于大部分JavaScript内置高阶函数

   + 如果指定了thisArg，那么this指向的就是thisArg

   + 如果没有指定thisArg，使用默认绑定



## 优先级

`new > bind > apply/call > 隐式绑定 > 默认绑定`

1. `new绑定和call、apply是不允许同时使用`，所以不存在谁的优先级更高

2. `call 和 apply 不可能同时使用`，所以也不存在谁的优先级更高 



## 箭头函数

1. 一种特殊的函数表达式写法
2. 箭头函数没有this 和 arguments 以及 super
3. 箭头函数没有显示原型对象，不能通过new调用



## 间接函数引用

```js
const obj = {
  fun() {
    console.log(this)
  }
}

const obj2 = {};

// obj2.fun = obj.fun表达式返回值是 obj.fun函数本身
(obj2.fun = obj.fun)()
```



`示例`

```js
globalThis.name = 'window'

const person1 = {
  name: 'person1',
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

const person2 = { name: 'person2' }

// person1.foo3()的返回值是函数本身，所以是默认绑定，this值是window
person1.foo1()(); 

// person1.foo2()返回箭头函数，不关心内部this
// this按照作用域链查找，外层是person1.foo2函数
// 所以this的值是person1
person1.foo2().call(person2);
```

