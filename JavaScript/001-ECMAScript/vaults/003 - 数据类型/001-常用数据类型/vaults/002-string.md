string可以分为

+ 单引号包裹
+ 双引号包裹
+ 反引号包裹 --- 模板字符串



## 其余类型转换为字符串

### [val].toString()

```js
// 基本数据类型值 调用toString方法需要使用小括号包裹
(12).toString() // => '12'
(NaN).toString() // => 'NaN'
(true).toString() // => 'true'

// 引用类型值 可以直接调用toString方法
[].toString() // => ''
[12].toString() // => '12'
[12, 34].toString() // => '12,34'

/^$/.toString() // => '/^$/'zq
```

```js
// null 和 undefined 没有对应的包装类 --- 无法调用toString方法
(null).toString() // error
(undefined).toString() // error
```

```js
// 特例
// 1. Object.prototype.toString() --- 检测数据类型
// 2. 大部分对象 重写原型上的toString方法
({}).toStirng() // '[object Object]'
```



### 字符串拼接

四则运算中，如果遇到了加法，且存在字符串 即使字符串拼接

```js
'10' + 10 // => 1010
```



其余情况下，将参数使用`Number()`转数值后，再进行运算

```js
'10' - 10 // => 10 - 10 => 0
'10px' - 10 // => NaN - 10 => NaN
```



null 和 undefined 可以通过字符串拼接 转换为字符串

```js
null +  '' // => 'null'
undefined + '' // => 'undefined'
```



即使是在类型转换的过程中，遇到了字符串 也是字符串拼接

```js
// 10 + '' --> 此时数组变成空字符串后直接执行字符串拼接，而不是再将空字符串转换为数字0
10 + [] // => '10'

10 + ['12'] // => 1012
```

```js
let a = 100 + true + 21.2 + null + undefined + 'Hello' + [] + null + 9 + false
console.log(a) // NaNHellonull9false
```





