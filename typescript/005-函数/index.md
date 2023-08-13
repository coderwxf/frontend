函数在应用程序中是基本的构建块，无论是本地函数、从其他模块导入的函数，还是类的方法。它们也是值，就像其他值一样，TypeScript 有许多方法来描述函数的调用方式。让我们学习如何编写描述函数的类型。



## 函数类型表达式

**==描述函数的最简单方式是使用函数类型表达式。这些类型在语法上类似于箭头函数：==**

```typescript
function greeter(fn: (a: string) => void) {
  fn("Hello, World");
}

function printToConsole(s: string) {
  console.log(s);
}

greeter(printToConsole);
```

==**上述代码中的 `(a: string) => void` 表示“一个具有一个名为 `a` 的参数，类型为 `string`，没有返回值的函数”。就像函数声明一样，如果没有指定参数类型，它会隐式地被推断为 `any` 类型。**==

==**请注意，参数名是必需的。函数类型 `(string) => void` 意味着“一个具有名为 `string` 类型为 `any` 的参数的函数”！**==

当然，我们可以使用类型别名来命名函数类型：

```typescript
type GreetFunction = (a: string) => void;
function greeter(fn: GreetFunction) {
  // ...
}
```

以上代码中的 `GreetFunction` 类型别名表示了函数类型 `(a: string) => void`。



## 调用签名

**==在 JavaScript 中，函数除了可调用之外，还可以具有属性。然而，函数类型表达式语法不允许声明属性。如果我们想描述一个既可调用又具有属性的内容，可以在对象类型中编写一个调用签名：==**

```typescript
type DescribableFunction = {
  description: string;
  (someArg: number): boolean;
};

function doSomething(fn: DescribableFunction) {
  console.log(fn.description + " returned " + fn(6));
}

function myFunc(someArg: number) {
  return someArg > 3;
}
myFunc.description = "default description";

doSomething(myFunc);
```

**==请注意，与函数类型表达式相比，语法略有不同 - 在参数列表和返回类型之间使用 `:` 而不是 `=>`。==**



## 构造函数

**==JavaScript函数也可以使用new操作符调用。TypeScript将这些称为构造函数，因为它们通常创建一个新对象。您可以通过在调用签名前面添加new关键字来编写一个构造签名：==**

```typescript
type SomeConstructor = {
  new (s: string): SomeObject;
};
function fn(ctor: SomeConstructor) {
  return new ctor("hello");
}
```


==**有些对象，比如JavaScript的Date对象，可以使用new或者不使用new调用。您可以在同一类型中任意组合调用和构造签名：==**

```typescript
interface CallOrConstruct {
  new (s: string): Date;
  (n?: number): string;
}
```



## 泛型函数

**==通常我们会编写一个函数，其中输入的类型与输出的类型有关，或者两个输入的类型以某种方式相关==**。让我们来考虑一个函数，它返回数组的第一个元素：

```typescript
function firstElement(arr: any[]) {
  return arr[0];
}
```


这个函数可以正常工作，但不幸的是返回类型是any。如果函数能返回数组元素的类型会更好。

**==在TypeScript中，泛型用于描述两个值之间的对应关系。我们可以通过在函数签名中声明一个类型参数来实现：==**

```typescript
function firstElement<Type>(arr: Type[]): Type | undefined {
  return arr[0];
}
```


通过在这个函数中添加一个类型参数Type，并在两个地方使用它，我们创建了函数输入（数组）和输出（返回值）之间的关联。现在当我们调用它时，会得到一个更具体的类型：

```typescript
// s 的类型为 'string'
const s = firstElement(["a", "b", "c"]);
// n 的类型为 'number'
const n = firstElement([1, 2, 3]);
// u 的类型为 undefined
const u = firstElement([]);
```



### 类型推断

==**请注意，我们在这个示例中没有必须指定Type。TypeScript会自动推断（选择）类型。**==

==**我们也可以使用多个类型参数**==。例如，map函数的独立版本如下所示：

```typescript
function map<Input, Output>(arr: Input[], func: (arg: Input) => Output): Output[] {
  return arr.map(func);
}

// 参数 'n' 的类型为 'string'
// 'parsed' 的类型为 'number[]'
const parsed = map(["1", "2", "3"], (n) => parseInt(n));
```


请注意，在这个示例中，TypeScript可以根据给定的字符串数组推断出Input类型参数的类型，也可以根据函数表达式的返回值（number）推断出Output类型参数。



### 类型约束

我们已经编写了一些可以处理任何类型的泛型函数。**==有时我们希望关联两个值，但只能操作某个特定类型的值。在这种情况下，我们可以使用约束来限制类型参数可以接受的类型范围。==**

让我们编写一个函数，返回两个值中较长的值。为了做到这一点，我们需要一个具有数字类型的length属性。我们通过编写一个extends子句将类型参数限制为该类型：

```typescript
function longest<Type extends { length: number }>(a: Type, b: Type) {
  if (a.length >= b.length) {
    return a;
  } else {
    return b;
  }
}

// longerArray 的类型为 'number[]'
const longerArray = longest([1, 2], [1, 2, 3]);
// longerString 的类型为 'alice' | 'bob'
const longerString = longest("alice", "bob");
// 错误！数字没有 'length' 属性
const notOK = longest(10, 100);
```


这个示例中有几个有趣的地方。**==我们允许TypeScript推断longest的返回类型。返回类型推断也适用于泛型函数==**。

因为我们将Type限制为{ length: number }类型，所以我们可以访问a和b参数的.length属性。如果没有类型约束，我们将无法访问这些属性，因为这些值可能是没有length属性的其他类型。

longerArray和longerString的类型是基于参数推断的。记住，泛型就是关联两个或多个具有相同类型的值！

最后，正如我们所希望的那样，调用longest(10, 100)被拒绝，因为number类型没有.length属性。



### 使用约束值

以下是使用通用约束时常见的错误：

```typescript
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum };
    // Type '{ length: number; }' is not assignable to type 'Type'.
    // '{ length: number; }' is assignable to the constraint of type 'Type',
    // but 'Type' could be instantiated with a different subtype of constraint '{ length: number; }'.
  }
}
```

这个函数看起来是没问题的 - `Type` 被约束为 `{ length: number }`，函数要么返回 `Type`，要么返回与约束相匹配的值。**==问题在于该函数承诺返回与传入的对象相同类型的对象，而不仅仅是与约束匹配的对象==**。如果这段代码是合法的，你可能会写出肯定不起作用的代码：

```typescript
// 'arr'得到值 { length: 6 }
const arr = minimumLength([1, 2, 3], 6);
// 然后在这里崩溃，因为数组有'slice'方法，但返回的对象没有！
console.log(arr.slice(0));
```

这段代码尝试使用 `slice` 方法对 `arr` 进行切片，但是返回的对象却没有 `slice` 方法，导致代码崩溃。

这个问题的根本原因是，在 `minimumLength` 函数中，我们并没有根据传入的对象类型构造一个新的对象，而是直接返回了一个与约束匹配的对象字面量。==**在 TypeScript 中，对象字面量是一种特殊的类型，无法与泛型类型参数 `Type` 相匹配。**==

要解决这个问题，我们需要使用类型断言来告诉编译器返回的对象类型与传入的对象类型相同。修改后的代码如下：

```typescript
function minimumLength<Type extends { length: number }>(
  obj: Type,
  minimum: number
): Type {
  if (obj.length >= minimum) {
    return obj;
  } else {
    return { length: minimum } as Type;
  }
}
```

现在，我们使用类型断言 `as Type` 来确保返回的对象类型与传入的对象类型相同。这样，代码就不会再报类型不匹配的错误了。

但需要注意的是，使用类型断言时需要确保类型断言是安全的。在这种情况下，我们假设返回的对象只是缺少了一些属性，但其他属性与传入的对象是相同的。如果你不确定类型断言是否安全，最好进行额外的检查和测试，以确保代码的正确性。



### 指定参数类型

==**TypeScript通常可以推断出泛型调用中的预期类型参数，但并非总是如此**==。例如，假设你编写了一个用于合并两个数组的函数：

```typescript
function combine<Type>(arr1: Type[], arr2: Type[]): Type[] {
  return arr1.concat(arr2);
}
```

通常情况下，使用不匹配的数组调用该函数会导致错误：

```typescript
const arr = combine([1, 2, 3], ["hello"]);
// Type 'string' is not assignable to type 'number'.
```

然而，如果你打算这样做，你可以手动指定类型参数 `Type`：

```typescript
const arr = combine<string | number>([1, 2, 3], ["hello"]);
```

通过在尖括号中明确指定类型参数 `<string | number>`，你告诉 TypeScript 在调用 `combine` 函数时使用这个特定的类型。这样，你可以将不同类型的数组传递给函数，并且 TypeScript 不会报类型不匹配的错误。

手动指定类型参数可以提供更精确的类型控制，但需要注意确保所指定的类型是符合实际情况的。如果你不确定应该使用哪个类型参数，可以根据具体的需求进行测试和调试，以确保代码的正确性。



### 推荐规范

编写通用函数是一件有趣的事情，但很容易过度使用类型参数。过多的类型参数或在不必要的情况下使用约束可能会导致推断失败，让函数的调用者感到沮丧。



#### 将类型参数下推

下面是两种看似相似的函数写法：

```typescript
function firstElement1<Type>(arr: Type[]) {
  return arr[0];
}

function firstElement2<Type extends any[]>(arr: Type) {
  return arr[0];
}
 
// a: number (good)
const a = firstElement1([1, 2, 3]);
// b: any (bad)
const b = firstElement2([1, 2, 3]);
```

乍一看，这两个函数似乎是相同的，但是 `firstElement1` 是一个更好的编写方式。它的推断返回类型是 `Type`，而 `firstElement2` 的推断返回类型是 `any`，因为 TypeScript 必须使用约束类型来解析 `arr[0]` 表达式，而不是在调用时“等待”解析元素。

规则==：**尽可能使用类型参数本身，而不是约束类型**。==



#### 尽可能少用泛型参数

这里还有一对类似的函数：

```typescript
function filter1<Type>(arr: Type[], func: (arg: Type) => boolean): Type[] {
  return arr.filter(func);
}

function filter2<Type, Func extends (arg: Type) => boolean>(
  arr: Type[],
  func: Func
): Type[] {
  return arr.filter(func);
}
```

我们创建了一个类型参数 `Func`，它并没有关联两个值。这总是一个警示信号，因为这意味着想要指定类型参数的调用者必须无缘无故手动指定额外的类型参数。`Func` 没有做任何事情，只会让函数变得更难阅读和理解！

规则：**==尽可能使用较少的类型参数。==**



#### 类型参数必须出现两次及以上

有时候我们会忘记一个函数可能不需要是泛型的：

```typescript
function greet<Str extends string>(s: Str) {
  console.log("Hello, " + s);
}
 
greet("world");
```

我们同样可以写一个更简单的版本：

```typescript
function greet(s: string) {
  console.log("Hello, " + s);
}
```

记住，类型参数用于关联多个值的类型。如果一个类型参数在函数签名中只使用一次，那它就没有起到关联的作用。这还包括推断的返回类型；例如，如果 `Str` 是 `greet` 的推断返回类型的一部分，那么它将关联参数和返回类型，因此尽管在代码中只出现一次，但会被使用两次。

规则：**==如果一个类型参数只出现在一个位置，请仔细考虑是否实际上需要它==**。



## 可选参数

JavaScript中的函数通常可以接受可变数量的参数。例如，数字的toFixed方法接受一个可选的数字位数参数：

```javascript
function f(n) {
  console.log(n.toFixed()); // 0个参数
  console.log(n.toFixed(3)); // 1个参数
}
```

**==在TypeScript中，我们可以通过在参数后加上?来将参数标记为可选的==**：

```typescript
function f(x?: number) {
  // ...
}
```

调用时，可以省略可选参数：

```typescript
f(); // OK
f(10); // OK
```


**==尽管参数的类型被指定为number，但是x参数实际上会有number | undefined类型，因为在JavaScript中未指定的参数的值为undefined。==**

**==您还可以提供参数的默认值：==**

```typescript
function f(x = 10) {
  // ...
}
```

现在，在f函数的函数体中，x将具有number类型，因为任何未定义的参数都将被替换为10。请注意，当参数是可选的时，调用者始终可以传递undefined，因为这只是模拟了一个"缺失"的参数：

```typescript
declare function f(x?: number): void;
// 省略部分
// 以下都是合法的调用
f();
f(10);
f(undefined);
```



### 回调函数中的可选参数

一旦您了解了可选参数和函数类型表达式，当编写调用回调函数的函数时，很容易犯以下错误：

```typescript
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    callback(arr[i], i);
  }
}
```

当编写index?作为可选参数时，人们通常意图是希望以下两个调用都是合法的：

```typescript
myForEach([1, 2, 3], (a) => console.log(a));
myForEach([1, 2, 3], (a, i) => console.log(a, i));
```

然而，实际上这意味着callback可能会以一个参数调用。换句话说，函数定义表示实现可能如下：

```typescript
function myForEach(arr: any[], callback: (arg: any, index?: number) => void) {
  for (let i = 0; i < arr.length; i++) {
    // 今天我不想提供索引
    callback(arr[i]);
  }
}
```

而TypeScript将强制执行这个含义，并产生一些实际上不可能出现的错误：

```typescript
myForEach([1, 2, 3], (a, i) => {
  console.log(i.toFixed());
  // 'i' 可能为 'undefined'
});
```

在JavaScript中，如果您调用的函数的参数比定义的参数多，额外的参数将被忽略。TypeScript的行为方式也相同。具有较少参数（类型相同）的函数总是可以替代具有更多参数的函数。

原则：在为回调函数编写函数类型时，除非您打算在调用函数时不传递该参数，否则不要编写可选参数。















---

签名