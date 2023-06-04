## JSON

在目前的开发中，JSON是一种非常重要的数据格式，它并不是编程语言，而是一种可以在服务器和客户端之间传输的数据格式。

JSON的全称是JavaScript Object Notation(JavaScript对象符号)

JSON是由Douglas Crockford构想和设计的一种轻量级资料交换格式，算是JavaScript的一个子集

虽然JSON被提出来的时候是主要应用JavaScript中，但是目前已经独立于编程语言，可以在各个编程语言中使用

例如我们可以使用json作为网络传输的数据交互格式，使用json作为配置文件格式，使用json作为nosql的数据存储格式



`其它数据交互格式`

| 格式     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| xml      | 早期的网络传输中主要是使用XML来进行数据交换<br /> 这种格式在解析、传输等各方面都弱于JSON，所以目前已经很少在被使用<br /> 仅仅作为一些后端语言的配置文件格式进行使用 |
| protobuf | google发明的一种新的数据交互格式<br />体积和解析，传输都优于json，但是因为这种格式比较新，所以兼容性并不是很好 |



### 基本语法

JSON的支持三种类型的值:

| 值     | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| 简单值 | 数字(Number)、字符串(String，不支持单引号)、布尔类型(Boolean)、null类型<br />JSON不支持undefined类型值 |
| 对象值 | 由key、value组成，key是字符串类型，并且必须添加双引号，值可以是简单值、对象值、数组值<br />tips: JSON只支持普通对象和数组对象，并不支持其它类型的对象(例如函数对象) |
| 数组值 | 数组的值可以是简单值、对象值、数组值<br />数组的最后一位不可以加上逗号 |



### 序列化和反序列化

| 操作     | 说明                          | 方法           |
| -------- | ----------------------------- | -------------- |
| 序列化   | JSON对象 转换成 字符串 的过程 | JSON.stringify |
| 反序列化 | 字符串 转换成 JSON对象 的过程 | JSON.parse     |

```js
const user = {
  name: 'Klaus',
  age: 23,
  friend: {
    name: 'Alex',
    age: 22
  }
}

// JSON.stringify基本使用
console.log(JSON.stringify(user))
// => {"name":"Klaus","age":23,"friend":{"name":"Alex","age":22}}

// JSON.stringify第二个参数可以是一个函数
// 用于在转换过程中 对序列化过程进行自定义操作
// 如果不需要 可以传null，表示使用默认的序列化函数
console.log(JSON.stringify(user, (key, value) => {
  if (key === 'age') {
    return value + 2
  }

  return value
}))
// => {"name":"Klaus","age":25,"friend":{"name":"Alex","age":24}}

// JSON.stringify的第三个参数 表示space参数
// 默认情况下，序列化后输出的JSON字符串是在一行显示的
// 我们可以传递space参数 以使输出的JSON参数更具有可读性
// 传入space参数后，对应的JSON字符串会在合适的地方自动进行换行
// 1. 如果space参数为Number类型值，就使用对应个数的空格作为内容之前的填充
// 2. 如果space参数为String类型值，就使用传入的字符串作为一个空格的填充
// 3. 如果space参数为其它类型值，静默失效，输出结果依旧在一行内进行显示
console.log(JSON.stringify(user, null, 2))
/*
=>
{
  "name": "Klaus",
  "age": 23,
  "friend": {
     "name": "Alex",
     "age": 22
   }
 }
*/

console.log(JSON.stringify(user, null, 'xxx'))
/*
=>
{
xxx"name": "Klaus",
xxx"age": 23,
xxx"friend": {
xxxxxx"name": "Alex",
xxxxxx"age": 22
xxx}
}
*/
```

```js
const jsonStr = '{"name":"Klaus","age":23,"friend":{"name":"Alex","age":22}}'

// 基本使用
console.log(JSON.parse(jsonStr))
// => { name: 'Klaus', age: 23, friend: { name: 'Alex', age: 22 } }

// 可以传入第二个参数
// 类型为函数，用于自定义反序列化过程
console.log(JSON.parse(jsonStr, (key, value) => {
  if (key === 'age') {
    return value + 2
  }

  return value
}))
// => { name: 'Klaus', age: 25, friend: { name: 'Alex', age: 24 } }
```



#### toJSON

```js
const user = {
  name: 'Klaus',
  age: 23,
  friend: {
    name: 'Alex',
    age: 22
  },
  toJSON() {
    // toJSON方法在调用的时候, 是通过当前对象进行调用的
    // 也就是说toJSON方法内部的this指向的是当前对象本身
    return this.name + ' ' + this.age
  }
}

// 如果当前对象实现了toJSON方法，那么在序列化的时候会使用对象的toJSON方法来替代默认的序列化方法
console.log(JSON.stringify(user)) // => Klaus 23
```



## Storage

Storage在web端也被称之为WebStorage, 用来对一些简单数据进行持久化存储

WebStorage主要提供了一种存储机制，可以让浏览器提供一种比cookie更直观的key、value存储方式

无论是localStorage还是sessionStorage，在获取一个不存在的值的时候，返回的都是null，而不是undefined

| 存储方式                  | 说明                                                         |
| ------------------------- | ------------------------------------------------------------ |
| localStorage (本地存储)   | 一种持久化的存储机制<br />在关闭掉网页重新打开时，存储的内容依然保留<br />除非手动删除，否则将永久存在 |
| sessionStorage (会话存储) | 提供的是本次会话的存储，在关闭掉会话时，存储的内容会被清除，更像是一种临时存储机制<br />会话通常是指服务器端和客户端之间的一段时间内的一系列交互<br />也就是说当关闭掉网页或者新开一个新的标签页时，sessionStorage中存储的内容会被移除<br />sessionStorage只能在同一个页签的多个同源页面之间进行storage共享 |



| 属性   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| length | 返回一个整数，表示存储在Storage对象中的数据项数量<br />只读属性 |

| 方法       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| key        | 该方法接受一个数值n作为参数，返回存储中的索引为n的key名称<br />如果对于索引位置上的key不存在，则返回null |
| getItem    | 该方法接受一个key作为参数，并且返回key对应的value<br />如果不存在对应的key，那么就会直接返回null |
| setItem    | 该方法接受一个key和value，并且将会把key和value添加到存储中<br />无则添加，有则修改<br />对应的key和value只能是字符串类型值，如果不是字符串类型值会被自动转换为字符串类型 |
| removeItem | 该方法接受一个key作为参数，并把该key从存储中删除             |
| clear      | 该方法的作用是清空存储中的所有key                            |

```js
// 对于需要在多处使用的值，我们可以将其抽离成一个单独的常量，以后我们在需要使用的地方直接使用对应的常量即可
// 1. 使用常量 可以给我们更好的提示，并且减少出错的可能性
// 2. 如果后期需要修改对应的值，只需要修改常量所存储的值即可，即修改一处即可，而不需要在每个使用的地方都要修改
// 这种类型的常量 约定俗成下 我们一般将这类常量的常量名 全部大写
const ACCESS_TOKEN = 'access_token'

localStorage.setItem(ACCESS_TOKEN, '6Y3tcPWzEkondDl4is9Lwd8E30hT8u4L7unQ9oph7v/SnIpnMn23OA9JtTZktDffiivEfcxQa1FEtCHYQA==')

console.log(localStorage.getItem(ACCESS_TOKEN));
```



### 简单封装

```js
class Cache {
  constructor(type = 'local') {
    this.cache = type === 'local' ? localStorage : sessionStorage
  }

  setItem(key, value) {
    // JSON.parse('undefined') 会直接报错
    // 因为JSON是没有undefined数据类型的
    if (!key || [undefined].includes(value)) {
      throw new TypeError('key 和 value 不能为空')
    }

    if (['object', 'function'].includes(typeof key)) {
      throw new TypeError('key只能是基本数据类型值')
    }

    this.cache.setItem(key, JSON.stringify(value))
  }

  getItem(key) {
    if (['object', 'function'].includes(typeof key)) {
      throw new TypeError('key只能是基本数据类型值')
    }

    return JSON.parse(this.cache.getItem(key))
  }

  removeItem(key) {
    if (['object', 'function'].includes(typeof key)) {
      throw new TypeError('key只能是基本数据类型值')
    }

    this.cache.removeItem(key)
  }

  clear() {
    this.cache.clear()
  }
}

// Test Code
const SESSTION_TEST = 'session_test'
const LOCAL_TEST = 'local_test'

const sessionCache = new Cache('session')
sessionCache.setItem(SESSTION_TEST, { name: 'Klaus', age: 23 })
console.log(sessionCache.getItem(SESSTION_TEST))

const localCache = new Cache()
localCache.setItem(LOCAL_TEST, { name: 'Alex', age: 18 })
console.log(localCache.getItem(LOCAL_TEST))
```

