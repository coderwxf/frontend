symbol是es6中新添加的基本数据类型，通过Symbol函数进行创建

## 基本使用

```js
// 创建一个symbol值, JS Engine 会在内存中创建一个唯一值
// 如果输出symbol 会直接输出 Symbol()
const s1 = Symbol() // => Symbol()

// 可以在创建symbol值的时候 传入描述符 以区分不同的symbol值
// 描述符需要是字符串类型 如果不是字符串类型 会被转换为字符串值后再进行使用
// 如果使用TS 那么描述符必须是 字符串类型 或 数值类型
const s2 = Symbol('Klaus') // => Symbol('Klaus')
```

```ts
const s1 = Symbol()
const s2 = Symbol()

// 每一个symbol值都是独一无二的
console.log(s1 === s2) // => false
```

```ts
// symbol值 不可以进行运算 但是可以被转换为字符串 或 布尔值
const s1 = Symbol

// console.log(s1 + 3) => error

console.log(s1.toString) // => Symbol()
console.log(Boolean(s1)) // => true
```

```ts
// symbol值的最主要作用是作为属性名
// symbol作为独一无二的值作为属性名 可以避免属性名重复问题
const s1 = Symbol()

const obj = {
  [s1]: 'Klaus'
}

// symbol值只能通过方括号语法去进行访问
console.log(obj[s1])
```

```ts
// Symbol.for创建的时候需要指定key，也就是描述符
// Symbol.for在创建的时候，会判断有没有通过Symbol.for创建的symbol值，且key是相同的
// 如果相同，直接复用之前已经创建的symbol值，没有相同的值就重新创建一个全新的symbol值

/*
  这里进行类型注解是因为在ts中，无法评估 Symbol.for 生成的symbol的相等性
  const s1 = Symbol.for('Klaus')
  const s2 = Symbol.for('Klaus')
  console.log(s1 === s2) // error
  需要显示声明任意一个symbol对应的值，这是ts类型系统的内部bug
*/
const s1: symbol = Symbol.for('Klaus')
const s2 = Symbol.for('Klaus')
const s3 = Symbol('Klaus')

console.log(s1 === s2) // => true
console.log(s1 === s3) // => false
```

```ts
// 要测试以下代码需要将ts配置文件中的target调整为ESNext
// 因为这是ES10中才出现的语法
const s1 = Symbol('Klaus')
console.log(s1.description) // => Klaus
console.log(Symbol.keyFor(s1)) // => undefined

const s2 = Symbol.for('Alex')
console.log(s2.description) // => Alex
console.log(Symbol.keyFor(s2)) // => Alex
```



## 迭代

```ts
const s1 = Symbol()

const obj = {
  [s1]: 'Klaus',
  age: 23,
  __proto__: {
    sex: 'male'
  }
}

// for-in 会输出自身和原型上所有的 可迭代非symbol属性
for(const key in obj) {
  console.log(key)
  // age sex
}

// 仅输出自身上 所有的 可迭代非symbol属性
console.log(Object.keys(obj)) // => ['age']

// 仅输出自身上 所有的 可迭代非symbol属性
console.log(Object.getOwnPropertyNames(obj)) // => ['age']

// 在JSON中不存在symbol类型值，所以在转换为JSON字符串的时候
// symbol值会被移除
console.log(JSON.stringify(obj))
```

```ts
const s1 = Symbol()

const obj = {
  [s1]: 'Klaus',
  age: 23,
  __proto__: {
    sex: 'male'
  }
}

// 获取所有的symbol值
console.log(Object.getOwnPropertySymbols(obj)) // => [ Symbol() ]

// 获取所有的属性值
console.log(Reflect.ownKeys(obj)) // => [ 'age', Symbol() ]
```



## 内置Symbol值

JavaScript提供了一系列的内置Symbol值

可以简单理解为这些内置Symbol值就类似于回调函数，用于扩展JS 原生API的行为

### iterator

可迭代对象的迭代器

```ts
class Person {
  name: string
  age: number

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }

  // 用于返回迭代器 --- 实例方法
  [Symbol.iterator]() {
    let index = 0
    const entries = Object.entries(this)

    // 迭代器必须有一个next方法
    // next方法需要返回一个对象
    // next返回的对象必须有value和done属性
    // done -- boolean 为true时候 迭代完成
    // value -- 值 本次迭代时候的返回结果
    return {
      next: () => ({
        done: index === entries.length,
        value: entries[index++]
      })
    }
  }
}

const per = new Person('Klaus', 26)

for (const item of per) {
  console.log(item)
}
/*
  =>
  [ 'name', 'Klaus' ]
  [ 'age', 26 ]
*/
```

```ts
// 上述代码可以被简化成如下代码
class Person {
  name: string
  age: number

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }

  *[Symbol.iterator]() {
    yield* Object.entries(this)
  }
}

const per = new Person('Klaus', 26)

for (const item of per) {
  console.log(item)
}
```



### toPrimitive

将一个对象转换为JS原生对象的时候，就会调用toPrimitive

如果实现了`Symbol.toPrimitive`方法, 就不会再去调用原生的valueOf`和`toString方法

```js
class Foo {
  num: number

  constructor(num: number) {
    this.num = num
  }

  // 因为实现了Symbol.toPrimitive方法 所以这里的valueOf和toString就不会再被执行了

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
  // Symbol.toPrimitive必须返回一个元素值 --- 如果不是原始值会尝试将其转换为原始值后再进行返回
  [Symbol.toPrimitive](hint: 'string'|'number'|'default') {
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
// @ts-ignore
console.log(foo / 2) // => 5

// 注意: 比较运算和加法运算的时候，hint的值是default，并不是number
// @ts-ignore
console.log(foo == 10) // => false == 10 -> false
// @ts-ignore
console.log(foo + 2) // => false + 2 -> 0 + 2 -> 2
console.log(foo + '2') // => false + '2' -> 'false2'

// 全等操作符在运算的时候，不会进行任何的类型转换
// 所以Symbol.toPrimitive方法在这里并不会被执行
// @ts-ignore
console.log(foo === '10') // => false

// 在将对象转换为boolean类型值时，会直接进行转换
// 而不会在此过程中调用valueOf方法 或 toString方法
console.log(!!foo) // => true
```



### toStringTag

```js
class Person {
  name: string
  age: number

  constructor(name: string, age: number) {
    this.name = name
    this.age = age
  }

  // Symbol.toStringTag 是一个get方法
  // Symbol.toStringTag的返回值用于替换object Object中的Object
  get [Symbol.toStringTag]() {
    return this.name
  }
}

const per = new Person('Klaus', 26)

console.log(per.toString()) // => [object Klaus]
```

