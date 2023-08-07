在本章中，我们将介绍在JavaScript代码中最常见的值类型，并解释在TypeScript中描述这些类型的方式。这不是一个详尽无遗的列表，未来的章节将介绍更多命名和使用其他类型的方法。

类型不仅仅可以出现在类型注解中，还可以出现在许多其他位置。随着我们学习类型本身，==我们还将了解可以引用这些类型以形成新结构的位置。==

我们将从回顾编写JavaScript或TypeScript代码时可能遇到的最基本和常见的类型开始。这些类型将成为更复杂类型的核心构建模块。



## 原始类型: `string`,`number`和  `boolean`

JavaScript有三种非常常用的原始类型：字符串（string）、数字（number）和布尔值（boolean）。在TypeScript中，每种类型都有对应的类型。正如你所预期的，如果你对这些类型的值使用JavaScript的typeof运算符，你将看到相同的名称：

- string代表字符串值，比如"Hello, world"
- number代表数字，比如42。JavaScript没有针对整数的特殊运行时值，所以没有等价于int或float的概念——一切都简单地是number类型。
- boolean代表两个值true和false。

> ==类型名称String、Number和Boolean（首字母大写）是合法的，但指的是一些特殊的内置类型，在你的代码中很少会出现。因此，应始终使用string、number和boolean作为类型。==



## Arrays

要指定类似于[1, 2, 3]的数组的类型，可以使用==number[]语法；这个语法对于任何类型都适用（比如string[]表示字符串数组，等等）。你可能也会看到写作`Array<number>`，意思是一样的==。在我们学习泛型时，我们将更多地了解`T<U>`语法。

==请注意，[number]是一种不同的情况；请参考元组（Tuples）一节。==



## any

==TypeScript还有一种特殊类型any，当你不希望特定值引发类型检查错误时，可以使用它。==

当一个值的类型是any时，你可以访问它的任何属性（这些属性的类型也将是any），像调用函数一样调用它，将它赋值给（或从）任何类型的值，或者几乎任何其他在语法上合法的操作。

```ts
let obj: any = { x: 0 };
// 下面的代码行都不会引发编译错误。
// 使用any类型将禁用所有进一步的类型检查，并且假定你比TypeScript更了解环境。
obj.foo();
obj();
obj.bar = 100;
obj = "hello";
const n: number = obj;
```

any类型在你不想写出一个长类型来让TypeScript相信某行代码是正确的时候非常有用。



### noImplicitAny

==当你没有指定类型，并且TypeScript无法从上下文中推断出类型时，编译器通常会默认为any类型。==

==然而，通常你应该避免使用any类型，因为any类型不会进行类型检查。使用编译器标志noImplicitAny将任何隐式的any类型标记为错误。==



## 变量的类型注解

==当使用`const`、`var`或`let`声明变量时，你可以选择性地添加类型注解来明确指定变量的类型：==

```typescript
let myName: string = "Alice";
```

TypeScript不使用"types on the left"风格的声明，例如`int x = 0;`。==类型注解总是在被注解的内容之后。==

然而，==在大多数情况下，这并不是必需的。在可能的情况下，TypeScript会根据代码的初始化器来自动推断变量的类型：==

```typescript
// 不需要类型注解 -- 'myName' 被推断为类型 'string'
let myName = "Alice";
```

大部分情况下，你不需要显式地学习推断规则。如果你刚开始使用 TypeScript，尝试使用比你想象中更少的类型注解 - 你可能会惊讶地发现，在 TypeScript 完全理解代码时，你所需要的类型注解非常少。



## 函数

函数是在 JavaScript 中传递数据的主要方式。TypeScript 允许你指定函数的输入和输出值的类型。



### 参数类型注解

当你声明一个函数时，你==可以在每个参数后面添加类型注解，声明函数接受的参数类型。参数类型注解在参数名称之后：==

```typescript
// 参数类型注解
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```

当参数具有类型注解时，传递给该函数的参数将被检查：

```typescript
// 如果执行，将会导致运行时错误！
greet(42);
```

输出：
```
Argument of type 'number' is not assignable to parameter of type 'string'.
```

==即使你的参数没有类型注解，TypeScript 仍然会检查你传递的参数数量是否正确。==



### 返回类型注解

==你还可以添加返回类型注解。返回类型注解出现在参数列表之后：==

```typescript
function getFavoriteNumber(): number {
  return 26;
}
```

==与变量类型注解类似，通常情况下你不需要返回类型注解，因为 TypeScript 会根据函数的返回语句推断函数的返回类型==。上面示例中的类型注解并不会改变任何东西。==有些代码库会显式地指定返回类型以便进行文档记录、防止意外更改或个人偏好。==



### 匿名函数

==匿名函数与函数声明有些不同。当函数出现在 TypeScript 能够确定如何调用该函数的位置时，该函数的参数会自动获得类型。==

以下是一个示例：

```typescript
const names = ["Alice", "Bob", "Eve"];

// 函数的上下文类型推断 - 参数 s 推断为 string 类型
names.forEach(function (s) {
  console.log(s.toUpperCase());
});

// 上下文类型推断也适用于箭头函数
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

即使参数 s 没有类型注解，TypeScript 也会根据 forEach 函数的类型以及数组的推断类型来确定 s 的类型。

==这个过程被称为上下文类型推断，因为函数出现的上下文会影响它应该具有的类型。==

类似于推断规则，你不需要显式地学习这是如何发生的，但了解它发生的情况可以帮助你注意到不需要类型注解的情况。稍后，我们将看到更多关于值出现的上下文如何影响其类型的示例。

==函数的调用上下文是指函数在被调用时所处的环境和状态，包括函数的运行环境、函数的参数值、返回值以及与函数调用相关的其他信息。==

==全局执行上下文 就是全局环境 就是全局作用域
函数执行上下文 就是函数运行环境 就是函数作用域==



## 对象

除了基本类型之外，你会遇到最常见的类型是对象类型（Object Types）。对象类型指的是具有属性（properties）的任何 JavaScript 值，几乎所有的值都属于对象类型！==要定义一个对象类型，我们只需列出其属性及其类型。==

例如，下面是一个接受类似坐标点的对象的函数：

```javascript
// 参数的类型注解是一个对象类型
function printCoord(pt: { x: number; y: number }) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
printCoord({ x: 3, y: 7 });
```

==在这里，我们使用一个具有两个属性（x 和 y）的类型注解来标注参数，它们的类型都是数字（number）。你可以使用逗号（,）或分号（;）来分隔属性，最后一个分隔符是可选的。==

==每个属性的类型部分也是可选的。如果你不指定类型，它将被默认为 any。==



### 可选属性

==对象类型还可以指定其某些或全部属性为可选属性。要实现这一点，只需在属性名后添加一个问号（?）：==

```javascript
function printName(obj: { first: string; last?: string }) {
  // ...
}
// 都可以
printName({ first: "Bob" });
printName({ first: "Alice", last: "Alisson" });
```

==在 JavaScript 中，如果你访问一个不存在的属性，你将得到 undefined 而不是运行时错误。因此，当你读取可选属性时，你必须在使用它之前检查是否为 undefined。==

```javascript
function printName(obj: { first: string; last?: string }) {
  // 错误 - 如果没有提供 'obj.last'，可能会崩溃！
  console.log(obj.last.toUpperCase());
  // 'obj.last' 可能为 'undefined'。
  if (obj.last !== undefined) {
    // 可行
    console.log(obj.last.toUpperCase());
  }

  // 使用现代 JavaScript 语法的安全替代方案：
  console.log(obj.last?.toUpperCase());
}
```



## Union类型

==的类型系统允许您使用各种运算符从现有类型构建新类型==。现在我们知道如何编写一些类型，是时候开始以有趣的方式组合它们了。

### 定义Union类型

==组合类型的第一种方式是联合类型。联合类型是由两个或多个其他类型组成的类型，表示值可以是这些类型中的任意一个。我们将这些类型称为联合的成员。==

让我们编写一个可以操作字符串或数字的函数：

```typescript
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
// OK
printId(101);
// OK
printId("202");
// Error
printId({ myID: 22342 });
// Argument of type '{ myID: number; }' is not assignable to parameter of type 'string | number'.
```

联合类型中的每个成员都是一个类型，要提供与联合类型匹配的值非常容易。如果您有一个联合类型的值，如何处理它呢？

==TypeScript只有在对联合的每个成员都有效的情况下才允许操作==。例如，如果您有一个字符串 | 数字的联合类型，您不能使用仅适用于字符串的方法：

```typescript
function printId(id: number | string) {
  console.log(id.toUpperCase());
  // Property 'toUpperCase' does not exist on type 'string | number'.
  // Property 'toUpperCase' does not exist on type 'number'.
}
```

==解决方案是使用代码缩小联合类型的范围，就像在没有类型注释的JavaScript中一样。当TypeScript根据代码的结构推断出值的更具体类型时，就会发生缩小。==

例如，TypeScript知道只有字符串值才会具有typeof值为"string"：

```typescript
function printId(id: number | string) {
  if (typeof id === "string") {
    // 在这个分支中，id的类型为'string'
    console.log(id.toUpperCase());
  } else {
    // 在这里，id的类型为'number'
    console.log(id);
  }
}
```

另一个示例是使用Array.isArray这样的函数：

```typescript
function welcomePeople(x: string[] | string) {
  if (Array.isArray(x)) {
    // 在这里：'x'是'string[]'类型
    console.log("Hello, " + x.join(" and "));
  } else {
    // 在这里：'x'是'string'类型
    console.log("Welcome lone traveler " + x);
  }
}
```

请注意，在else分支中，我们不需要做任何特殊处理 - 如果x不是string[]，那么它必须是一个字符串。

有时候，联合中的所有成员都具有共同的特性。例如，数组和字符串都有一个slice方法。==如果联合的每个成员都有一个共同的属性，您可以在不缩小的情况下使用该属性：==

```typescript
// 返回类型被推断为number[] | string
function getFirstThree(x: number[] | string) {
  return x.slice(0, 3);
}
```

> 可能会让人困惑的是，类型的联合似乎具有这些类型属性的交集。这不是偶然的 - 联合这个名称源自类型理论。联合类型number | string由每种类型的值的联合组成。请注意，对于给定的两个具有相应事实的集合，只有这些事实的交集适用于集合本身的联合。例如，如果我们有一个戴帽子的高个子人的房间，还有一个戴帽子的讲西班牙语的人的房间，在合并这些房间之后，我们只知道每个人都必须戴着帽子。



## 类型别名

我们可以通过直接在类型注解中编写对象类型和联合类型来使用它们。这样做很方便，但通常我们希望多次使用同一类型，并通过一个名称来引用它。

==类型别名正是这样的一个名称，用于表示任何类型==。类型别名的语法如下：

```typescript
type Point = {
  x: number;
  y: number;
};
 
// 与之前的例子完全相同
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

==你实际上可以使用类型别名给任何类型命名，不仅限于对象类型==。例如，类型别名可以命名一个联合类型：

```typescript
type ID = number | string;
```

==请注意，类型别名只是别名 - 不能使用类型别名创建不同/独立的相同类型的“版本”。当你使用别名时，它就好像你编写了别名所表示的类型一样==。换句话说，下面的代码可能看起来是非法的，但根据 TypeScript 的规则是允许的，因为这两个类型都是对同一类型的别名：

```typescript
type UserInputSanitizedString = string;
 
function sanitizeInput(str: string): UserInputSanitizedString {
  return sanitize(str);
}
 
// 创建一个经过处理的输入
let userInput = sanitizeInput(getInput());
 
// 仍然可以重新赋值为字符串
userInput = "new input";
```



## 接口（Interfaces）

==接口声明是另一种给对象类型命名的方式：==

```typescript
interface Point {
  x: number;
  y: number;
}
 
function printCoord(pt: Point) {
  console.log("The coordinate's x value is " + pt.x);
  console.log("The coordinate's y value is " + pt.y);
}
 
printCoord({ x: 100, y: 100 });
```

就像我们在上面使用类型别名时一样，这个例子的工作方式就好像我们使用了一个匿名对象类型一样。TypeScript 只关注我们传递给 `printCoord` 的值的结构 - 它只关心它是否具有期望的属性。TypeScript 只关注类型的结构和能力，这就是为什么我们将 TypeScript 称为一种结构化类型系统。

==类型别名和接口非常相似，在许多情况下你可以自由选择使用它们。几乎所有接口的特性在类型中都是可用的，其中一个关键区别是类型不能被重新打开以添加新的属性，而接口则始终是可扩展的。==



### 扩展属性

通过接口扩展属性

```ts
interface Animal {
  name: string;
}

interface Bear extends Animal {
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```

通过交叉类型扩展类型别名

```ts
type Animal = {
  name: string;
}

type Bear = Animal & { 
  honey: boolean;
}

const bear = getBear();
bear.name;
bear.honey;
```



为一个现有接口扩展属性

```ts
interface Window {
  title: string;
}

interface Window {
  ts: TypeScriptAPI;
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
```

接口在被创建后就不能再被二次修改

```ts
type Window = {
  title: string;
}

type Window = {
  ts: TypeScriptAPI;
}

 // Error: Duplicate identifier 'Window'.
```

+ 类型别名不能参与声明合并，但接口可以
+ 接口只能用于声明对象的形状，而不能重命名基本类型



## 类型断言

==类型断言（Type Assertions）是在 TypeScript 中用于指定值的更具体类型的一种机制。有时候，TypeScript 只能知道某个值的类型是某个通用类型（比如 HTMLElement），而你可能知道该值的具体类型（比如 HTMLCanvasElement）。在这种情况下，你可以使用类型断言来指定更具体的类型。==

通常，==类型断言使用 `as` 关键字进行表示==，例如：

```typescript
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

==你也可以使用尖括号的语法（不适用于 .tsx 文件），其等效于上述方式：==

```typescript
const myCanvas = <HTMLCanvasElement>document.getElementById("main_canvas");
```

==需要注意的是，类型断言仅在编译时起作用，不会影响代码的运行时行为。==

==类型断言只是告诉编译器某个值的类型，没有运行时的检查。如果类型断言错误，不会触发异常或生成 null 值。==

==TypeScript 只允许类型断言将类型转换为更具体或更通用的版本。这个规则避免了“不可能”的强制转换==，例如：

```typescript
const x = "hello" as number;
```

这里的类型断言将字符串类型转换为数字类型，但是它们之间没有重叠，因此编译器会给出错误提示。

==有时候，这个规则可能过于保守，会禁止一些复杂的强制转换，尽管它们是有效的。如果遇到这种情况，你可以使用两个断言，首先将表达式断言为 `any`（或后面介绍的 `unknown`），然后再将其断言为目标类型：==

```typescript
const a = (expr as any) as T;
```

这样就可以绕过编译器的限制进行类型转换。



## 字面量类型

除了一般的字符串和数字类型之外，我们可以在类型位置引用特定的字符串和数字。

一种思考方式是考虑 JavaScript 提供了不同的声明变量的方式。==`var` 和 `let` 允许更改变量中存储的内容，而 `const` 则不允许。TypeScript 在字面类型方面反映了这一点。==

```typescript
let changingString = "Hello World";
changingString = "Olá Mundo";
// 因为 `changingString` 可以表示任何可能的字符串，所以 TypeScript 在类型系统中描述它时就是这样的
// let changingString: string;
 
const constantString = "Hello World";
// 因为 `constantString` 只能表示一个可能的字符串，它具有字面类型的表示方式
// const constantString: "Hello World";
```

单独使用字面类型并没有太大的价值：

```typescript
let x: "hello" = "hello";
// OK
x = "hello";
// ...
x = "howdy";
// Type '"howdy"' is not assignable to type '"hello"'.
```

拥有只能有一个值的变量并没有太大的用处！

==但是通过将字面值组合成联合类型，您可以表达一个更有用的概念== - 例如，只接受一定集合的已知值的函数：

```typescript
function printText(s: string, alignment: "left" | "right" | "center") {
  // ...
}

printText("Hello, world", "left");
printText("G'day, mate", "centre");
// Argument of type '"centre"' is not assignable to parameter of type '"left" | "right" | "center"'.
```

数字字面类型的工作方式相同：

```typescript
function compare(a: string, b: string): -1 | 0 | 1 {
  return a === b ? 0 : a > b ? 1 : -1;
}
```

当然，您可以将它们与非字面类型结合使用：

```typescript
interface Options {
  width: number;
}

function configure(x: Options | "auto") {
  // ...
}

configure({ width: 100 });
configure("auto");
// Argument of type '"automatic"' is not assignable to parameter of type 'Options | "auto"'.
```

==还有一种字面类型：布尔字面类型。只有两种布尔字面类型，你可能猜到了，它们就是 `true` 和 `false` 类型。其实布尔类型本身只是 `true | false` 的联合类型的别名。==



### 字面量接口

==当您用对象初始化一个变量时，TypeScript会假设该对象的属性可能会在以后改变值==。例如，如果您编写了以下代码：

```typescript
const obj = { counter: 0 };
if (someCondition) {
  obj.counter = 1;
}
```

TypeScript不会认为将1赋值给之前为0的字段是一个错误。换句话说，obj.counter的类型必须是number，而不是0，因为类型用于确定读取和写入的行为。

对于字符串也是如此：

```typescript
declare function handleRequest(url: string, method: "GET" | "POST"): void;
 
const req = { url: "https://example.com", method: "GET" };
handleRequest(req.url, req.method);
```

上面的例子中，req.method被推断为string，而不是"GET"。因为在创建req和调用handleRequest之间的代码可能会被执行，这可能会将新的字符串（如"GUESS"）赋给req.method，TypeScript认为这段代码有错误。

有两种方法可以解决这个问题。

==您可以在任一位置添加类型断言来改变推断：==

```typescript
// 改变1：
const req = { url: "https://example.com", method: "GET" as "GET" };
// 改变2：
handleRequest(req.url, req.method as "GET");
```

改变1的意思是“我打算让req.method始终具有字面类型"GET"”，防止在之后将"GUESS"赋给该字段。改变2的意思是“出于其他原因，我知道req.method的值是"GET"”。

==您可以使用`as const`将整个对象转换为字面类型：==

```typescript
const req = { url: "https://example.com", method: "GET" } as const;
handleRequest(req.url, req.method);
```

==`as const`后缀在类型系统中的作用类似于const，确保所有属性都被赋予字面类型，而不是更一般的类型，如string或number。==



## null 和 undefined

==JavaScript有两个原始值用于表示缺失或未初始化的值：null 和 undefined。==

==TypeScript也有同名的两个对应类型。这些类型的行为取决于是否打开了 strictNullChecks 选项。==

strictNullChecks 关闭时
==在关闭 strictNullChecks 选项时，仍然可以正常访问可能为 null 或 undefined 的值，并且可以将 null 和 undefined 赋给任何类型的属性==。这类似于没有 null 检查的语言（例如 C#、Java）的行为。==不检查这些值往往是错误的主要来源；我们始终建议在代码库中实践时，将 strictNullChecks 打开。==

strictNullChecks 开启时
==在打开 strictNullChecks 选项时，当一个值为 null 或 undefined 时，您需要在使用该值的方法或属性之前进行检查。就像在使用可选属性之前检查 undefined 一样，我们可以使用 narrowing 来检查可能为 null 的值：==

```typescript
function doSomething(x: string | null) {
  if (x === null) {
    // 什么都不做
  } else {
    console.log("Hello, " + x.toUpperCase());
  }
}
```



### 非空断言操作符（后缀`!`）

==TypeScript还有一种特殊的语法，用于从类型中移除 null 和 undefined，而无需进行任何显式的检查。在任何表达式之后写上`!`相当于对该值进行类型断言，表明该值不会为 null 或 undefined：==

```typescript
function liveDangerously(x?: number | null) {
  // 没有错误
  console.log(x!.toFixed());
}
```

与其他类型断言一样，这不会改变代码的运行时行为，因此只有在确保值不会为 null 或 undefined 时才使用`!`非空断言。



## Enums

==枚举是由TypeScript添加到JavaScript中的一项功能，它允许描述一个值，该值可以是一组可能的命名常量之一。与大多数TypeScript功能不同，枚举不是JavaScript的类型级别添加，而是添加到语言和运行时中的一项功能。因此，这是一个你应该知道存在的功能，但除非你确定，否则可能不要使用==。你可以在枚举参考页面上阅读更多关于枚举的信息。



## 不常见的原始类型

值得一提的是，在类型系统中，JavaScript还有其他一些表示原始类型的方式。虽然我们在这里不会详细讨论它们。

### bigint

从ES2020开始，JavaScript中有一种用于表示非常大整数的原始类型，即BigInt：

```typescript
// 通过BigInt函数创建一个bigint
const oneHundred: bigint = BigInt(100);

// 通过字面量语法创建一个BigInt
const anotherHundred: bigint = 100n;
```

你可以在TypeScript 3.2发布说明中了解更多关于BigInt的信息。



### symbol

JavaScript中有一种用于通过函数Symbol()创建全局唯一引用的原始类型：

```typescript
const firstName = Symbol("name");
const secondName = Symbol("name");

if (firstName === secondName) {
  // 这个比较似乎是无意的，因为'typeof firstName'和'typeof secondName'的类型没有重叠。
  // 这个条件永远不会成立
}
```

你可以在Symbols参考页面上了解更多关于它们的信息。

