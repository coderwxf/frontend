## any

any 类型表示没有任何限制，该类型的变量可以赋予任意类型的值。

变量类型一旦设为`any`，TypeScript 实际上会关闭这个变量的类型检查。

即使有明显的类型错误，只要句法正确，都不会报错。

因此any可以看成是顶层类型，即所有类型的父类型，任意类型都可以赋值给any类型

```ts
let x:any;

x = 1; // 正确
x = 'foo'; // 正确
x = true; // 正确
```



由于这个原因，应该尽量避免使用`any`类型，否则就失去了使用 TypeScript 的意义。

实际开发中，`any`类型主要适用于为了适配以前老的 JavaScript 项目，让代码快速迁移到 TypeScript，可以把变量类型设为`any`



对于开发者没有指定类型、TypeScript 必须自己推断类型的那些变量，如果无法推断出类型，TypeScript 就会认为该变量的类型是`any`。

这显然是很糟糕的情况，所以TypeScript 提供了一个编译选项`noImplicitAny`，打开该选项，只要推断出`any`类型就会报错。



这里有一个特殊情况，即使打开了`noImplicitAny`，使用`let`和`var`命令声明变量，但不赋值也不指定类型，是不会报错的。

```ts
var x; // 不报错
let y; // 不报错
```

上面示例中，变量`x`和`y`声明时没有赋值，也没有指定类型，TypeScript 会推断它们的类型为`any`。这时即使打开了`noImplicitAny`，也不会报错

```ts
let x;

x = 123;
x = { foo: 'hello' };
```

由于这个原因，建议使用`let`和`var`声明变量时，如果不赋值，就一定要显式声明类型或赋予初始值，否则可能存在安全隐患。



`any`类型除了关闭类型检查，还有一个很大的问题，就是它会“污染”其他变量。

它可以赋值给其他任何类型的变量（因为没有类型检查），导致其他变量出错

```ts
let x:any = 'hello';
let y:number;

y = x; // 不报错 -- y的类型依旧是number，但是因为值是x，即any所以关闭了后续所有的类型检测

y * 123 // 不报错
y.toFixed() // 不报错
```



## unknown

为了解决`any`类型“污染”其他变量的问题，TypeScript引入了`unknown`

所有类型的值都可以分配给`unknown`类型

```ts
let x: unknown;

// 子类型可以赋值给父类型
x = true; // 正确
x = 42; // 正确
x = 'Hello World'; // 正确
```

`unknown`类型跟`any`类型的不同之处在于，

1. `unknown`类型的变量，不能直接赋值给其他类型的变量（除了`any`类型和`unknown`类型）。

   ```ts
   let v:unknown = 123;
   
   // 父类型不能赋值给子类型
   let v1:boolean = v; // 报错
   let v2:number = v; // 报错
   ```

   

2. 它必须经过类型缩小后才可以被使用，可以视为严格版的`any`。

   ```ts
   let v1:unknown = { foo: 123 };
   v1.foo  // 报错
   
   let v2:unknown = 'hello';
   v2.trim() // 报错
   
   let v3:unknown = (n = 0) => n + 1;
   v3() // 报错
   ```

   

总之，`unknown`可以看作是更安全的`any`。一般来说，凡是需要设为`any`类型的地方，通常都应该优先考虑设为`unknown`类型。

在集合论上，`unknown`也可以视为所有其他类型（除了`any`）的全集，所以它和`any`一样，也属于 TypeScript 的顶层类型。



## never

由于不存在任何属于“空类型”的值，所以该类型被称为`never`，即不可能有这样的值

never是最底层类型，所以never类型值，不可能赋给它任何值，否则都会报错



`never`类型的使用场景，主要是在一些类型运算之中，保证类型运算的完整性

```ts
function fn(x:string|number) {
  if (typeof x === 'string') {
    // ...
  } else if (typeof x === 'number') {
    // ...
  } else {
    x; // never 类型
  }
}
```

另外，不可能返回值的函数，返回值的类型就可以写成`never` 「 如抛出异常的函数和死循环函数 」

```ts
function InfiniteFn: never() {
  while (true) {}
}
```



never是最底层类型，所以never是其余任意类型的子类型，可以赋值给其余任意类型

```ts
function f():never {
  throw new Error('Error');
}

let v1:number = f(); // 不报错
let v2:string = f(); // 不报错
let v3:boolean = f(); // 不报错
```

