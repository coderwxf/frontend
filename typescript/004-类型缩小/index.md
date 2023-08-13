想象我们有一个名为`padLeft`的函数。

```typescript
function padLeft(padding: number | string, input: string): string {
  throw new Error("Not implemented yet!");
}
```

如果`padding`是一个数字，它将把它作为我们要在`input`前面添加的空格数。如果`padding`是一个字符串，它应该只是将`padding`添加到`input`的前面。让我们尝试实现当`padLeft`接收一个数字作为`padding`时的逻辑。

```typescript
function padLeft(padding: number | string, input: string) {
  return " ".repeat(padding) + input;
}
```

我们得到了一个错误，错误提示是关于`padding`的。TypeScript警告我们将一个类型为`number | string`的值传递给`repeat`函数，而`repeat`函数只接受一个数字，这是正确的。换句话说，我们没有明确检查`padding`是否是一个数字，也没有处理它是字符串的情况，所以让我们来做这个处理。

```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

如果这看起来像一个不太有趣的JavaScript代码，那就是我们的目的。除了我们添加的类型注解外，这段TypeScript代码看起来像JavaScript。==**TypeScript的目标是使编写典型JavaScript代码尽可能简单，而无需过度弯曲以获得类型安全**==。

==**尽管它看起来可能不起眼，但在底层实际上有很多事情发生。就像TypeScript使用静态类型分析运行时值一样，在JavaScript的运行时控制流构造（如if/else、条件三元运算符、循环、真值检查等）上叠加了类型分析，这些都可能影响到类型。**==

在我们的if语句中，==**TypeScript看到`typeof padding === "number"`并将其理解为一种特殊形式的代码，称为类型保护**==。==**TypeScript根据程序可能采取的路径来跟踪执行，以分析给定位置上值的最具体类型。它查看这些特殊的检查（称为类型保护）和赋值，并将类型细化为比声明更具体的类型，这个过程称为类型缩小**==。在许多编辑器中，我们可以观察到这些类型在变化，我们的示例中也会这样做。

```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
                        
// (parameter) padding: number
  }
  return padding + input;
           
// (parameter) padding: string
}
```

TypeScript理解并支持几种不同的缩小类型的构造方式。



## typeof类型守卫

正如我们所见，==**JavaScript支持typeof运算符，它可以在运行时提供关于值类型的非常基本的信息。TypeScript期望它返回一组特定的字符串：**==

- "string"
- "number"
- "bigint"
- "boolean"
- "symbol"
- "undefined"
- "object"
- "function"

就像我们在padLeft中看到的那样，这个运算符经常在许多JavaScript库中出现，并且TypeScript可以理解它以缩小不同分支中的类型。

在TypeScript中，检查typeof返回的值是一种类型守卫。因为TypeScript对typeof在不同值上的操作进行了编码，所以它知道JavaScript中的一些怪癖。例如，请注意在上面的列表中，typeof不返回字符串null。请看下面的示例：

```typescript
function printAll(strs: string | string[] | null) {
  if (typeof strs === "object") {
    for (const s of strs) {
      console.log(s); // 'strs' is possibly 'null'.
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  } else {
    // do nothing
  }
}
```

在printAll函数中，我们尝试检查strs是否是对象，以查看它是否是数组类型（现在可能是一个很好的机会来强调，JavaScript中的数组是对象类型）。但是事实证明，在JavaScript中，null的typeof实际上是"object"！这是历史上不幸的意外之一。

有足够经验的用户可能不会感到惊讶，但不是每个人都在JavaScript中遇到过这种情况；幸运的是，TypeScript让我们知道strs仅被缩小为string[] | null，而不仅仅是string[]。

这可能是一个好的转入到我们所说的“真实性”检查的内容。



## 真值缩小

“真实性”可能不是你在字典中能找到的词，但它在JavaScript中却是非常常见的一个概念。

==**在JavaScript中，我们可以在条件语句、&&、||、if语句、布尔取反（!）等中使用任何表达式**==。例如，if语句并不总是期望它们的条件具有boolean类型。

```typescript
function getUsersOnlineMessage(numUsersOnline: number) {
  if (numUsersOnline) {
    return `There are ${numUsersOnline} online now!`;
  }
  return "Nobody's here. :(";
}
```

==**在JavaScript中，像if这样的结构首先将它们的条件“强制转换”为布尔值，以理解它们，然后根据结果是true还是false来选择它们的分支。像0、NaN、""（空字符串）、0n（bigint版本的零）、null、undefined这样的值都会强制转换为false，而其他值则会被强制转换为true。您可以通过将它们通过Boolean函数，或使用较短的双重布尔取反来始终将值强制转换为布尔值。（后者的优点是TypeScript推断出一个狭窄的字面布尔类型true，而将前者推断为boolean类型。）**==

```typescript
// 下面两个都会得到 true
Boolean("hello"); // 类型: boolean, 值: true
!!"world"; // 类型: true,    值: true
```

这种行为相当受欢迎，特别是为了保护免受null或undefined等值的影响。例如，让我们尝试将其用于我们的printAll函数。

```typescript
function printAll(strs: string | string[] | null) {
  if (strs && typeof strs === "object") {
    for (const s of strs) {
      console.log(s);
    }
  } else if (typeof strs === "string") {
    console.log(strs);
  }
}
```

你会注意到，我们通过检查strs是否为truthy来消除了上面的错误。这至少避免了我们运行代码时遇到以下可怕错误：

```shell
TypeError: null is not iterable
```

请记住，对于基本类型进行真实性检查通常容易出错。例如，考虑另一种编写printAll的尝试。

```typescript
function printAll(strs: string | string[] | null) {
  // !!!!!!!!!!!!!!!!
  //  DON'T DO THIS!
  //   KEEP READING
  // !!!!!!!!!!!!!!!!
  if (strs) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

我们用一个真实性检查来包裹整个函数体，但这有一个微妙的缺点：我们可能不再正确处理空字符串的情况。

TypeScript在这里并没有对我们造成伤害，但如果您对JavaScript不太熟悉，这种行为值得一提。==**TypeScript通常可以帮助您尽早捕获错误，但是如果您选择不对一个值进行任何处理，那么它可以做的事情就有限了，而不会过度指导。如果您愿意，您可以使用linter来确保您处理了这些情况。**==

最后，关于通过真实性缩小类型的问题是，使用!的布尔取反可以从取反分支中过滤掉值。

```typescript
function multiplyAll(
  values: number[] | undefined,
  factor: number
): number[] | undefined {
  if (!values) {
    return values;
  } else {
    return values.map((x) => x * factor);
  }
}
```



## 相等性缩小

==**TypeScript中也使用switch语句和等式检查符号（如`===、!==、==、!=`）来缩小类型**==。例如：

```typescript
function example(x: string | number, y: string | boolean) {
  if (x === y) {
    // 我们现在可以在 'x' 或 'y' 上调用任何 'string' 方法。
    x.toUpperCase();
    y.toLowerCase();
  } else {
    console.log(x);
    console.log(y);
  }
}
```

当我们在上面的例子中检查x和y是否相等时，TypeScript知道它们的类型也必须相等。由于字符串是x和y都可能采用的唯一公共类型，TypeScript知道在第一个分支中x和y必须是字符串。

检查特定字面值（而不是变量）也可以工作。在我们关于真实性缩小的部分中，我们编写了一个printAll函数，它容易出错，因为它意外地未正确处理空字符串。相反，我们可以进行特定的检查来阻止null，并且TypeScript仍然正确地从strs类型中删除null。

```typescript
function printAll(strs: string | string[] | null) {
  if (strs !== null) {
    if (typeof strs === "object") {
      for (const s of strs) {
        console.log(s);
      }
    } else if (typeof strs === "string") {
      console.log(strs);
    }
  }
}
```

==**JavaScript的宽松等式检查符号`==`和`!=`也可以正确缩小类型**==。如果您不熟悉，检查某些东西是否等于null实际上不仅检查它是否是特定的null值，还检查它是否可能是undefined。同样适用于`== undefined`：它检查值是否为null或undefined。

```typescript
interface Container {
  value: number | null | undefined;
}
 
function multiplyValue(container: Container, factor: number) {
  // 从类型中删除 'null' 和 'undefined'。
  if (container.value != null) {
    console.log(container.value);
    // 现在我们可以安全地将 'container.value' 乘以因子。
    container.value *= factor;
  }
}
```



## in缩小

==**JavaScript有一个用于确定对象或其原型链中是否具有名称为属性的运算符：in运算符。TypeScript考虑了这一点作为缩小潜在类型的一种方式。**==

例如，对于代码："value" in x，其中"value"是一个字符串字面量，而x是一个联合类型。"true"分支缩小了具有可选或必需属性value的x类型，而"false"分支缩小了具有可选或缺失属性value的类型。

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
 
function move(animal: Fish | Bird) {
  if ("swim" in animal) {
    return animal.swim();
  }
 
  return animal.fly();
}
```

**==需要强调的是，可选属性在缩小时将存在于两个分支中==**。例如，人类既可以游泳又可以飞行（通过正确的装备），因此应该在in检查的两个分支中都出现：

```typescript
type Fish = { swim: () => void };
type Bird = { fly: () => void };
type Human = { swim?: () => void; fly?: () => void };
 
function move(animal: Fish | Bird | Human) {
  if ("swim" in animal) {
    animal; // 缩小为 Fish | Human 类型
  } else {
    animal; // 缩小为 Bird | Human 类型
  }
}
```



## `instanceof`缩小

**==JavaScript有一个运算符用于检查一个值是否是另一个值的“实例”。更具体地说，在JavaScript中，x instanceof Foo检查x的原型链是否包含Foo.prototype==**。虽然我们不会在这里深入探讨，但当我们进入类时，它们仍然可以对大多数可以用new构造的值很有用。正如您可能已经猜到的那样**==，instanceof也是一种类型守卫，并且TypeScript会在instanceof守卫的分支中进行类型缩小。==**

```typescript
function logValue(x: Date | string) {
  if (x instanceof Date) {
    console.log(x.toUTCString());
    // 缩小 x 的类型为 Date
  } else {
    console.log(x.toUpperCase());
    // 缩小 x 的类型为 string
  }
}
```



## 赋值

正如我们之前提到的，**==当我们对任何变量进行赋值时，TypeScript会查看赋值语句的右侧并相应地缩小左侧的类型==**。

```typescript
let x = Math.random() < 0.5 ? 10 : "hello world!";
   
let x: string | number
x = 1;
 
console.log(x); // 输出 1，缩小为 number 类型
 // let x: number

x = "goodbye!"; // 报错，无法将 string 类型赋值给 number 类型

console.log(x);
 // let x: string
```

**==请注意，每个赋值都是有效的。即使在第一个赋值后，x的观察类型更改为number，我们仍然可以将字符串赋给x。这是因为x的声明类型 - x开始时的类型 - 是string | number，并且可分配性始终针对声明类型进行检查。==**

如果我们将布尔值赋给x，我们将看到一个错误，因为它不是声明类型的一部分。

```typescript
let x = Math.random() < 0.5 ? 10 : "hello world!";
   
let x: string | number
x = 1;
 
console.log(x); // 输出 1，缩小为 number 类型
           
let x: number
x = true; // 报错，无法将 boolean 类型赋值给 string | number 类型
 
console.log(x);
           
// let x: string | number
```



## 控制流分析

在之前的示例中，我们介绍了TypeScript在特定分支中进行缩小的一些基本示例。但是，不仅仅是从每个变量向上查找类型守卫，还有更多的内容，例如在if、while、条件语句等中。

```typescript
function padLeft(padding: number | string, input: string) {
  if (typeof padding === "number") {
    return " ".repeat(padding) + input;
  }
  return padding + input;
}
```

在padLeft函数的第一个if块中，有一个return语句。TypeScript能够分析这段代码，并且可以看到在padding为number的情况下，函数的其余部分（return padding + input;）是不可达的。因此，TypeScript能够在函数的其余部分中将padding的类型从string | number缩小为string。

**==这种基于可达性的代码分析称为控制流分析，TypeScript使用这种流分析在遇到类型守卫和赋值时缩小类型。当分析一个变量时，控制流可以一次又一次地分离和重新合并，而该变量在每个点上的类型可能是不同的。==**

```typescript
function example() {
  let x: string | number | boolean;
 
  x = Math.random() < 0.5;
 
  console.log(x); // 可能是 boolean 类型
 
  if (Math.random() < 0.5) {
    x = "hello";
    console.log(x); // 缩小为 string 类型
  } else {
    x = 100;
    console.log(x); // 缩小为 number 类型
  }
 
  return x; // 缩小为 string | number 类型
}
```

==**在example函数中，变量x的类型在不同的位置可能是不同的。通过控制流分析，TypeScript能够根据条件的可达性缩小变量的类型。在每个分支中，x的类型都会被缩小为相应的值的类型，最终返回的x类型为string | number。**==



## 类型断言

到目前为止，我们已经使用现有的JavaScript构造来处理类型缩小，但有时您可能希望更直接地控制代码中的类型如何改变。

**==要定义一个用户定义的类型守卫，我们只需定义一个返回类型为类型谓词的函数：==**

```typescript
function isFish(pet: Fish | Bird): pet is Fish {
  return (pet as Fish).swim !== undefined;
}
```

**==在这个示例中，pet is Fish是我们的类型谓词。谓词采用参数名 is 类型 的形式，其中参数名必须是当前函数签名中的参数名。==**


==**pet is Fish 是类型谓词**==
==**pet是参数 必须是联合类型**==
==**Fish是联合类型的一种**==
==**类型谓词一般用于返回布尔类型的函数**==
==**如果返回true  说明参数pet是Fish**==
==**返回false 说明参数pet是Bird**==

每当调用isFish并传入某个变量时，TypeScript会将该变量缩小为具体的类型，前提是原始类型是兼容的。

```typescript
// 'swim' 和 'fly' 的调用现在都可以通过类型检查。
let pet = getSmallPet();
 
if (isFish(pet)) {
  pet.swim();
} else {
  pet.fly();
}
```

请注意，TypeScript不仅知道在if分支中pet是一个Fish；它还知道在else分支中，您没有一个Fish，所以您必须有一个Bird。

您可以使用类型守卫isFish来过滤Fish | Bird数组并获得Fish数组：

```typescript
const zoo: (Fish | Bird)[] = [getSmallPet(), getSmallPet(), getSmallPet()];
const underWater1: Fish[] = zoo.filter(isFish);
// 或者等价地
const underWater2: Fish[] = zoo.filter(isFish) as Fish[];
 
// 对于更复杂的例子，谓词可能需要重复
const underWater3: Fish[] = zoo.filter((pet): pet is Fish => {
  if (pet.name === "sharkey") return false;
  return isFish(pet);
});
```

此外，类也可以使用 is 类型 进行缩小类型。

您可以在类和接口的方法的返回位置使用 this is 类型 来实现基于 this 的类型守卫。当与类型缩小（例如 if 语句）混合使用时，目标对象的类型将缩小为指定的类型。

```typescript
class FileSystemObject {
  isFile(): this is FileRep {
    return this instanceof FileRep;
  }
  isDirectory(): this is Directory {
    return this instanceof Directory;
  }
  isNetworked(): this is Networked & this {
    return this.networked;
  }
  constructor(public path: string, private networked: boolean) {}
}

class FileRep extends FileSystemObject {
  constructor(path: string, public content: string) {
    super(path, false);
  }
}

class Directory extends FileSystemObject {
  children: FileSystemObject[];
}

interface Networked {
  host: string;
}

// 子类赋值给父类 --- 多态
const fso: FileSystemObject = new FileRep("foo/bar.txt", "foo");

if (fso.isFile()) {
  fso.content; // 类型缩小为 FileRep
} else if (fso.isDirectory()) {
  fso.children; // 类型缩小为 Directory
} else if (fso.isNetworked()) {
  fso.host; // 类型缩小为 Networked & FileSystemObject
}
```

基于 this 的类型守卫的常见用例是允许对特定字段进行延迟验证。例如，在验证 hasValue 为 true 之后，该示例将从 box 中的值中移除 undefined：

```typescript
class Box<T> {
  value?: T;

  hasValue(): this is { value: T } {
    return this.value !== undefined;
  }
}

const box = new Box();
box.value = "Gameboy";

box.value; // 类型为 unknown

if (box.hasValue()) {
  box.value; // 类型缩小为 T
}
```

通过使用基于 this 的类型守卫，可以在验证条件满足时缩小值的类型，从而提供更准确的类型信息。



## 断言函数

在 TypeScript 中，为了处理类型检查的不足，引入了断言签名（assertion signatures）的概念，用于建模断言函数。

**==断言签名有两种类型。第一种类型模拟了 Node 的 assert 函数的工作方式。它确保被检查的条件在包含的作用域中必须为 true。==**

```typescript
function assert(condition: any, msg?: string): asserts condition {
  if (!condition) {
    throw new AssertionError(msg);
  }
}
```

==`asserts condition` 表示传入 `condition` 参数的内容必须为 true，否则将抛出错误==。这意味着在其余的作用域中，该条件必须为真。通过这个断言函数的例子，我们确实捕捉到了之前的 `yell` 示例中的错误。

```typescript
function yell(str) {
  assert(typeof str === "string");
  return str.toUppercase();
  //         ~~~~~~~~~~~
  // 错误: 类型 'string' 上不存在属性 'toUppercase'。
  //        你是否想写 'toUpperCase'？
}
```

另一种类型的断言签名不是检查条件，而是告诉 TypeScript 特定的变量或属性具有不同的类型。

```typescript
function assertIsString(val: any): asserts val is string {
  if (typeof val !== "string") {
    throw new AssertionError("Not a string!");
  }
}
```

在 `asserts val is string` 中，确保在调用 `assertIsString` 后，传入的任何变量将被知道是一个字符串。

```typescript
function yell(str: any) {
  assertIsString(str);
  // 现在 TypeScript 知道 'str' 是一个 'string' 类型。
  return str.toUppercase();
  //         ~~~~~~~~~~~
  // 错误: 类型 'string' 上不存在属性 'toUppercase'。
  //        你是否想写 'toUpperCase'？
}
```

这些断言签名与类型谓词签名非常相似，它们都非常具有表达性。通过这些断言签名，我们可以表达一些相当复杂的想法。

```typescript
function assertIsDefined<T>(val: T): asserts val is NonNullable<T> {
  if (val === undefined || val === null) {
    throw new AssertionError(
      `Expected 'val' to be defined, but received ${val}`
    );
  }
}
```



## 辨别联合

大多数我们目前看过的示例都集中在缩小单一变量的范围上，涉及到的是像字符串、布尔值和数字这样的简单类型。虽然这很常见，但在JavaScript中，我们通常会处理稍微复杂一些的结构。

为了一些动机，让我们想象我们正在尝试编码圆形和正方形这样的形状。圆形保持其半径，而正方形保持其边长。我们将使用一个名为kind的字段来告诉我们正在处理的是哪种形状。下面是对Shape的第一次尝试定义。

```typescript
interface Shape {
  kind: "circle" | "square";
  radius?: number;
  sideLength?: number;
}
```

请注意，我们使用了字符串字面类型的联合："circle"和"square"，以告诉我们是否应将形状视为圆形或正方形。通过使用"circle" | "square"而不是字符串，我们可以避免拼写错误。

```typescript
function handleShape(shape: Shape) {
  // oops!
  if (shape.kind === "rect") {
    // ...
  }
}
```

此比较似乎是无意的，因为类型'"circle" | "square"'和'"rect"'没有重叠。

我们可以编写一个getArea函数，根据处理的是圆形还是正方形应用正确的逻辑。我们先尝试处理圆形。

```typescript
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

在启用了strictNullChecks的情况下，这会给我们一个错误提示，这是正确的，因为半径可能未定义。但是，如果我们对kind属性执行适当的检查，会发生什么呢？

```typescript
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
}
```

嗯，TypeScript仍然不知道该怎么办。我们遇到了一个问题，我们对值的了解比类型检查器还要多。我们可以尝试使用非空断言（在shape.radius之后加上!）来表示半径肯定存在。

```typescript
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius! ** 2;
  }
}
```

但是这并不理想。我们不得不对类型检查器大喊，使用那些非空断言（!）来说服它shape.radius已定义，但是如果我们开始移动代码，这些断言是容易出错的。此外，在strictNullChecks之外，我们仍然可以意外访问任何这些字段（因为在读取它们时，可选属性被假定始终存在）。我们肯定可以做得更好。

这种Shape的编码问题在于类型检查器无法根据kind属性知道radius或sideLength是否存在。我们需要向类型检查器传达我们的了解。考虑到这一点，让我们重新定义Shape。

```typescript
interface Circle {
  kind: "circle";
  radius: number;
}
 
interface Square {
  kind: "square";
  sideLength: number;
}
 
type Shape = Circle | Square;
```

在这里，我们将Shape适当地分解为两种具有不同kind属性值的类型，但是radius和sideLength在它们各自的类型中被声明为必需属性。

让我们看看当我们尝试访问Shape的radius时会发生什么。

```typescript
function getArea(shape: Shape) {
  return Math.PI * shape.radius ** 2;
}
```

这仍然是一个错误。当radius是可选的时，我们得到一个错误（在启用strictNullChecks时），因为TypeScript无法确定该属性是否存在。现在Shape是一个联合类型，TypeScript告诉我们shape可能是一个Square，而Squares上没有定义radius！这两种解释都是正确的，但只有Shape的联合编码会导致错误，无论strictNullChecks如何配置。

但是，如果我们再次检查kind属性会发生什么呢？

```typescript
function getArea(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.radius ** 2;
  }
}
```

错误消失了！**==当联合中的每个类型都包含具有字面类型的公共属性时，TypeScript将其视为具有辨别联合，并可以缩小联合的成员类型。==**

==在这种情况下，kind就是那个公共属性（这被认为是Shape的辨别属性）==。检查kind属性是否为"circle"消除了没有kind属性类型为"circle"的Shape的所有类型。这将shape缩小为类型Circle。

相同的检查也适用于switch语句。现在我们可以尝试编写完整的getArea而不需要使用烦人的!非空断言。

```typescript
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
  }
}
```

这里重要的是Shape的编码方式。向TypeScript传达正确的信息——Circle和Square实际上是具有特定kind字段的两种不同类型——是至关重要的。这样做让我们能够编写与我们原本会编写的JavaScript没有任何区别的类型安全的TypeScript代码。从那里开始，类型系统能够做出“正确”的决策，并在switch语句的每个分支中推断出类型。

顺便说一句，尝试使用上面的示例并删除一些return关键字。您会发现在switch语句的不同分支中意外地跳转时，类型检查可以帮助避免错误。

辨别联合不仅适用于圆形和正方形这样的概念，它们还适用于表示JavaScript中的任何消息方案，例如在网络上发送消息（客户端/服务器通信）或在状态管理框架中编码变更。



### never类型

==**当缩小范围时，您可以将联合类型的选项减少到没有任何可能性和剩余内容的程度。在这些情况下，TypeScript 将使用 never 类型来表示一个不应该存在的状态。**==



### 穷尽性检查

**==`never`类型可以分配给任何类型，但没有类型可以分配给`never`（除了`never`本身）。这意味着我们可以使用缩小操作并依赖于`never`类型来进行完备性检查，确保在`switch`语句中处理了所有可能的情况。==**

例如，如果我们在`getArea`函数中添加一个`default`分支，并尝试将形状赋值给`never`类型，那么当处理了所有可能的情况时，不会引发错误。

```typescript
type Shape = Circle | Square;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

尝试给`Shape`联合类型添加一个新的成员，将会导致 TypeScript 报错：

```typescript
interface Triangle {
  kind: "triangle";
  sideLength: number;
}

type Shape = Circle | Square | Triangle;

function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
// Type 'Triangle' is not assignable to type 'never'.
      return _exhaustiveCheck;
  }
}
```

在这个例子中，当我们尝试将`Triangle`类型赋值给`never`类型时，TypeScript会报错，提示我们存在未处理的类型。这个错误是 TypeScript 提供的一种保护机制，确保我们在`switch`语句中处理了所有可能的情况。

这种使用`never`类型进行完备性检查的方法可以帮助我们编写更健壮和可靠的代码，确保我们没有遗漏任何情况。





