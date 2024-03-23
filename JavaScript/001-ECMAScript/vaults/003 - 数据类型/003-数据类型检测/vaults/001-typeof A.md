## 特点

1. 是一个`运算符`

2. 返回值 是 一个`全小写的字符串`， 代表了参数的数据类型

   > 一般情况下，在浏览器的控制台中，
   >
   > + 字符串类型值 使用黑色字体表示
   >
   > + 数值类型值 使用蓝色字体表示

```js
typeof NaN // => "number"
typeof typeof NaN // => "string" 『 typeof 的返回值是 字符串 』
```



## 使用

`基本数据类型检测`

```js
typeof 1 // => "number"
typeof 'Hello' // => "string"
typeof true // => "boolean"
typeof undefined // => "undefined"
typeof Symbol() // => "symbol"
```

```js
// null 空对象指针 不是对象，但被归为对象 --- typeof的局限性
typeof null // => "object"
```



`引用数据类型检测`

```js
typeof {} // => "object"
typeof [] // => "object" --- typeof的局限性
typeof () => {} // => "function"
```



## 局限性

1. 将null 单独归为了对象，而不是普通数据类型
2. 对于引用数据类型，只能区分函数类型，别的引用类型一律视为object，无法进行更近一步的细分

