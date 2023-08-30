枚举允许开发人员定义一组字符串或数值定义方便理解的命名常量。使用时对应值将会是所有枚举值中的任意一个

枚举值会修改JS的运行时行为

TypeScript提供了基于数字和基于字符串的两种枚举类型。



数字枚举

```ts
// 枚举 一般首字母大写 约定俗成
enum Direction {
  // 枚举值一般为常量 所以枚举成员名一般为全大写或大驼峰 --- 约定俗成
  Up = 1,
  Down,
  Left,
  Right,
}
```

我们有一个数字枚举，其中Up的初始值为1。从那点开始，所有后续的成员都会自动递增。换句话说，Direction.Up的值为1，Down的值为2，Left的值为3，Right的值为4。

如果我们希望，我们也可以完全省略初始值

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}

type DirectionStrings = keyof typeof Direction; // Up | Down | Left | Right
```

在这种情况下，Up的值将为0，Down的值将为1，以此类推。

这种自动递增的行为在我们可能不关心成员值本身，但确保每个值在同一枚举中都是不同的情况下非常有用



枚举在运行时是真实存在的对象

上述枚举在运行时会被编译为

```ts
"use strict";
var Direction;
(function (Direction) {
    Direction[Direction["Up"] = 0] = "Up";
    Direction[Direction["Down"] = 1] = "Down";
    Direction[Direction["Left"] = 2] = "Left";
    Direction[Direction["Right"] = 3] = "Right";
})(Direction || (Direction = {}));
```



除了创建一个具有成员属性名称的对象之外，数值枚举成员还获得了从枚举值到枚举名称的反向映射

它存储正向映射（名称->值）和反向映射（值->名称)

请注意，字符串枚举成员根本不会生成反向映射。

```ts
console.log(Direction.Up) // 0
console.log(Direction['Up']) // 0
console.log(Direction[0]) // Up
```



字符串枚举

在字符串枚举中，每个成员必须使用字符串字面量或另一个字符串枚举成员进行常量初始化

字符串枚举没有自动递增行为，但是字符串枚举值是有语义的

```ts
enum Direction {
  Up = "UP",
  Down = "DOWN",
  Left = "LEFT",
  Right = "RIGHT",
}
```



异构枚举

异构枚举就是枚举值既有字符串也有数值的枚举，但一般没用实际意义，很少使用

```ts
enum BooleanLikeHeterogeneousEnum {
  No = 0,
  Yes = "YES",
}
```



计算和常量成员

每个枚举成员都有与之关联的值，可以是常量或计算得到的

如果枚举成员满足以下条件，则被视为常量：

1. 它是枚举中的第一个成员且没有初始化器，此时它被赋值为0
2. 如果一个枚举成员没有初始化器，并且前一个枚举成员是一个数字常量，那么当前枚举成员的值将是前一个枚举成员的值加一
3. 枚举成员通过常量枚举表达式进行初始化 
   + 在编译时可以计算出结果的表达式 就是 常量枚举表达式
   + 常量枚举表达式计算为NaN或Infinity会导致编译时错误
   + 在表达式中使用另一个常量枚举值 或 通过常量枚举值进行四则运算或二进制运算就可以得到最终结果的表达式

其余任何情况下，枚举成员都被视为是计算得出的



没有初始值的枚举要么需要放在最前面，要么需要放在使用数字常量或其他常量枚举成员初始化的数字枚举之后。

换句话说，以下情况是不允许的, 因为此时不知道getSomeValue方法的返回值类型 无法判断枚举成员B的值是否可以通过自动递增得出

```ts
enum E {
  A = getSomeValue(),
  B,
// Enum member must have initializer.
}
```



联合枚举和枚举成员类型

当枚举的所有成员都具有字面值枚举值时，会出现一些特殊的语义

1. 枚举成员也会成为类型, 这种类型被叫做枚举成员类型

   ```ts
   enum ShapeKind {
     Circle,
     Square,
   }
    
   // 因为ShapeKind是数值枚举 所以可以看成ShapeKind的所有成员都是数值字面量
   // 此时 ShapeKind.Circl 和 ShapeKind.Square 都可以看成是一种TS类型
   interface Circle {
     kind: ShapeKind.Circle;
     radius: number;
   }
   ```

2. 枚举类型本身实际上成为了每个枚举成员的联合类型，这种类型被叫做联合枚举

   ```ts
   enum E {
     Foo,
     Bar,
   }
   // type E = 0 | 1
   ```



为了避免支付额外生成的代码和访问枚举值时的额外间接性的成本(普通枚举在运行时是实际存在的对象，并且获取对应值需要通过访问对象属性的方式才可以获取到时间ide值)，可以使用const枚举

```ts
const enum Enum {
  A = 1,
  B = A * 2,
}
```



const枚举只能使用常量枚举表达式，因为const枚举和常规枚举不同，它们在编译期间完全被删除。const枚举成员会在使用的地方被内联。

```ts
const enum Direction {
  Up,
  Down,
  Left,
  Right,
}
 
let directions = [
  Direction.Up,
  Direction.Down,
  Direction.Left,
  Direction.Right,
];
```

会被编译为

```ts
"use strict";
let directions = [
    0 /* Direction.Up */,
    1 /* Direction.Down */,
    2 /* Direction.Left */,
    3 /* Direction.Right */,
];
```



环境枚举

在TypeScript中，如果在枚举关键字之前加上`declare`，则该枚举被称为环境枚举（ambient enum）

环境枚举（Ambient enums）用于描述已经存在的枚举类型的形状

环境枚举的成员都是计算枚举成员

在环境枚举中，没有自动递增行为，因为环境枚举仅用于描述已经存在的枚举类型的形状，而不会生成实际的 JavaScript 代码

因此，环境枚举的成员必须显式地指定其值，而不能依赖于自动递增

```ts
declare enum Enum {
  A = 1, // 1
  B, // undefined
  C = 2, // 2
}
```



枚举完成的功能和常量对象完成的功能在本质上是一致的

但是使用常量对象可以更好地与原生 JavaScript 对象进行兼容

因为枚举是TS的扩展功能，而常量对象是JS的原生功能

因此尽可能地使用常量对象而不是枚举是一种较为推荐的做法

```ts
const enum EDirection {
  Up,
  Down,
  Left,
  Right,
}

const ODirection = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3,
} as const;

// Using the enum as a parameter
function walk(dir: EDirection) {}

// It requires an extra line to pull out the values
type Direction = typeof ODirection[keyof typeof ODirection];
function run(dir: Direction) {}

walk(EDirection.Left);
run(ODirection.Right);
```





----

const枚举存在一些潜在的问题。这些问题主要适用于环境中的const枚举（即.d.ts文件中的const枚举）以及在项目之间共享const枚举。如果你发布或使用.d.ts文件，那么这些问题很可能适用于你，因为tsc --declaration会将.ts文件转换为.d.ts文件。

首先，在isolatedModules文档中提到的原因导致了环境中的const枚举与isolatedModules模式的不兼容。这意味着如果你发布环境中的const枚举，下游的消费者将无法同时使用isolatedModules模式和这些枚举值。

其次，你可以在编译时轻松内联来自依赖项版本A的值，并在运行时导入版本B。如果不非常小心，版本A和B的枚举可以具有不同的值，这可能导致出现意外的错误，例如执行错误的if语句分支。这些错误尤其隐蔽，因为通常会在构建项目的同时运行自动化测试，并且使用相同的依赖版本，这会完全忽略这些错误。

另外，当使用importsNotUsedAsValues: "preserve"时，对于作为值使用的const枚举，导入不会被省略。但是，环境中的const枚举无法保证存在运行时的.js文件。这些无法解析的导入会导致运行时错误。通常，为了确保导入被省略，可以使用只用于类型的导入（type-only imports），但当前不支持将const枚举值用于只用于类型的导入。

以下是避免这些问题的两种方法：

1. 完全不使用const枚举。你可以借助代码检查工具来禁止使用const枚举。显然，这样可以避免任何与const枚举相关的问题，但也会阻止项目内联自身的枚举。与从其他项目内联枚举不同，内联项目自身的枚举不会引发问题，并且会对性能产生影响。

1. 不发布环境中的const枚举，而是使用preserveConstEnums来取消const枚举的const限定符。这就是TypeScript项目本身内部采用的方法。preserveConstEnums会将const枚举生成与普通枚举相同的JavaScript代码。然后，你可以在构建步骤中安全地从.d.ts文件中删除const限定符。

通过这种方式，下游的消费者将不会内联你项目中的枚举，避免了上述的问题，但项目仍然可以内联自己的枚举，而不是完全禁止使用const枚举。
