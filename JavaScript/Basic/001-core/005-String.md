在JavaScript中，字符串类型可以用双引号、单引号或反引号表示 「 使用双引号和单引号没有区别 」

除了双引号和单引号，还有一种反引号，反引号的使用更加灵活，可以在字符串中嵌入变量或表达式

```js
let result = `2 + 3 = ${2 + 3}`;
```



如果字符串中包含特殊字符（如引号），可以使用转义字符（反斜杠）来处理

```js
let message = "My name is \"Klaus\"";
let path = "C:\\Program Files\\";
```



字符串本身也有一些属性和方法，比如获取字符串长度：

```javascript
let message = "Hello World";
console.log(message.length); // 11
```

字符串之间可以拼接，使用加号运算符：

```javascript
let firstName = "Klaus";
let info = "My name is " + firstName;
```

或者使用反引号（推荐）：

```javascript
let firstName = "Klaus";
let info = `My name is ${firstName}`;
```



## 标签模板字符串

```js
function foo(...args) {
  console.log(args)
}

// 普通调用方式
foo(10, 20, 30)

// 标签模板字符串调用方式
// 标签模板字符串调用本质是一种特殊的函数调用方式
foo`my name is, my age is`
// => [['my name is, my age is']]

const username = 'Klaus'
const age = 24

// 模板字符串会被解析形成如下参数后被传入对应函数:
// 1. 第一个元素是数组，是被模块字符串拆分的字符串组合
// 2. 之后的参数内容是一个个模块字符串传入的变量值
foo`my name is ${username}, my age is ${age}`
// => [ [ 'my name is ', ', my age is ', '' ], 'Klaus', 24 ]
```



## Object.keys/Object.values/Object.entries

Object.keys/Object.values/Object.entries 可以被用于在数组, 对象 和 字符串上
Object.keys/Object.values/Object.entries 获取的属性名和属性值 是自身的属性名和属性值, 不包含原型对象上的对应属性名和属性值



## String Padding

```js
const str = 'Hello'


// 默认填充符为空格
// 第二个参数需要是字符串
// 如果不是字符串会尝试将其转换为字符串后，使用转换后形成的字符串的第一个字符作为填充字符
console.log(str.padStart(10, 'a')) // => aaaaaHello
console.log(str.padEnd(10, 'a')) // => Helloaaaaa
```

