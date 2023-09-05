在JavaScript中，我们通过对象来组织和传递数据。而在TypeScript中，我们通过对象类型来表示这些对象



对象类型可以是匿名的

```ts
function greet(person: { name: string; age: number }) {
  return "你好 " + person.name;
}
```



使用接口来命名

```ts
interface Person {
  name: string;
  age: number;
}
 
function greet(person: Person) {
  return "你好 " + person.name;
}
```



可以使用类型别名

```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "你好 " + person.name;
}
```



## 属性修饰符

对象类型中的每个属性都可以指定一些内容：类型、属性是否可选以及属性是否可写。

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number; // 可选属性 --- number | undefined
  yPos?: number; // undefined可以通过设置默认值来移除
}
 
function paintShape(opts: PaintOptions) {
  // ...
}
```

```ts
interface SomeType {
  readonly prop: string; // 只读 类型检测期间是只读的 --- 运行时是可读可写的 --- ts不改变js的运行时
}
```

readonly 和 const类似 它只是表示属性本身不能被重新赋值 并不是说它的内部内容不能被更改





## 索引签名

有时候你不知道类型的属性名称，但是你知道值的形状。

在这种情况下，你可以使用索引签名来描述可能的值的类型

```ts
interface StringArray {
  [index: number]: string;
}
 
const myArray: StringArray = getStringArray();
const secondItem = myArray[1];
```

索引签名属性只允许一些特定的类型：字符串、数字、符号、模板字符串模式以及只包含这些类型的联合类型



字符串索引签名要求所有的属性类型与索引签名的返回类型匹配，这是因为字符串索引签名允许使用对象的属性名来访问属性，例如obj.property和obj["property"]是等价的

```ts
interface NumberDictionary {
  [index: string]: number;
 
  length: number; // 正确
  name: string; // error
}
```

```ts
interface NumberOrStringDictionary {
  [index: string]: number | string;
  length: number; // 正确，length是一个数字
  name: string; // 正确，name是一个字符串
}
```

可以将索引签名设置为只读，以防止对其索引进行赋值

```ts
interface ReadonlyStringArray {
  readonly [index: number]: string;
}
 
let myArray: ReadonlyStringArray = getReadOnlyStringArray();
myArray[2] = "Mallory"; // error
```



## 过度属性检查

对象字面量在赋值给其他变量或作为参数传递时会接受特殊处理，并进行过度属性检查

如果对象字面量具有目标类型不存在的任何属性，你将会得到一个错误提示

解决方法一  --- 类型断言

解决方法二 --- 为类型添加索引类型 以声明可以存在多余属性

```ts
interface SquareConfig {
  color?: string;
  width?: number;
  [propName: string]: any;
}
```

解决方法三 --- 字面量赋值给变量后再进行传递 以绕过过度属性检测

1. 当直接将字面量类型赋值给变量或作为函数参数传递时，会触发过度属性检查

2. 当将字面量类型赋值给另一个变量，并且这个变量的类型中存在与字面量类型中的属性相匹配的属性时，可以绕过过度属性检查

   ```ts
   let squareOptions = { colour: "red", width: 100 };
   let mySquare = createSquare(squareOptions);
   ```

3. 如果变量的类型中没有任何一个属性与字面量类型中的属性相匹配，那么仍然会触发过度属性检查

   ```ts
   let squareOptions = { colour: "red" };
   let mySquare = createSquare(squareOptions); // error
   ```

   

## 扩展类型

在编程中，经常会有一些类型是其他类型的更具体的版本。

通过在接口上使用extends关键字，我们可以有效地从其他已命名类型中复制成员，并添加任何我们想要的新成员

这可以减少我们需要编写的类型声明冗余，并且可以表明同一属性的多个不同声明可能是相关的（对于使用者）

```ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

// 支持多继承
interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```



## 交集类型（Intersection Types）

使用 & 运算符定义交集类型

交集类型（intersection types）的构造，主要用于组合现有的对象类型。

```ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}

type ColorfulCircle = Colorful & Circle;
```



### 接口和交叉类型的区别

Type 类型的 Box 是一种其内容具有 Type 类型的东西

 Type 是将被某种其他类型替换的占位符

我们可以完全避免使用重载，而是使用泛型函数



## Arrray类型

每当我们编写像 number[] 或 string[] 这样的类型时，实际上只是` Array<number>` 和 `Array<string>` 的简写

现代 JavaScript 还提供了其他泛型数据结构，比如` Map<K, V>、Set<T> 和 Promise<T>`。这只是意味着，由于 Map、Set 和 Promise 的行为方式，它们可以与任何一组类型一起使用



### ReadonlyArray 类型

ReadonlyArray 是一种特殊类型，用于描述不应该被更改的数组。

与 Array 不同，没有可以用于 ReadonlyArray 的构造函数

相反，我们可以将常规的数组分配给 ReadonlyArray。

正如 TypeScript 为 `Array<Type>` 提供了 Type[] 的简写语法一样，它也为` ReadonlyArray<Type> `提供了 readonly Type[] 的简写语法。

最后需要注意的一点是，与属性的 readonly 修饰符不同，常规数组和 ReadonlyArray 之间的可赋值性不是双向的。

```ts
let x: readonly string[] = [];
let y: string[] = [];

x = y; // 错误
y = x;
```



## 元组类型

元组类型是另一种数组类型，它知道它包含多少个元素，以及在特定位置包含哪些类型的元素。

与 ReadonlyArray 类似，它在运行时没有表示，但对 TypeScript 非常重要

```ts
type StringNumberPair = [string, number];
```

对于类型系统来说，StringNumberPair 描述了索引 0 包含字符串，索引 1 包含数字的数组

元组类型在基于约定的 API 中非常有用，其中每个元素的含义是“明显的”。这使得我们可以在解构它们时使用任何我们想要的变量名称



元组可以通过在元素类型后面写问号（?）来具有可选属性。可选元组元素只能出现在末尾，并且还会影响 length 的类型。

对应的length 会变成 number字面量 联合类型 如 2 | 3



元组还可以具有 rest 元素，其必须是数组/元组类型。

带有 rest 元素的元组没有设置“length” - 它只有不同位置的一组已知元素。

```ts
type StringNumberBooleans = [string, number, ...boolean[]];
type StringBooleansNumber = [string, ...boolean[], number];
type BooleansStringNumber = [...boolean[], string, number];
```





为什么可选元素和 rest 元素可能有用呢？这允许 TypeScript 将元组与参数列表对应起来。元组类型可以在剩余参数和参数中使用，因此以下代码：

```ts
function readButtonInput(...args: [string, number, ...boolean[]]) {
  const [name, version, ...input] = args;
  // ...
}

// 等价于

function readButtonInput(name: string, version: number, ...input: boolean[]) {
  // ...
}
```



### 只读元组

元组类型有只读变体，可以通过在它们前面添加 readonly 修饰符来指定 - 就像使用数组简写语法一样

在大多数代码中，元组往往会被创建并保持不变，因此在可能的情况下将类型标记为只读元组是一个很好的默认选择

```ts
function doSomething(pair: readonly [string, number]) {
  // ...
}
```



这也很重要，因为使用 const 断言的数组字面量将被推断为只读元组类型

```ts
let point = [3, 4] as const;

function distanceFromOrigin([x, y]: [number, number]) {
  return Math.sqrt(x ** 2 + y ** 2);
}

distanceFromOrigin(point); // error --- 只读元组 不可以赋值给 可写元组
```













---

与属性的 readonly 修饰符不同，常规数组和 ReadonlyArray 之间的可赋值性不是双向的。

接口和交叉类型的区别



除了长度检查之外，这些简单的元组类型与声明特定索引的属性和使用数字字面类型声明长度的数组版本等价。

```ts
interface StringNumberPair {
  // 专用属性
  length: 2;
  0: string;
  1: number;

  // 其他 'Array<string | number>' 成员...
  slice(start?: number, end?: number): Array<string | number>;
}
```