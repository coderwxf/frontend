对象和基本数据类型一起运算的时候，会将对象转换为基本数据类型后再使用

对象转基本数据类型使用的是内部的`toPrimitive`, 可以通过`Symbol.toPrimitive`重写内部的`toPrimitive`方法



## toString

1. 将一个对象转换为对应的字符串形式，该方法返回一个表示该对象的字符串
2. 对象输出`[object Object]`
3. 部分对象重写了自己的`toString`方法

```js
const obj = {}
const arr = [1, 2, 3]
const fun = () => {}
const err = new Error('我是错误信息')
const date = new Date()

console.log(typeof obj.toString(), obj.toString()) // => string [object Object]

// 数组输出的是arr.join(',') ---- 如果是空数组，返回的就是空字符串
console.log(typeof arr.toString(), arr.toString()) // => string 1,2,3

// 函数返回函数体本身
console.log(typeof fun.toString(), fun.toString()) // => string () => {}

console.log(typeof fun.toString(), fun.toString()) // => string "Error: 我是错误信息"

console.log(typeof date.toString(), date.toString())
// => string Fri Nov 05 2021 13:57:12 GMT+0800 (中国标准时间)
```



`toString是最精确的类型判断方式之一`

```js
console.log(toString.call(''))           // => [object String]
console.log(toString.call(22))           // => [object Number]
console.log(toString.call(undefined))    // => [object Undefined]
console.log(toString.call(null))         // => [object Null]
console.log(toString.call(new Date))     // => [object Date]

console.log(toString.call(Math))         // => [object Math]
console.log(toString.call(globalThis))   // => [object global]

console.log(toString.call(()=>{}))       // => [object Function]
console.log(toString.call({}))           // => [object Object]
console.log(toString.call([]))           // => [object Array]

console.log(toString.call(new Set()))    // => [object Set]
console.log(toString.call(new Map()))    // => [object Map]
```



### Symbol.toStringTag

```js
const user = {
  name: 'Klaus',
  age: 23,

  // Symbol.toStringTag 是一个get访问器函数
  get [Symbol.toStringTag]() {
    // Symbol.toStringTag方法需要返回一个字符串类型的值
    // 如果返回的值的类型不是字符串类型，js会直接使用默认的type类型
    // 例如 如果这里的Symbol.toStringTag方法返回值为2, 是一个number类型值
    // 那么user.toString()的结果为 [object Object]
    // 也就是直接使用默认的type值 Object, 而不会尝试将2转换为'2'
    return this.name
  }
}

console.log(user.toString()) // => [object Klaus]
```



## valueOf

1. 返回当前对象的原始值
2. 如果当前对象无法获取到对应的基本数据类型，那么会将对象本身原封不动的返回

```js
// valueOf --- 如果能返回原始值，就返回原始值
const num = 123
const bool = true
const str = 'Klaus'

console.log(num.valueOf()) // => 123
console.log(bool.valueOf()) // => true
console.log(str.valueOf()) // => Klaus

// valueOf --- 如果无法返回原始值，就返回参数对象本身
const obj = {}
const arr = [1, 2, 3]
const fun = () => {}

console.log(obj.valueOf()) // => {}
console.log(arr.valueOf()) // => [ 1, 2, 3 ]
console.log(fun.valueOf()) // => [Function: fun]

// Date对象 重写了valueOf方法 --- 返回时间所对应的时间戳
const date = new Date()
console.log(date.valueOf()) // => 1636092865039
```



## toPrimitive

```js
// toPrimitive的伪代码形式如下:
toPrimitive(target, PreferredType = 'default': 'string' | 'number')
```

+ 如果没有定义PreferredType, 则默认为`default`

+ 如果PreferredType 的值为`default`，执行流程和PreferredType的值为`number`时的执行流程一致



PreferredType的值为number

+ 优先调用valueOf方法

+ 如果valueOf方法的返回值不是基本数据类型值，则继续调用toString方法

+ 如果toString方法返回值是基本数据类型值，则将基本数据类型转换为number类型值
+ 如果toString方法的返回值依旧不是一个基本数据类型值，则抛出一个异常，直接报错



PreferredType的值为string

+ 优先调用toString方法
+ 如果toString方法的返回值是基本数据类型值，则直接将该值转换为string类型值
+ 如果valueOf方法返回值是基本数据类型值，则将基本数据类型转换为string类型值
+ 如果valueOf方法的返回值依旧不是一个基本数据类型值，则抛出一个异常，直接报错



## [Symbol.toPrimitive]

如果实现了`Symbol.toPrimitive`方法，JS就会使用`Symbol.toPrimitive`来将引用类型转换为基本数据类型，而不会再去调用原生的`valueOf`和`toString`

```js
class Foo {
  constructor(num) {
    this.num = num
  }

  
  // 重写valueOf方法
  valueOf() {
    console.log('valueOf')
    return this.num
  }

  // 重写toString方法
  toString() {
    console.log('toString')
    return this.num + ''
  }

  // hint有三种取值可能性 string | number | default（缺省值）
  [Symbol.toPrimitive](hint) {
    if (hint === 'number') {
      // hint --> number
      return this.num
    } else if (hint === 'string') {
      // hint --> string
      return this.num + ''
    }

    // hint --> default
    return false
  }
}

// TEST CODE
const foo = new Foo(10)

console.log(Number(foo)) // => 10
console.log(String(foo)) // => '10'

console.log(`${foo}`) // => '10'
console.log(foo / 2) // => 5

// 注意: 比较运算和加法运算的时候，hint的值是default，并不是number
// 但是值为default的时候，其流程和number一致
console.log(foo == 10) // => false == 10 -> false
console.log(foo + 2) // => false + 2 -> 0 + 2 -> 2
console.log(foo + '2') // => false + '2' -> 'false2'

// 全等操作符在运算的时候，不会进行任何的类型转换
// 所以Symbol.toPrimitive方法在这里并不会被执行
console.log(foo === '10') // => false

// 在将对象转换为boolean类型值时，会直接进行转换
// 而不会在此过程中调用valueOf方法 或 toString方法
console.log(!!foo) // => true
```



有些内置对象重写了自己的`Symbol.toPrimitive`，因此他们有自己的转换规则

```js
// 保存原始的valueOf方法和toString方法
const valueOf = Object.prototype.valueOf
const toString = Object.prototype.toString

// 添加valueOf日志
Object.prototype.valueOf = function () {
  console.log('valueOf')
  return valueOf.call(this)
}

// 添加toString日志
Object.prototype.toString = function () {
  console.log('toString')
  return toString.call(this)
}

const dateValueOf =  Date.prototype.valueOf
const dateToString =  Date.prototype.toString

// 添加Date的valueOf日志
Date.prototype.valueOf = function () {
  console.log('date valueOf')
  return dateValueOf.call(this)
}

// 添加Date的toString日志
Date.prototype.toString = function () {
  console.log('date toString')
  return dateToString.call(this)
}

const date = new Date()

// Date重写了自己对应的toString方法和valueOf方法
// 对于Date实例而言，加法运算和判等运算 会调用Date.prototype.toString方法
// 而不是和普通对象那样去调用valueOf方法
console.log(date + 1)
console.log(date == 2)
```



### 示例

#### 如何让`a===1 && a===2 && a===3`和`a==1 && a==2 && a==3`的判断结果为true

判等可能会触发`隐式类型转换`，所以可以使用 `valueOf` 来实现

```js
class A {
  constructor(value) {
    this.value = value;
  }

  valueOf() {
    return this.value ++
  }
}

const a = new A(1)
console.log(a == 1 && a == 2 && a == 3) // => true
```



而全等并不会进行类型转换，只能通过`defineProperty`或`Proxy`来进行监听

```js
let target = {
  a: 0
}

const proxy = new Proxy(target, {
  get(target, key, receiver) {
    if (key === 'a') {
      target[key]++
      return Reflect.get(target, key, receiver)
    }
  }
})

console.log(proxy.a===1 && proxy.a===2 && proxy.a===3) // => true
```

