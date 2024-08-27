this是函数调用时，被动态绑定的一个对象 「 运行时绑定，和函数编写位置无关 」

this指向的那个对象称之为 函数执行上下文，其对应的值默认情况下是函数的调用者

 this的值一般可以分为以下四种情况:

1. 默认绑定 「独立函数调用，直接调用」
   + 在非严格模式中 this指向 globalThis
   + 在严格模式中 this指向 undefined
2. 隐式绑定
3. 显示绑定 「 call，apply和bind 」
4. new绑定



## call / apply

修正this并立即调用函数

```js
Function.prototype.call(this, arg1, arg2, ...)

Function.prototype.apply(this, [arg1, arg2, ...])
```

this绑定规则

1. 如果是对象，直接使用
2. 如果不是对象
   + 非严格模式，使用包装类对象，如果没有包装类，就使用默认绑定规则
   + 严格模式，直接使用，不进行任何转换



### 模拟实现

```js
// 公共逻辑
Function.prototype.__bindThisAndExec__ = function(thisArg, ...args) {
  const fnSymbol = Symbol('__fn__')

  // 将this『 函数对象本身 』 挂载到 thisArg上
  Object.defineProperty(thisArg, fnSymbol, {
    value: this,
    configurable: true // 便于后续移除函数对象
  })

  const res = thisArg[fnSymbol](...args)

  // 移除临时挂载的函数
  Reflect.deleteProperty(thisArg, fnSymbol)

  return res
}

// 模拟apply
Function.prototype.customApply = function(thisArg, args) {
  // 如果是null和undefined, thisArg 变成 globalThis
  // 其余情况，尽可能转换为对应对象形式
  const thisArgument = [null, undefined].includes(thisArg) ? globalThis : Object(thisArg)

  return this.__bindThisAndExec__(thisArgument, ...args)
}

// 模拟call
Function.prototype.customCall = function(thisArg, ...args) {
  const thisArgument = [null, undefined].includes(thisArg) ? globalThis : Object(thisArg)

  return this.__bindThisAndExec__(thisArgument, ...args)
 }

// TEST CODE
function foo(name, age) {
  console.log(this)
  return `${name} - ${age}`
}

const thisArg  = {
  name: 'Klaus'
}

const applyRes = foo.customApply(thisArg, ['Klaus', 23])
console.log(applyRes)

const callRes = foo.customCall(thisArg, 'Klaus', 23)
console.log(callRes)
```



## bind

修正this，并返回一个新函数，但不立即调用

返回的新函数，被称之为怪异函数或绑定函数

因为返回的新函数无论以何种方式进行调用，其内部this的值都是bind调用时候绑定的this，和调用方式无关

```js
function fun(name, age) {
  console.log(this, name, age)
}

const foo = fun.bind({ name: 'thisArg' }, 'Klaus')

// 实际传入的参数为 [...bind传入的除this外的所有参数, ...调用foo传入的参数] 
foo(23) // => { name: 'thisArg' } Klaus 23
```



### 模拟实现

```js
Function.prototype.customBind = function(thisArg, ...args) {
  // 返回的需要是个箭头函数，以便于返回的函数内部this指向正确
  return (...params) => {
    const thisArgument = [null, undefined]?.includes(thisArg) ? globalThis : Object(thisArg)
    
    const fnSymbol = Symbol('__fn__')

    Object.defineProperty(thisArgument, fnSymbol, {
      value: this,
      configurable: true
    })

    return thisArgument[fnSymbol](...args, ...params)
    // 调用完后，不用删除fnSymbol, 因为bind返回的绑定函数可以被多次调用
  }
}

// Test Code
function fun(name, age) {
  console.log(this)
  return `${name} - ${age}`
}

const foo = fun.customBind({name: 'Klaus'}, 'Klaus')

console.log(foo(23))
console.log(foo(18))
```



## 回调函数的this

回调函数并不是我们自己调用的，而是由另一个函数调用的

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

// obj2.fun = obj.fun的返回值就是obj.fun
(obj2.fun = obj.fun)()
```



`示例`

```js
// 如果是浏览器，全局name是window 「 var声明变量会挂载到window对象上，且默认window.name的值是空字符串 」 
// 如果是node环境，全局name的值是undefined 「 var/let/const声明变量都不会被挂载到global上，且默认不存在global.name 」
var name = 'window'

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

// person1.foo3()的返回值是函数本身，所以是默认绑定，this值是globalThis
person1.foo1()(); 

// person1.foo2()返回箭头函数，不关心内部this, 按作用域链查找this值
// 所以this的值是person1
person1.foo2()();
```

