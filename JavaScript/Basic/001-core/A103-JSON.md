1. 服务器和客户端的数据交换格式
2. JSON全称 JavaScript Object Notation (JavaScript对象符号)
3. 最开始应用于JavaScript, 现在各个语言中都可以使用
4. 现常用于服务器和客户端的数据交互格式和配置文件格式



**其它格式**

| 格式     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| XML      | 1. 早期数据交互格式 「 已被淘汰 」<br />2. 目前作为一些框架的配置文件使用 「 如Java 」 |
| protobuf | 1. google推出的新数据交互格式<br />2. 交互性能比JSON优异，但兼容性相对较差 |



### 基本语法

JSON支持三个类型值

1. 简单值
2. 对象值
3. 数组值

> JSON仅支持普通对象和数组，别的类型对象一律不支持



**简单值**

1. 简单值就是JavaScript的基本数据类型
2. 不支持undefined，bigInt，symbol等别的语言中不存在的类型
3. 字符串必须使用双引号进行包裹



**对象值**

1. key只能是数字或字符串，且必须使用双引号包裹
2. 值可以是三种类型值的任意一个



**数组值**

1. 数组值可以是三种类型值中的任意一种
2. 最后一位后不可以有逗号



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



### toJSON

```js
const user = {
  name: 'Klaus',
  age: 23,
  friend: {
    name: 'Alex',
    age: 22
  },
  toJSON() {
    return this.name + ' - ' + this.age
  }
}

// 如果当前对象实现了toJSON方法，会使用toJSON代替默认序列化方法
// 调用方式类似于`user.toJSON()`，所以toJSON内部this指向对象本身
console.log(JSON.stringify(user)) // => "Klaus - 23"
```

