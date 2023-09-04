TypeScript的类型系统旨在使编写典型的JavaScript代码尽可能容易，而不需要过多强调类型安全

，TypeScript的目标是以最少的干扰方式为现有的JavaScript结构提供类型



TypeScript的类型系统不仅可以进行静态类型分析，还可以进行控制流分析。通过分析程序可能采取的不同路径，TypeScript可以推断出给定位置上值的最具体可能类型



TypeScript看到typeof padding === "number"，并将其理解为一种特殊形式的代码，称为类型保护（type guard）, 达到类型缩小的效果

它会检查这些特殊的检查（称为类型保护）和赋值，将类型细化为比声明更具体的类型，这个过程被称为缩小类型范围。

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

TypeScript 期望它返回以下一组字符串：

"string"（字符串）
"number"（数字）
"bigint"（大整数）
"boolean"（布尔值）
"symbol"（符号）
"undefined"（未定义）
"object"（对象）
"function"（函数）



### 真值缩小

在JavaScript中，我们可以在条件语句、&&运算符、||运算符、if语句、布尔取反（！）等中使用任何表达式

在JavaScript中，像if这样的结构首先会将其条件“强制”转换为布尔值，然后根据结果是true还是false选择不同的分支

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

在JavaScript中，有一个用于确定对象或其原型链中是否存在具有特定名称属性的运算符：in运算符。

TypeScript将其视为一种缩小潜在类型的方式。

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

JavaScript提供了一个运算符用于检查一个值是否是另一个值的“实例”。

具体来说，在JavaScript中，x instanceof Foo检查x的原型链是否包含Foo.prototype



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

这种基于可达性的代码分析被称为控制流分析（control flow analysis），TypeScript在遇到类型保护和赋值时使用这种流程分析来缩小类型



### 类型谓词

要定义一个用户自定义的类型保护（type guard），我们只需要定义一个返回类型为类型谓词（type predicate）的函数：

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

断言函数是一组特定的函数，如果出现意外情况，它们会抛出错误。它们被称为"断言"函数。

执行完毕 说明条件成立 否则函数会因为抛出异常而被迫中止

```ts
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new AssertionError(msg);
  }
}
```

```ts
function assertIsString(val: any): asserts val is string {
  if (typeof val !== "string") {
    throw new AssertionError("Not a string!");
  }
}
```

在TS中 类型谓词可以被认为是一种特殊的断言函数



## 区分联合类型

我们可以尝试使用非空断言（!）（在shape.radius之后加上！）来表明半径肯定存在

当联合类型的每个成员都包含具有字面量类型的公共属性时，TypeScript将其视为区分联合（discriminated union），并可以缩小联合的成员。

在这种情况下，kind就是这个公共属性（被认为是Shape的区分属性）

```ts
interface Circle {
  kind: "circle";
  radius: number;
}

interface Square {
  kind: "square";
  sideLength: number;
}

type Shape = Circle | Square;

function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
    // (parameter) shape: Circle
  }
}
```



## never

当缩小类型联合的范围时，你可以将选项减少到一种情况，即排除所有可能性，不再剩下任何选项

在这种情况下，TypeScript会使用`never`类型表示一种不应存在的状态。



### 穷尽性检查

`never`类型可以分配给任何类型，但是没有类型可以分配给`never`（除了`never`本身）

```ts
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    // 如果联合类型没有处理完全 就会将没有处理的类型赋值给never 从而报错
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

