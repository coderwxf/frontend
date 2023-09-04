基本类型：

JavaScript有三种非常常用的基本类型：字符串（string）、数字（number）和布尔值（boolean）。TypeScript中每种类型都有对应的类型

JavaScript没有整数的特殊运行时值，所以没有int或float的等价物 - 一切都是number

类型名称String、Number和Boolean（首字母大写）是合法的，但是它们指的是一些特殊的内置类型，很少会在你的代码中出现。始终使用string、number或boolean来表示类型。



数组
要指定类似于[1, 2, 3]的数组的类型，可以使用number[]的语法；这个语法适用于任何类型（例如string[]表示字符串数组，依此类推）。你也可能会看到这样的写法Array<number>，它的意思是一样的



any
TypeScript还有一个特殊的类型any，当你不想让特定值导致类型检查错误时，可以使用它。

当一个值的类型是any时，你可以访问它的任何属性（这些属性本身将是any类型），像调用函数一样调用它，将它赋值给（或从）任何类型的值，或者几乎任何其他在语法上合法的操作。

使用`any`会禁用所有进一步的类型检查，并且假设您比TypeScript更了解环境。

当您没有指定类型，并且TypeScript无法从上下文中推断出类型时，编译器通常会默认为an

通常情况下您应该避免使用any，因为any类型不会进行类型检查。可以使用编译器标志noImplicitAny将任何隐式的any标记为错误。





在使用const、var或let声明变量时，您可以选择性地添加类型注释来明确指定变量的类型：

然而，在大多数情况下，这是不需要的。TypeScript会尽可能地自动推断代码中的类型



函数
函数是在JavaScript中传递数据的主要手段。TypeScript允许您指定函数的输入和输出值的类型。

参数类型注释
当您声明一个函数时，您可以在每个参数后面添加类型注释，以声明函数接受的参数的类型。参数类型注释放在参数名称之后：

```ts
function greet(name: string) {
  console.log("Hello, " + name.toUpperCase() + "!!");
}
```



即使您在参数上没有类型注释，TypeScript仍然会检查您是否传递了正确数量的参数。

返回类型注释
您还可以添加返回类型注释。返回类型注释出现在参数列表之后：

```ts
function getFavoriteNumber(): number {
  return 26;
}
```



与变量类型注释类似，通常您不需要返回类型注释，因为TypeScript会根据函数的返回语句推断函数的返回类型

一些代码库会显式指定返回类型以用于文档目的，防止意外更改，或仅出于个人喜好。



匿名函数
匿名函数与函数声明有些不同。当函数出现在TypeScript可以确定如何调用它的位置时，该函数的参数将自动获得类型

```ts
const names = ["Alice", "Bob", "Eve"];
 
// Contextual typing for function - parameter s inferred to have type string
names.forEach(function (s) {
  console.log(s.toUpperCase());
});
 
// Contextual typing also applies to arrow functions
names.forEach((s) => {
  console.log(s.toUpperCase());
});
```

尽管参数`s`没有类型注释，TypeScript使用`forEach`函数的类型以及数组的推断类型来确定`s`的类型。

这个过程被称为上下文类型推断，因为函数发生的上下文提供了它应该具有的类型。



对象类型
除了基本类型之外，您将遇到的最常见的类型是对象类型。这指的是具有属性的任何JavaScript值

为了定义一个对象类型，我们只需列出它的属性及其类型。

```ts
const foo = { x: number; y: number }
```

在这里，我们使用两个属性 - x 和 y - 对参数进行了类型注解，它们都是数字类型。你可以使用逗号或者分号来分隔属性，最后一个分隔符是可选的。

每个属性的类型部分也是可选的。如果你不指定类型，将默认为 any 类型。

的一些或全部属性是可选的。要实现这一点，请在属性名后面加上 ?。

```ts
const obj = { first: string; last?: string }
```

在 JavaScript 中，如果访问一个不存在的属性，你将得到 undefined 值而不是运行时错误。因此，当你读取可选属性时，在使用之前必须先检查是否为 undefined。





联合类型
TypeScript 的类型系统允许你使用各种运算符从现有类型构建新类型

联合类型是由两个或多个其他类型组成的类型，表示值可以是其中任意一个类型。我们将这些类型称为联合的成员。

```ts
const id = number | string
```



TypeScript 只允许操作对于联合的每个成员都有效的操作

解决方法是使用代码缩小联合类型，就像在没有类型注解的 JavaScript 中一样。

当 TypeScript 可以根据代码的结构推断出一个更具体的值类型时，就会发生缩小。



类型别名

如果我们希望多次使用相同的类型，并通过一个名称引用它。就可以使用类型别名

```ts
type NumberOrString = number | string;
```

请注意，别名只是别名 - 你不能使用类型别名来创建不同/不同版本的相同类型



接口和别名都可以用来声明对象的结构

类型别名和接口非常相似，而且在许多情况下，你可以自由选择它们之间的区别

几乎所有接口的特性都可以在类型中使用，关键的区别在于类型不能重新打开以添加新属性，而接口始终是可扩展的

![image-20230828231329210](https://s2.loli.net/2023/08/28/4DhGKQItFOfbX3m.png) 

![image-20230828231522672](https://s2.loli.net/2023/08/28/CG1Uljzcevsum8i.png) 

接口只能用于声明对象的形状，而不能重新命名基本类型。



类型断言

有时候你会有关于一个值类型的信息，但是TypeScript无法自动推断出来。

例如，如果你使用document.getElementById，TypeScript只知道它会返回一个HTMLElement类型的对象，但是你可能知道你的页面上总是会有一个特定ID的HTMLCanvasElement。

在这种情况下，你可以使用类型断言来指定一个更具体的类型：

```ts
const myCanvas = document.getElementById("main_canvas") as HTMLCanvasElement;
```

由于类型断言在编译时被移除，所以与类型断言相关的运行时检查是不存在的。如果类型断言是错误的，不会生成异常或空值



TypeScript只允许将类型断言转换为更具体或更不具体的类型版本。这个规则防止了"不可能"的强制类型转换，

有时这个规则可能过于保守，会禁止一些可能是有效的更复杂的强制类型转换。

如果出现这种情况，你可以使用两个断言，先将类型断言为any（或者后面我们会介绍的unknown），然后再将其转换为所需的类型：

```ts
const a = (expr as any) as T;
```



字面量类型

除了通用的string和number类型外，我们还可以在类型位置引用特定的字符串和数字。

```ts
let changingString = "Hello World"; // string
const constantString = "Hello World"; // Hello World
```



拥有只能具有一个值的变量并没有太大用处！但是通过将字面量组合成联合类型，你可以表达一个更有用的概念

实际上，boolean类型本身只是true | false的联合类型的别名。



可以使用`as const`将整个对象转换为类型字面量

```ts
const req = { url: "https://example.com", method: "GET" } as const;
// { url: "https://example.com", method: "GET" }
```

`as const`后缀的作用类似于`const`，但是针对类型系统，确保所有属性都被分配为字面量类型，而不是更一般化的类型，如string或number。



null和undefined

JavaScript有两个原始值用于表示缺失或未初始化的值：null和undefined。

TypeScript有两个相应的同名类型。这些类型的行为取决于是否打开了strictNullChecks选项。

strictNullChecks关闭时，可能为null或undefined的值仍然可以正常访问，并且可以将null和undefined赋值给任何类型的属性

strictNullChecks打开时，当一个值为null或undefined时，你需要在使用该值的方法或属性之前进行检查，并且不可以将null和undefined赋值给任何类型的属性

我们总是建议人们在代码库中实际可行的情况下打开strictNullChecks。



非空断言操作符（后缀!）

TypeScript还有一种特殊的语法，可以在不进行任何显式检查的情况下从类型中移除null和undefined。

在任何表达式之后写上!实际上是一种类型断言，表示该值不是null或undefined：

就像其他类型断言一样，这不会改变代码的运行时行为，因此只有在你确定值不能为null或undefined时才使用

```ts
console.log(x!.toFixed());
```



枚举（Enums）

枚举是由TypeScript添加到JavaScript的一项功能，它允许描述一个值，该值可以是一组可能的命名常量之一。

与大多数TypeScript功能不同，这不是对JavaScript的类型级别的添加，而是对语言和运行时的添加



较少常见的原始类型

JavaScript中有一个用于表示非常大整数的原始类型，即BigInt

number 和bigint是两种完全独立的类型 不可以相互赋值

```ts
// Creating a bigint via the BigInt function
// bigint --- 类型都是小写字母
const oneHundred: bigint = BigInt(100);
 
// Creating a BigInt via the literal syntax
const anotherHundred: bigint = 100n;
```



bigint和number在存储结构上是完全不同的，因此它们之间不能进行直接的四则运算

```ts
console.log(3.141592 * 10000n); // error
console.log(3145 * 10n); // error
console.log(BigInt(3145) * 10n); // okay!
```



用typeof操作符时，bigint会产生一个新的字符串："bigint"。因此，TypeScript会正确使用typeof进行类型缩小

```ts
console.log(100n) // bigint
```



