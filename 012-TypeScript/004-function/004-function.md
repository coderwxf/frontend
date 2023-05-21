函数是构成任何应用程序的基本部件，无论是在本地定义的函数，还是从其他模块导入的函数，或者是类的方法

在TypeScript中，函数也可以被认为是一种数据类型



## 函数类型表达式

描述函数的最简单方法是使用函数类型表达式。这些类型在语法上类似于箭头函数

```ts
// (a: string) => void 
//      --- 有一个字符串参数的函数，该函数不会存在任何的返回值
//      --- a为参数名  string为参数类型，默认为any

//      --- 参数名是必需 
//      ---  (string) => void表示的是一个名为 string，类型为 any 的参数的函数
type GreetFunction = (a: string) => void;
```



## 调用签名

在 JavaScript 中，函数除了可以被调用外，还可以具有属性，然而，函数类型表达式语法不允许声明属性

如果我们想描述一个具有属性的可调用对象，可以在对象类型中编写一个调用签名

```ts
type DescribableFunction = {
  // 函数的属性
  description: string;
  // 函数的调用签名 --- 使用冒号分割
  (someArg: number): boolean;
};
```



## 构造签名

JavaScript 函数也可以使用 `new` 操作符调用

TypeScript 将其称为构造函数，因为它们通常创建一个新对象，可以通过在调用签名前面添加 `new` 关键字来编写构造签名

```ts
type SomeConstructor = {
  new (s: string): any;
}
```

有些对象，可以使用或不使用 `new` 进行调用（例如 JavaScript 的 Date 对象）此时可以任意组合调用和构造签名

```ts
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): number;
}
```



## 泛型函数

编写一个函数，其中输入类型与输出类型相关联，或两个输入类型以某种方式相关联，就可以使用泛型函数

在 TypeScript 中，泛型用于描述两个值之间的对应关系，所以在实际使用中，泛型必须在一个函数中出现两个以上

如果一个泛型在一个函数中仅出现不操作一次，那么这个泛型参数往往是没有必要的

```ts
// T就是类型参数 即泛型
function firstElement<T>(arr: T[]): T | undefined {
  return arr[0];
}

// 在调用的时候，不需要传递泛型, 类型系统可以通过参数推导出来
const n = firstElement([1, 2, 3]);

// 也可以显示传递
const s = firstElement<string>(["a", "b", "c"]);
```

也可以传递多个类型参数

```ts
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}
```



### 泛型约束

使用泛型约束可以约束泛型类型参数的类型，以确保它们具有某些属性或方法

当在泛型类型参数上使用约束条件时，TypeScript 将强制要求函数参数符合约束类型，并且参数的类型将是类型约束而不是参数本身的类型

虽然在泛型函数中，泛型参数只能使用约束类型中声明的属性和方法，但是返回的类型要求和传入的类型一致(此时不是约束类型)

```ts
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
```



### 指定类型参数

TypeScript 通常可以推断泛型调用中预期的类型参数，但并不总是如此

```ts
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}

// 此时就需要显示指定类型参数
// const arr = combine([1, 2, 3], ["hello"]); 
//     --- Type 不可以是同时是string和number，但可以是string | number
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```



## 可选参数

在 JavaScript 中，函数通常会接受任意数量的参数

在 TypeScript 中，通过在参数后面加上问号来标记它为可选参数

```ts
// 此时 typeof x -> number | undefined
function f(x?: number) {
  // ...
}
```

当然也可以使用默认参数

```ts
// typeof x -> string --- 因为默认情况下，x会被赋值为10 而不是undefined
function f(x = 10) {
  // ...
}
```

在JavaScript中，如果使用的参数比定义时的参数多，多余的参数将被忽略

TypeScript的行为方式也相同。具有较少参数的函数（类型相同）可以始终代替具有更多参数的函数。



## 函数重载

有些 JavaScript 函数可以接受不同数量和类型的参数，在TypeScript中，可以通过编写重载签名来指定可以以不同方式调用的函数

在 TypeScript 中，函数重载是指在同一个函数名下定义多个函数签名，以允许函数接受不同数量和类型的参数

在使用函数重载时，只有在参数个数不同的情况下才需要使用重载，如果参数类型不同，应优先考虑使用联合类型作为参数类型

```ts
// 函数签名 --- 通常是两个及以上 --- 重载签名
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// 实际函数体 --- 需要兼容所有的函数签名 --- 实现签名
// 实现签名不能直接调用，必须通过调用重载签名之一来触发它
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
```



## this

在 TypeScript 中，关键字 this 表示函数的上下文，通常情况下，TypeScript 可以通过代码流分析自动推断出 this 的类型

```ts
const user = {
  id: 123,
  admin: false,
  becomeAdmin: function () {
    this.admin = true;
  },
};
```



但是在一些特定的情况下，可能需要手动指定函数中 this 的类型

JavaScript 规范规定不能有一个名为 this 的参数，因此 TypeScript 使用这个语法空间来在函数体中声明 this 的类型

```ts
interface DB {
  // this参数必须是参数列表中的第一个参数
  filterUsers(filter: (this: User) => boolean): User[];
}
 
const db = getDB();
const admins = db.filterUsers(function (this: User) {
  return this.admin;
});
```



## 剩余参数

除了使用可选参数或重载来创建能够接受各种固定参数数量的函数外，我们还可以使用 rest parameters 定义可以接受无限数量参数的函数

rest parameters 出现在所有其他参数之后，并使用 ... 语法

在TypeScript中，这些参数的类型注释隐式推导为`any[]`，给定的任何类型注释必须是`Array<T>`或`T[]`的形式，或是元组类型

```ts
const args = [8, 5];
// tsc会将剩余参数推导为数组
const angle = Math.atan2(...args); // error

const args = [8, 5] as const;
// 使用as const后 tsc会将剩余参数推导为元组
const angle = Math.atan2(...args);
```



## 解构赋值

可以使用参数解构来方便地将提供的对象解包为一个或多个本地变量，并在函数体中使用它们

在 TypeScript 中，对象的类型注解需要写在解构语法后面

```ts
function sum({ a, b, c }: { a: number; b: number; c: number }) {
  console.log(a + b + c);
}
```



## 函数的可赋值性

```ts
type voidFunc = () => void;
 
// 即使类型声明要求返回void, 但是在赋值的时候，任何返回值类型都可以被赋值
// 只不过在类型推导的时候 typeof f1 -> () => void --- 显示返回的返回值类型会被丢弃 --- 相当于一个宽泛的类型赋值给了一个更严谨的类型
const f1: voidFunc = () => {
  return true;
};
 
const f2: voidFunc = () => true;
 
const f3: voidFunc = function () {
  return true;
};
```

```ts
function f2(): void {
  // 当一个函数显示声明函数的返回值类型是void的时候，则只能返回undefined，其余类型都将是不被允许的
  return true; // error
}
```

