number类型可以细分为

1. 常规数字 ( 正数/负数、整数/小数、0 )
2. NaN --- not a number
3. +Infinity/-Infinity --- 正无穷大/负无穷大



## NaN

1. NaN表示当前值不是一个数值类型值
2. NaN本身是数据类型值
3. NaN和任何数据类型都不相等，包括NaN本身
   + 因为NaN代表不是一个数，那么可能是任何东西，例如字符串，布尔值，对象都可以
   + 所以NaN没有办法进行任何的比较
   + 也不可以通过与NaN进行对比，来判断一个数是否是有效数字

```js
// == 是否相等 === 是否全等
console.log(NaN == NaN) // false
```



## isNaN

检测一个值是否为非有效数字

```js
console.log(isNaN(NaN)) // true --- 不是有效数字
console.log(isNaN(10)) // false --- 是有效数字

// isNaN执行流程:
// 	1. 如果参数不是number，尝试通过Number()将参数转换为number类型值
// 	2. 判断参数是否是一个有效数字 
console.log(isNaN('10')) // false
```



## isInfinite

检测一个值是否无穷大

```js
console.log(Number.MAX_VALUE) // JS可以表示的最大浮点数
console.log(Number.MIN_VALUE) // JS可以表示的最小浮点数

console.log(Number.MAX_SAFE_INTEGER) // JS可以表示的最大整数
console.log(Number.MIN_SAFE_INTEGER) // JS可以表示的最小整数

console.log(isFinite(Number.MAX_VALUE + 1)) // => true
console.log(isFinite(Number.MIN_VALUE - 1)) // => true
```





## 任意类型转number的五种方式

### Number(val)

> 1. 在转换过程中，遇到了空值 (如 null、空字符串), 对应的空值会被当做0进行处理
>
> 2. 浏览器内置的转换规则

`number => number`

```js
Number(10) // => 10
```



`string => number`

1. 参数必须是有效数字才能正常转换
2. 不可以包括任何非有效数字字符 --- 第一个点除外

```js
Number('12.5') // => 12.5
Number('12.5px') // => NaN 
Number('') // => 0 --- 特例
```



`boolean => number`

```js
Number(true) // => 1
Number(false) // => 0
```



`null => number`

```js
Number(null) // => 0
```



`undefined => number`

```js
Number(undefined) // => NaN
```



`object => number`

1. 调用对象的`toString`方法，将对象转换为字符串
2. 再执行Number方法

```js
Number({}) // => NaN ====> {}.toString() => '[object Object]'
Number([1, 2]) // => NaN ====> [1, 2].toStirng() => '1,2'

Number([]) // => 0 ====>  [].toStirng() => ''
Number([12]) // => 12 ====> [12].toString() => 12
```



### parseInt(val, [flag])/parseFloat(val, [flag])

> 不是内置的转换规则，是浏览器额外提供的方法

从左往右 尽可能的提取数字

1. 将参数转换为字符串
2. 尝试对参数值进行转换



`parseInt` --- 遇到第一个非数字字符 就结束

`parseFloat --- 遇到第一个非数字字符 就结束 (第一个点 除外)

```js
parseInt('12.5px') // => 12
parseInt('12.5px12') // => 12
parseInt('width: 12.5px') // => NaN

parseInt('width: 12.5px') // => NaN
parseInt('12.5px12') // => 12.5
parseInt('12.5px') // => 12.5
```



###  == 

相等判断，等式两边类型不一致，将等式两边值都转换为number类型值后再进行判断

```js
'10' == 10 // => true
```



`四则运算`

四则运算中，如果运算元素不是数值，会尝试先将其转换为数值

字符串拼接除外
