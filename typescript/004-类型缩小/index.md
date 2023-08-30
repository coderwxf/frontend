TypeScript的类型系统不仅可以进行静态类型分析，还可以进行控制流分析。通过分析程序可能采取的不同路径，TypeScript可以推断出给定位置上值的最具体可能类型



TypeScript看到typeof padding === "number"，并将其理解为一种特殊形式的代码，称为类型保护（type guard）, 达到类型缩小的效果

```ts
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  
  return padding + input;
}
```



## 类型缩小的方式

### typeof

在TypeScript中，针对typeof返回值进行检查就是一种类型守卫



### 真值缩小

在JavaScript中，我们可以在条件语句、&&运算符、||运算符、if语句、布尔取反（！）等中使用任何表达式

以下这些值：

0
NaN
""（空字符串）
0n（bigint类型的零）
null
undefined

都会被强制转换为false，而其他值会被强制转换为true

```ts
Boolean("hello"); // boolean
!!"world";  // true --- 字面量类型
```



### 想等性缩小

TypeScript也使用switch语句和相等性检查（例如`===、!==、==和!=`）来缩小类型

JavaScript使用"=="和"!="进行宽松的相等性检查时，也会正确地进行缩小范围的判断



### in操作符

在JavaScript中，有一个用于确定对象或其原型链中是否存在具有特定名称属性的运算符：in运算符。TypeScript将其视为一种缩小潜在类型的方式。

"value" in x，---- X需要是联合类型

```ts
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```



### instanceOf

JavaScript中有一个运算符用于检查一个值是否是另一个值的"实例"

具体来说，在JavaScript中，x instanceof Foo会检查x的原型链是否包含Foo.prototype



### 等式赋值

```ts
let x = Math.random() < 0.5 ? 10 : "hello world!";
   
// 虽然x被缩小为了number 但是初始类型是string | number
// 所以可以被重新赋值字符串
let x: string | number 
x = 1; // number
x = "goodbye!"; // string
```



### 控制流分析

这种基于可达性的代码分析被称为控制流分析，TypeScript在遇到类型守卫和赋值时使用该流分析来缩小类型范围。



### 类型谓词

代码是在运行时执行的 但我们可以通过类型谓词让tsc可以在编译阶段获取到具体的返回值类型

```ts
// 函数返回值的类型是 类型谓词
// 参数是联合类型  --- 一般是两种类型的联合
// parameterName is Type --- 被称之为类型谓词
// parameterName 必须是参数列表中的形参名
function isFish(pet: Fish | Bird): pet is Fish {
  // 函数的返回值是布尔类型值
  // 如果为true  断言成功
  // 否则断言失败
  return (pet as Fish).swim !== undefined;
}
```



## 断言函数

