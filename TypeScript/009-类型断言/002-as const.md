`A as const`是一种特殊断言

1. 将A的类型断言为不可变类型，缩小成 TypeScript 允许的最小类型
2. `as const`本质依旧是类型推导 --- 如果存在显示声明的类型，以声明类型为准

```ts
let s1 = 'JavaScript' as const; // type s -> JavaScript
let s2: string = 'JavaScript' as const; // type s -> string
```



`A as const`中，A只能是字面量，不能是变量和JS表达式

```ts
let s = ('Java' + 'Script') as const; // 报错
```



`as const` 断言可以用于整个对象，也可以用于对象的单个属性

```ts
const v1 = { x: 1, y: 2 }; // 类型是 { x: number; y: number; }

const v2 = { x: 1 as const, y: 2 }; // 类型是 { x: 1; y: number; }

const v3 = { x: 1, y: 2 } as const; // 类型是 { readonly x: 1; readonly y: 2; }
```



```ts
enum Foo {
  X,
  Y,
}
let e1 = Foo.X;            // Foo => 等价于 Foo.X | Foo.Y 等价于 0 | 1
let e2 = Foo.X as const;   // Foo.X => 等价于 0
```

