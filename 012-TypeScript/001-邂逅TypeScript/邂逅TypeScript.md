### 类型注解

TypeScript 不使用“类型在左侧”的声明方式，比如 int x = 0；类型注释总是会在被标记的对象之后，比如 x: number = 0;

```ts
// let|var|const 变量名: 类型注解 
let myName: string = "Klaus";
```

然而，在大多数情况下这并不是必需的。在任何可能的情况下，TypeScript 会尝试自动推断代码中的类型

```ts
// 变量的类型会根据其初始化值的类型进行类型推导
let myName= "Klaus";
```

