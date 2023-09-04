## keyof

keyof 运算符接受一个对象类型并生成其键的字符串或数字字面量联合类型

```ts
type Point = { x: number; y: number };
type P = keyof Point;
```

如果类型具有字符串或数字索引签名，则 keyof 将返回这些类型

```ts
type Arrayish = { [n: number]: unknown };
type A = keyof Arrayish; // number

// 在这个例子中，M 是 string | number 类型
// 这是因为 JavaScript 对象的键总是被强制转换为字符串
type Mapish = { [k: string]: boolean };
type M = keyof Mapish; // number | string
```



## typeof

值和类型是不同的东西。要引用值 的类型，我们使用 typeof

typeof只能用于标识符（即变量名）或其属性上

```ts
let s = "hello"; 
let n: typeof s; // string
```



## 索引类型

我们可以使用索引访问类型来查找另一个类型上的特定属性

```ts
type Person = { age: number; name: string; alive: boolean };
type Age = Person["age"]; // number
```



索引类型中的索引必须是类型，不可以是具体的值

```ts
type Person = { age: number; name: string; alive: boolean };
const age = "age"
type Age = Person[age]; // error - 此时的age变成了具体值 而不是类型
```



索引类型本身也是一种类型，因此我们可以使用联合类型、keyof 或其他类型来进行索引

如果尝试索引一个不存在的属性，会出现错误

```ts
type I1 = Person["age" | "name"]; // string | number
 
type I2 = Person[keyof Person]; // string | number | boolean
 
type AliveOrName = "alive" | "name";
type I3 = Person[AliveOrName]; //  string | boolean
```

```ts
const MyArray = [
  { name: "Alice", age: 15 },
  { name: "Bob", age: 23 },
  { name: "Eve", age: 38 },
];
 
// 数组本质是key为递增number的对象
type Person = typeof MyArray[number];
/*
	{
    name: string;
    age: number;
	}
*/

type Age = typeof MyArray[number]["age"]; // number
```



## 条件类型

条件类型的形式类似于JavaScript中的条件表达式（condition ? trueExpression : falseExpression）

```ts
SomeType extends OtherType ? TrueType : FalseType;
```

```ts
interface Animal {
  live(): void;
}
interface Dog extends Animal {
  woof(): void;
}
 
type Example1 = Dog extends Animal ? number : string; // number
```



当一个函数需要根据不同的类型来执行不同的操作时，如果不使用条件类型，我们可能需要编写大量的函数重载来区分不同类型的参数及其对应的返回值。

每个函数重载都描述了特定类型的参数和返回值的组合，这样会导致代码冗长且难以维护

```ts
interface IdLabel {
  id: number /* some fields */;
}
interface NameLabel {
  name: string /* other fields */;
}
 
function createLabel(id: number): IdLabel;
function createLabel(name: string): NameLabel;
function createLabel(nameOrId: string | number): IdLabel | NameLabel;

function createLabel(nameOrId: string | number): IdLabel | NameLabel {
  throw "unimplemented";
}
```



而使用条件类型，我们可以将这些类型选择逻辑集中在一个地方，并根据不同的类型条件进行相应的选择。这样可以避免编写大量的函数重载，简化了代码的编写和维护

条件类型在泛型中的应用可以帮助我们更灵活地根据输入类型来确定输出类型，从而使代码更加简洁和可维护。

```ts
type NameOrId<T extends number | string> = T extends number
  ? IdLabel
  : NameLabel;
```

条件类型可以嵌套使用

```ts
type NameOrId<T extends number | string | boolean> = T extends number
  ? IdLabel
  : T extends string ? NameLabel : BoolLabel;
```



### 条件类型约束

条件类型可以通过对类型进行约束，从而在真分支中进一步限制泛型类型。

这种约束的效果类似于类型守卫对类型进行缩小的功能

```ts
type MessageOf<T> = T extends { message: unknown } ? T["message"] : never;

interface Email {
  message: string;
}

interface Dog {
  bark(): void;
}

type EmailMessageContents = MessageOf<Email>;
// 类型EmailMessageContents = string

type DogMessageContents = MessageOf<Dog>;
// 类型DogMessageContents = never
```



在条件类型中进行推断就是根据条件对类型进行判断，并从中推断出相应的类型

使用`infer`关键字可以在条件类型的`true`分支中声明性地引入一个新的泛型类型变量，然后从中推断出类型

```ts
<!-- 我们使用infer关键字引入了一个名为Item的新的泛型类型变量，它用来表示数组的元素类型 -->
type Flatten<Type> = Type extends Array<infer Item> ? Item : Type;
```

```ts
type GetReturnType<Type> = Type extends (...args: never[]) => infer Return
  ? Return
  : never;
 
type Num = GetReturnType<() => number>;   // number
```



当从具有多个调用签名的类型进行推断时（比如函数的重载类型），推断会基于最后一个签名（通常是最通用的情况）

所以推荐在函数重载时，越宽泛的类型写在越后边

```ts
declare function stringOrNum(x: string): number;
declare function stringOrNum(x: number): string;
declare function stringOrNum(x: string | number): string | number;

type T1 = ReturnType<typeof stringOrNum>; // number | string
```



### 分配条件类型（Distributive Conditional Types）

当条件类型作用于一个泛型类型时，如果给定一个联合类型，它们就会变成分布式的

```ts
type ToArray<Type> = Type extends any ? Type[] : never;

// ToArray对于 string | number 进行分配操作，它会分别作用于联合类型的每个成员类型
// 相当于 ToArray<string> | ToArray<number>;
type StrArrOrNumArr = ToArray<string | number>; // string[] | number[];
```

通常情况下，分配行为是期望的行为。为了避免这种行为，可以在extends关键字的两边加上方括号

```ts
type ToArrayNonDist<Type> = [Type] extends [any] ? Type[] : never;

type StrArrOrNumArr = ToArrayNonDist<string | number>; // (string | number)[]
```



## 映射类型

借助索引类型 从一个类型中新建出一个新的类型 那么这个类型就可以被认为是映射类型



索引签名用于声明那些事先未声明的属性的类型：

```ts
// OnlyBoolsAndHorses 就是一个索引类型
type OnlyBoolsAndHorses = {
  [key: string]: boolean | Horse;
};
 
const conforms: OnlyBoolsAndHorses = {
  del: true,
  rodney: false,
};
```



`OptionsFlags`是一个映射类型

```ts
type OptionsFlags<Type> = {
  [Property in keyof Type]: boolean;
};
```



在映射类型中，还有两个额外的修饰符可以应用：readonly 和 ?，分别影响可变性和可选性

通过在修饰符前加上 - 或 +，可以移除或添加这些修饰符。如果没有添加前缀，则默认为 +

```ts
// 从类型的属性中移除 'readonly' 修饰符
type CreateMutable<Type> = {
  -readonly [Property in keyof Type]: Type[Property];
};
 
type LockedAccount = {
  readonly id: string;
  readonly name: string;
};
 
type UnlockedAccount = CreateMutable<LockedAccount>;
```

```ts
// 从类型的属性中移除 'optional' 修饰符
type Concrete<Type> = {
  [Property in keyof Type]-?: Type[Property];
};
 
type MaybeUser = {
  id: string;
  name?: string;
  age?: number;
};
 
type User = Concrete<MaybeUser>;
```



通过`as`关键字 可以在创建映射类型的时候，为属性起新的属性名



## 模板字面量类型

模板字面类型是在字符串字面类型的基础上构建的，它可以通过联合类型展开为多个字符串

们与JavaScript中的模板字面字符串具有相同的语法，但在类型位置上使用。当与具体的字面类型一起使用时，模板字面类型通过连接内容来生成新的字符串字面类型

```ts
type World = "world";
 
type Greeting = `hello ${World}`; // hello world
```

当联合类型用于插入位置时，类型是每个联合成员可能表示的所有字符串字面类型的集合

```ts
type EmailLocaleIDs = "welcome_email" | "email_heading";
type FooterLocaleIDs = "footer_title" | "footer_sendoff";
 
type AllLocaleIDs = `${EmailLocaleIDs | FooterLocaleIDs}_id`; // 2 * 2 = 4 一共四种类型       
// type AllLocaleIDs = "welcome_email_id" | "email_heading_id" | "footer_title_id" | "footer_sendoff_id"
```

