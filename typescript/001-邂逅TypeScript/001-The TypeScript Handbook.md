虽然JavaScript程序的大小、范围和复杂性呈指数级增长，但JavaScript语言表达不同代码单元之间关系的能力并未相应增长。

这也就是意味着使用着可以根据自己的喜欢组织代码，在多人维护的大项目中这可能导致代码结构混乱、可维护性差以及难以理解和调试的问题。因此，在处理复杂的JavaScript程序时，良好的代码组织和设计原则仍然非常重要，以确保代码的可读性、可维护性和可扩展性

再加上JavaScript的相当奇特的运行时语义，这种语言和程序复杂性之间的不匹配使得JavaScript开发在大规模情况下成为一项难以管理的任务。

程序员编写的最常见的错误类型可以描述为类型错误：在期望使用一种类型的值时使用了另一种类型的值。

这可能是由于简单的拼写错误、不理解库的API表面、对运行时行为做出错误的假设或其他错误导致的。

TypeScript的目标是成为JavaScript程序的静态类型检查器，换句话说，它是一个在代码运行之前运行的工具（静态），并确保程序的类型正确（类型检查）。

---

TypeScript是JavaScript的一种“变体”或“扩展”

JavaScript最初设计用于快速使用的语言(在网页中添加简单的交互)，然后发展成为一种可以写百万行代码的完整工具

每种语言都有其特殊之处，JavaScript由于其谦逊的起源而具有许多独特之处和令人惊讶的地方。以下是一些例子：

1. JavaScript的等号运算符（==）强制转换操作数

   ```ts
   console.log("" == 0) // true
   console.log(1 < x < 3) // true --- 1 < x => true | false --- 0|1 < 3 --- true
   ```

2. JavaScript还允许访问不存在的属性

   ```ts
   const obj = { width: 10, height: 15 };
   const area = obj.width * obj.heigth; // 10 * undefined --- NaN
   ```

大多数编程语言在发生此类错误时都会抛出错误，有些甚至会在编译过程中抛出错误——在任何代码运行之前

在不运行代码的情况下检测错误被称为静态检查。

根据操作的值的类型来确定何为错误何不为错误被称为静态类型检查

TypeScript在执行之前对程序进行错误检查，它是基于值的类型进行检查的，因此它是一个静态类型检查器

---

TypeScript（TS）是JavaScript（JS）的超集，因此JS语法在TS中是合法的

TypeScript不会将任何JavaScript代码视为错误，因为它的语法是兼容的。这意味着你可以将任何有效的JavaScript代码放入TypeScript文件中，而无需担心代码的具体编写方式



然而，TypeScript是一种带有类型规则的超集，它添加了关于不同类型的值如何使用的规则

之前关于obj.heigth的错误并不是语法错误：它是一种以不正确的方式使用某种值（类型）的错误。



TypeScript 的类型检查器旨在允许正确的程序通过，同时尽可能捕捉到常见的错误

可以配置一些设置来调整TypeScript 对代码的严格检查程度



运行时行为

TypeScript 也是一种保留 JavaScript 运行时行为的编程语言

作为原则，TypeScript 不会更改 JavaScript 代码的运行时行为。

如果你将代码从 JavaScript 移动到 TypeScript，它保证以相同的方式运行，即使 TypeScript 认为代码存在类型错误



类型擦除

一旦 TypeScript 的编译器完成对代码的检查，它会擦除类型以生成最终的“编译”代码。

这意味着一旦你的代码被编译，生成的普通 JS 代码就没有类型信息。

这也意味着，TypeScript 不会根据推断的类型更改程序的行为

总之，虽然在编译过程中可能会看到类型错误，但类型系统本身对程序运行时的工作方式没有影响。



由于 TypeScript 是 JavaScript 的超集，它可以在 TypeScript 项目中直接使用 JavaScript 库，而不需要进行任何修改或转换。因此没有特定于 TypeScript 的框架或库



TypeScript是JavaScript的运行时加上一个编译时类型检查器

----

TypeScript提供了JavaScript的所有功能，并在其之上提供了额外的层次：TypeScript的类型系统。



通过推断进行类型推断
TypeScript了解JavaScript语言，并在许多情况下为你生成类型

```ts
let helloWorld = "Hello World";
```

通过理解JavaScript的工作原理，TypeScript可以构建一个接受JavaScript代码但具有类型的类型系统。这提供了一种类型系统，无需在代码中添加额外的字符来显式声明类型

这可以提供编辑器的自动提示功能



可以使用接口声明显式描述该对象的形状

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



TypeScript通过一些其他类型扩展了这个列表，例如any（允许任何类型）、unknown（确保使用该类型的人声明类型是什么）、never（不可能发生该类型）和void（返回undefined或没有返回值的函数）。

你会发现有两种语法来构建类型：接口（Interfaces）和类型（Types）。在需要特定功能时，应优先使用接口。当你需要特定功能时，使用类型。



类型组合

联合类型
使用联合类型，你可以声明一个类型可以是多种类型之一。例如，你可以将布尔类型描述为true或false之一：



泛型（Generics）为类型提供了变量



`declare`关键字通常在类型声明文件（.d.ts）中使用，用于声明全局变量或外部模块的类型信息, 即告诉编译器某个变量、属性或方法已经存在或将在运行时提供



结构化类型系统

TypeScript的核心原则之一是类型检查侧重于值的形状。这有时被称为“鸭子类型”或“结构类型”。

在结构类型系统中，如果两个对象具有相同的形状，它们被认为是相同类型的。

形状匹配只需要对象的一部分字段匹配即可。如果对象或类具有所有所需的属性，TypeScript将表示它们匹配，而不考虑实现细节。



```shell
npm install -g typescript

# tsc，即TypeScript编译器
tsc greeter.ts # 同目录下 生成 greeter.js

# 警告提升为报错 有类型错误 就生成js文件失败
tsc --noEmitOnError hello.ts
```



