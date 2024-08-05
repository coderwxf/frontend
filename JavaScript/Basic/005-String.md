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

