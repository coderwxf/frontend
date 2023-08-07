TypeScript与JavaScript之间存在一种不同寻常的关系。==TypeScript提供了所有JavaScript的功能，并在此基础上添加了一层：TypeScript的类型系统==。

例如，JavaScript提供了诸如字符串和数字之类的语言原语，但它不会检查您是否一致地分配了这些类型。而TypeScript会进行检查。

==JavaScript 的原语（primitive）是指该语言中的基本数据类型

这意味着您现有的JavaScript代码也是有效的TypeScript代码。TypeScript的主要优点是它可以突出显示代码中的意外行为，降低出现错误的几率。

本教程提供了对TypeScript的简要概述，重点介绍其类型系统。



通过推断类型

==TypeScript了解JavaScript语言，并在许多情况下为您生成类型==。例如，在创建一个变量并将其赋值给特定值时，TypeScript将使用该值作为其类型。

```typescript
let helloWorld = "Hello World";

// let helloWorld: string
```

==通过理解JavaScript的工作原理，TypeScript可以构建一个接受JavaScript代码但具有类型的类型系统。这提供了一种类型系统，而无需在代码中添加额外的字符来明确类型==。这就是TypeScript在上面的示例中知道helloWorld是一个字符串的方式。

==您可能在Visual Studio Code中编写过JavaScript，并且使用了编辑器的自动补全功能。Visual Studio Code在内部使用TypeScript，使您更容易使用JavaScript编码==。



定义类型
==在JavaScript中，您可以使用各种设计模式。然而，某些设计模式会使类型无法自动推断（例如，使用动态编程的模式）。为了涵盖这些情况，TypeScript支持了JavaScript语言的扩展，该扩展提供了让您告诉TypeScript类型应该是什么的位置。==

例如，要创建一个具有推断类型的对象，其中包括name: string和id: number，您可以编写：

```typescript
const user = {
  name: "Hayes",
  id: 0,
};
```


==您可以使用接口声明来明确描述该对象的形状：==

```typescript
interface User {
  name: string;
  id: number;
}
```


然后，==您可以使用变量声明后的 : TypeName 语法，声明一个JavaScript对象符合您的新接口的形状==：

```typescript
const user: User = {
  name: "Hayes",
  id: 0,
};
```


如果您提供的对象与您提供的接口不匹配，TypeScript会发出==警告：==

```typescript
interface User {
  name: string;
  id: number;
}
 
const user: User = {
  username: "Hayes",
};
```


由于JavaScript支持类和面向对象编程，TypeScript也支持。您可以在类中使用接口声明：

```typescript
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


您可以使用接口对函数的参数和返回值进行注释：

```typescript
function deleteUser(user: User) {
  // ...
}
 
function getAdminUser(): User {
  //...
}
```


在JavaScript中已经有一小组原始类型可用：boolean、bigint、null、number、string、symbol和undefined，您可以在接口中使用它们。TypeScript通过几个额外的类型扩展了这个列表，例如any（允许任何类型）、unknown（确保使用此类型的人声明了类型）和never（不可能发生的类型）以及void（返回undefined或没有返回值的函数）。

==您会发现有两种语法用于构建类型：接口（Interfaces）和类型（Types）。在需要特定功能时，请使用类型，但优先使用接口。==



## 组合类型

==在TypeScript中，您可以通过组合简单类型来创建复杂类型。有两种常用的方式：使用联合类型（unions）和泛型（generics）。==

### 联合类型（Unions）

==使用联合类型，您可以声明一个类型可以是多种类型之一==。例如，您可以将布尔类型描述为true或false：

```typescript
type MyBool = true | false;
```
注意：如果您在上面悬停在MyBool上方，您会发现它被归类为boolean。这是结构类型系统的属性。稍后会详细介绍。

==联合类型的一个常见用例是描述一个值允许的字符串或数字字面量的集合：==

```typescript
type WindowStates = "open" | "closed" | "minimized";
type LockStates = "locked" | "unlocked";
type PositiveOddNumbersUnderTen = 1 | 3 | 5 | 7 | 9;
```
联合类型还提供了处理不同类型的方式。例如，您可能有一个接受数组或字符串的函数：

```typescript
function getLength(obj: string | string[]) {
  return obj.length;
}
```
==要了解变量的类型，可以使用typeof：==

类型	谓词
`string	typeof s === "string"
number	typeof n === "number"
boolean	typeof b === "boolean"
undefined	typeof undefined === "undefined"
function	typeof f === "function"
array	Array.isArray(a)`

例如，您可以使一个函数根据传入的是字符串还是数组返回不同的值：

```typescript
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
  }
  return obj;
}
```


### 泛型（Generics）

==泛型为类型提供了变量。一个常见的例子是数组。不使用泛型的数组可以包含任何类型。而使用泛型的数组可以描述数组包含的值的类型。==

```typescript
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

您可以声明自己使用泛型的类型：

```typescript
interface Backpack<Type> {
  add: (obj: Type) => void;
  get: () => Type;
}

// 这一行是一个快捷方式，告诉TypeScript有一个名为`backpack`的常量，不用担心它是从哪里来的。
declare const backpack: Backpack<string>;

// object是一个字符串，因为我们在上面将其声明为Backpack的变量部分。
const object = backpack.get();

// 由于backpack变量是一个字符串，所以不能将数字传递给add函数。
backpack.add(23);
// Argument of type 'number' is not assignable to parameter of type 'string'.
```


## 结构类型系统

==TypeScript的核心原则之一是类型检查关注的是值的形状。这有时被称为“鸭子类型”或“结构类型”。==

==在结构类型系统中，如果两个对象具有相同的形状，它们被认为是相同的类型。==

```typescript
interface Point {
  x: number;
  y: number;
}

function logPoint(p: Point) {
  console.log(`${p.x}, ${p.y}`);
}

// 输出 "12, 26"
const point = { x: 12, y: 26 };
logPoint(point);
```
point变量从未声明为Point类型。然而，TypeScript在类型检查中将point的形状与Point的形状进行比较。它们具有相同的形状，因此代码通过了检查。

==形状匹配只需要对象的字段的子集匹配即可。==

```typescript
const point3 = { x: 12, y: 26, z: 89 };
logPoint(point3); // 输出 "12, 26"

const rect = { x: 33, y: 3, width: 30, height: 80 };
logPoint(rect); // 输出 "33, 3"

const color = { hex: "#187ABF" };
logPoint(color);
// Argument of type '{ hex: string; }' is not assignable to parameter of type 'Point'.
//  Type '{ hex: string; }' is missing the following properties from type 'Point': x, y
```
类和对象遵循相同的形状匹配规则：

```typescript
class VirtualPoint {
  x: number;
  y: number;

  constructor(x: number, y: number) {
    this.x = x;
    this.y = y;
  }
}

const newVPoint = new VirtualPoint(13, 56);
logPoint(newVPoint); // 输出 "13, 56"
```
==如果对象或类具有所有必需的属性，无论实现细节如何，TypeScript都会认为它们匹配。==



## 下一步

这是对日常使用TypeScript的语法和工具的简要概述。从这里开始，您可以：

从头到尾阅读完整的手册
探索Playground中的示例

