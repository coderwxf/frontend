函数是任何应用程序的基本构建块，无论是本地函数、从另一个模块导入的函数，还是类的方法



## 类型表达式

`(a: string) => void` 的语法意味着“一个带有一个参数 a，类型为 string，没有返回值的函数”。

就像函数声明一样，如果没有指定参数类型，它会被隐式地设为 any 类型

请注意，参数名是必需的。函数类型 `(string) => void` 的意思是“一个带有一个名为 string，类型为 any 的参数的函数”！



## Call Signatures（调用签名）

在JavaScript中，函数除了可被调用之外，还可以具有属性

函数类型表达式的语法不允许声明属性。如果我们想描述一个具有属性的可调用对象，可以在对象类型中编写一个调用签名

```ts
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};
```

请注意，与函数类型表达式相比，语法略有不同 - 在参数列表和返回类型之间使用`:`而不是`=>`。



## 构造函数

JavaScript函数也可以使用new操作符调用。TypeScript将其称为构造函数，因为它们通常用于创建新对象。您可以通过在调用签名前面添加new关键字来编写构造函数签名

```ts
type SomeConstructor = {
  new (s: string): SomeObject;
};
```



## 泛型函数

```ts
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}

function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}
```



在可能的情况下，使用类型参数本身而不是约束它

```ts
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}
 
// 当存在约束类型时，TypeScript会使用约束类型来表示泛型类型，而不会根据实际传入的参数类型来推导更具体的类型。
function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);

// b: any (bad)
const b = firstElement2([1, 2, 3]);
```



## 可选参数

JavaScript中的函数经常接受可变数量的参数

在TypeScript中，我们可以通过在参数后面添加?来将参数标记为可选的

```ts
function f(x?: number) {
  // ...
}
```

x的类型将为number | undefined，因为在JavaScript中未指定的参数的值为undefined。

还可以为参数提供默认值

```ts
function f(x = 10) {
  // ...
}
```



**当为回调函数编写函数类型时，除非你打算在调用函数时不传递该参数，否则不要编写可选参数**

在JavaScript中，如果你调用一个函数时提供的参数多于函数定义中的参数数量，多余的参数将被忽略。

TypeScript的行为与此相同。**参数较少（类型相同）的函数始终可以取代参数较多的函数**

```ts
// 虽然在实际调用的时候 index可能没有实际值 但是这是回调函数的类型 所以不要使用可选类型
// 因为在调用的时候 我可以将index传递给调用者 而调用者在实际调用的时候可以不使用索引值，也可以使用索引值
// 但如果设置index为可选类型 表示在回调函数实际定义的时候 要手动判断index是否存在
function myForEach(arr: any[], callback: (arg: any, index: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

```ts
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```



## 函数重载

某些JavaScript函数可以以多种参数数量和类型进行调用

在TypeScript中，我们可以通过编写重载签名来指定可以以不同方式调用的函数

为此，先编写一些函数签名（通常是两个或更多），然后是函数的实现体

尽可能的优先使用联合类型 而不是使用函数重载

```ts
// 重载签名
function makeDate(timestamp: number): Date;
function makeDate(m: number, d: number, y: number): Date;

// 具有兼容签名的函数实现 --- 可以简单理解为在类型检测的时候 函数实现对外是不可见的
// 函数有一个实现签名，但不能直接调用该签名。虽然我们在必需参数之后编写了两个可选参数的函数，但不能用两个参数来调用它！

// 函数实现的参数和返回值 都必须兼容所有的函数重载
function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
  if (d !== undefined && y !== undefined) {
    return new Date(y, mOrTimestamp, d);
  } else {
    return new Date(mOrTimestamp);
  }
}
const d1 = makeDate(12345678);
const d2 = makeDate(5, 5, 5);
const d3 = makeDate(1, 3);
```



## this

在函数中，关键字 `this` 表示当前函数所属的对象。TypeScript可以通过代码分析来推断函数中的 `this` 是什么对象，但有些情况下我们需要手动指定 `this` 的类型

通常在回调函数的场景中会遇到这种情况，其中另一个对象会控制何时调用我们的函数。为了在函数体内声明 `this` 的类型，我们可以使用 `(this: 类型)` 的语法



```ts
interface User {
  id: number;
  admin: boolean;
}
declare const getDB: () => DB;

interface DB {
  // this是ts中用于指定this类型的特殊形参 会在运行时被移除
  // this必须在参数列表的第一位
  filterUsers(filter: (this: User) => boolean): User[];
}

const db = getDB();
const admins = db.filterUsers(function () {
  return this.admin;
});
```



## 其它类型

### void

`void` 代表不返回任何值的函数的返回类型。当一个函数没有返回语句，或者返回语句没有返回具体的值时，`void` 被推断为返回类型：

在 JavaScript 中，如果一个函数没有返回任何值，它会隐式地返回 `undefined`  所以undefined是void的子类型

```ts
// 推断的返回类型是 void
function noop() {
  return;
}
```



### object

特殊类型 `object` 指代任何非原始类型的值（字符串、数字、大整数、布尔、符号、null 或 undefined）

它与空对象类型 `{}` 不同，也与全局类型 `Object` 不同

尽可能不要使用object 使用接口或类型别名来指定对象类型

| 类型   | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| object | 任何的非原始数据类型值                                       |
| {}     | 没有自定义的属性和方法的空对象                               |
| Object | 任何的对象类型值<br />基本数据（除null 和 undefined外）都可以视为是对应包装类类型实例<br />所以可以赋值给Object |

```ts
let o1: {} = {}
let o2: Object = {}
let o3: object = {}

o2 = undefined // error
o2 = 1111 // success
o3 = 1111 // error
```



### unknown

`unknown` 类型表示任意值。它类似于 `any` 类型，但更安全，因为不能对 `unknown` 类型的值执行任何操作：



### never

有些函数永远不会返回值：如 抛出异常的函数 或死循环的函数



`never` 类型表示永远不会观察到的值。在返回类型中，这意味着函数抛出异常或终止程序的执行。

```ts
function fail(msg: string): never {
  throw new Error(msg);
}
```



`never` 也会在 TypeScript 确定联合类型中没有剩余项时出现

```ts
function fn(x: string | number) {
  if (typeof x === "string") {
    // 做某事
  } else if (typeof x === "number") {
    // 做其他事情
  } else {
    x; // 类型为 'never'！
  }
}
```



### Function

全局类型 `Function` 描述了 JavaScript 中所有函数值的属性，如 `bind`、`call`、`apply` 等。

它还具有特殊属性，即类型为 `Function` 的值始终可以调用；这些调用返回 `any` 类型

尽可能的不要使用Function，而是还有函数类型表达式

```ts
function doSomething(f: Function) {
  return f(1, 2, 3);
}
```



## Rest参数

除了使用可选参数或重载来创建可以接受各种固定参数数量的函数外，我们还可以使用rest参数定义可以接受任意数量参数的函数。

rest参数出现在其他参数之后，并使用...语法
