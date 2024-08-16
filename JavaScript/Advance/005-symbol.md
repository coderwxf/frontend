Symbol值是通过Symbol函数来生成的，生成后可以作为属性名，也就是在ES6中，对象的属性名可以使用字符串，也可以使用Symbol值

```js
// Symbol是函数 不是构造函数，不需要通过new关键字来进行调用
const s1 = Symbol() // => Symbol()
const s2 = Symbol()

// Symbol即使多次创建值，它们也是不同的：Symbol函数执行后每次创建出来的值都是独一无二的
console.log(s1 == s2) // => false
```



可以在创建的时候传入一个字符串 「 非字符串会转字符串后使用 」 作为symbol变量的标识符，区分用

```js
const s1 = Symbol('name')
console.log(s1) // => Symbol(name)
```



Symbol 只能转布尔和字符串类型

```js
const symbol = Symbol()

// console.log(symbol + 2) => error

// Symbol值可以转换为字符串和布尔值
console.log(symbol.toString()) // => 'Symbol()'
console.log(!!symbol) // => true
```



创建两个相同的symbol

```js
// Symbol.for方法会先根据key去进行查找
// 如果不传参数，也会被认为是相同的key
// 如果对应的key已经被创建就直接返回对应的symbol值
// 如果对应的key没有被创建，那么会新建一个新的symbol值，并将其返回
const s1 = Symbol.for()
const s2 = Symbol.for()
console.log(s1 === s2) // => true
```



```js
// 对于description, 可以通过symbol的description属性去进行获取
let s = Symbol('bar')
console.log(s.description) // => bar

// 对于key, 可以通过symbol的keyFor方法进行获取
s = Symbol.for('baz')
console.log(Symbol.keyFor(s)) // => baz
```



for-of 和 Object.keys，Object.getOwnPropertyNames 遍历出来的key只能是可迭代的非symbol属性

JSON.stringify 方法将对象转换为字符串的时候，也会自动过滤symbol属性

如果我们需要获取到Symbol属性值，需要使用Object.getOwnPropertySymbols方法

使用Reflect.ownKeys 可以获取对象自身的所有的属性名，包括symbol属性名，可迭代属性名和不可迭代属性名



常见的内置symbol值

1. Symbol.iterator
2. Symbol.toPrimitive

```js
const obj = {
  name: '',

  // 将对象转换为基本数据类型时，会调用Symbol.toPrimitive指向的方法
  // 1. 当hint值为number， 表示 对象 -> number
  // 2. 当hint值为string， 表示 对象 -> string
  // 3. 当hint值为default， 表示 对象 -> string | number
  //    + 加号可能是数字相加 也可能是字符串破解 --- hint的值就是default
  //    + 判等操作(即 == 和 != --> 全等操作不会进行任何隐式类型转换) --- hint的值也是default

  // 对象 -> boolean --- 不会调用Symbol.toPrimitive指向的方法
  [Symbol.toPrimitive]: hint => {
    console.log(hint)

    if (!hint) {
      console.log('boolean')
    }

    if (hint === 'number') {
      return 0
    } else if (hint === 'string') {
      return ''
    } else {
      return false
    }
  }
}

console.log(!!obj)
```

