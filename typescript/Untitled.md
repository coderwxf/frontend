JavaScript是一种为了快速开发具有交互性的网站而被设计的语言，像如今已经发展成为了一种用于编写数百万行应用程序的成熟工具

然而  JavaScript`不够完善和严谨的语法特性` 和 `日益增长的应用复杂度` 使得JavaScript开发在规模化管理方面变得困难

1. **JavaScript是动态类型语言**。变量的实际类型只有在运行时才会被确定

   这就意味着JavaScript必须通过运行应用，才可以发现对应错误，这不仅不方便调试，也更容易出现隐藏bug

   

2. **JavaScript拥有一些特殊的语法规则**

   

   + JavaScript可以访问一个对象上没有声明的属性

   ```ts
   const obj = { width: 10, height: 15 };
   const area = obj.width * obj.heigth; // 10 * undefined => NaN
   ```

   

   + JavaScript 具有一些特殊的隐式转换规则

   ```ts
   console.log("" == 0) // => true
   console.log(1 < x < 3) // => true
   ```

   

这类错误基本都可以被归类为**类型错误**。这往往是由于简单的拼写错误、对库API的不理解、对运行时行为做出错误假设或其他错误引起



因此，TypeScript的目标是成为**JavaScript程序的静态类型检查器**，在实际运行时依旧需要将TypeScript编译为原生JavaScript后才可以被浏览器正确解析

一旦TypeScript的编译器完成对代码的检查，它会在编译时进行类型擦除。这意味着一旦编译完成，生成的纯JS代码就没有类型信息。

这也意味着，TypeScript永远不会基于它推断的类型改变程序的运行时行为。

+ 如果将代码从JavaScript移动到TypeScript中，无论在编译过程中是否出现类型错误，类型系统本身对程序运行的方式不会产生任何的影响，这意味着代码可以以原本的方式正常运行
+ TypeScript在运行时就是去除类型的JavaScript，因此TypeScript可以很方便的和任何的JavaScript库一起配合使用



### 类型推断

TS提供了一种类型系统，无需在代码中添加额外的字符来明确类型

+ 绝大多数情况下，TS类型系统可以根据当前上下文推导出变量类型
+ Visual Studio Code可以根据类型系统进行合理的代码提示

```ts
let helloWorld = "Hello World"; // type helloWorld -> string
```



您可以在JavaScript中使用各种设计模式。然而，某些设计模式使得自动推断类型变得困难（例如使用动态编程的模式）

动态编程 --- 可以根据运行时条件动态地生成具体的类型，不同的运行时条件可能导致不同的类型，例如条件类型

由于条件类型是根据代码上下文推导的，所以它们可能会导致自动推导失败。

为了涵盖这些情况，TypeScript支持JavaScript语言的扩展，为您提供了告诉TypeScript类型应该是什么的位置。

```ts
interface User {
  name: string;
  id: number;
}

const user: User = {
  name: "Hayes",
  id: 0,
};
```

如果您提供了一个与您提供的接口不匹配的对象，TypeScript会发出警告：



JavaScript中已经有一小组基本类型可用：布尔值、大整数、空值、数字、字符串、符号和未定义的值，您可以在接口中使用它们。TypeScript通过添加一些其他类型扩展了此列表，例如any（允许任何内容）、unknown（确保使用此类型的人声明类型是什么）、never（不可能发生此类型）和void（返回undefined或没有返回值的函数）



您会发现有两种语法用于构建类型：接口和类型。应优先使用接口。当需要特定功能时，请使用类型。



### 组合类型

使用TypeScript，您可以通过组合简单类型来创建复杂类型。有两种流行的方法：使用联合类型和泛型。

#### 联合类型

使用联合类型，您可以声明一个类型可以是多个类型之一

```ts
type MyBool = true | false;
```

要了解变量的类型，请使用typeof --- 通过typeof进行类型缩小

| Type      | Predicate                          |
| :-------- | :--------------------------------- |
| string    | `typeof s === "string"`            |
| number    | `typeof n === "number"`            |
| boolean   | `typeof b === "boolean"`           |
| undefined | `typeof undefined === "undefined"` |
| function  | `typeof f === "function"`          |
| array     | `Array.isArray(a)`                 |



#### 泛型

泛型为类型提供了变量 --- 类型变量

```ts
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}
 
// declare 关键字用于声明变量或函数的类型而不需要实际实现它们
// 也就是说 declare 用于声明存在backpack 类型为Backpack<string>
// 但是在哪里定义的，在哪里使用的 无需关心
declare const backpack: Backpack<string>;
 
const object = backpack.get(); // string
 
backpack.add(23); // error
```



## 结构化类型系统

TS是结构类型系统，核心原则之一是类型检查关注的是值所具有的形状。这有时被称为“鸭子类型”或“结构类型”。

在结构类型系统中，如果两个对象具有相同的形状，它们被认为是相同类型的

换句话说，它们的成员变量和成员函数的名称、类型和数量相同。这种类型系统与传统的基于类的类型系统不同，后者关注的是对象的类别（即对象的基类或派生类）

```ts
interface User {
  name: string;
  id: number;
}
 
class UserAccount {
  name: string;
  id: number;
 
  constructor(name: string, id: number) {
    this.name = name;
    this.id = id;
  }
}
 
const user: User = new UserAccount("Murphy", 1);
```

```ts
interface Point {
  x: number;
  y: number;
}
 
function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}
 
// logs "12, 26"
const point = { x: 12, y: 26 };
logPoint(point);
```

形状匹配只需要对象的一部分字段匹配即可。这意味着，如果一个对象具有与另一个对象相同的属性名称和类型，即使它们具有不同的属性名和类型，它们也可以被视为具有相同的类型。这种类型匹配方式被称为结构类型匹配，它是 TypeScript 的一个核心特性。

如果我们将一个对象A赋值给另一个对象B，那么A必须具有B中所有属性，并且它们的类型需要匹配。但是，A也可以包含其他额外的属性，而这些属性不会影响它们之间的匹配关系



TypeScript的类型系统提供了许多相同的好处，例如更好的代码补全、更早的错误检测和程序各部分之间更清晰的通信(结构约束)

```shell
 npm install -g typescript
 
 # ts文件以ts作为文件后缀名
 # tsc - typescript compiler -- TypeScript 编译器
 tsc greeter.ts
```



## 类型注释

在 TypeScript 中，类型注释是记录函数或变量预期合约的轻量级方式