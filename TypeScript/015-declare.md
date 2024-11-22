`declare` 用于告诉 TypeScript 编译器某些变量、函数、类、模块或枚举是由外部代码定义的。

它本质是一种特殊的类型声明「 类型断言 」，所以`declare`不会在生成的 JavaScript 中出现。



```ts
declare const myVariable: string;
```

该文件中没有定义字符串类型值`myVariable`

但引入的模块或全局一定存在这个变量，可以直接用



declare的一个常用功能是 **模块声明**

为外部 JavaScript 库定义类型。

```typescript
declare module "myModule" {
  export function myExportedFunction(): void;
}
```

#### 示例：

```typescript
declare module 'io' {
  export function readFile(filename: string): string;
}
```



declare 也可以实现类型扩展

```ts
import type { Foo as Bar } from './foo';

declare module './foo' {
  // 本地模块foo中，叫Foo，不是Bar
  interface Foo {
    y: number;
  }
}

let bar: Bar = {
  x: 1,
  y: 2
};
```

```ts
export {};

// 使用 `declare global` 来扩展全局对象。
declare global {
  interface String {
    toSmallString(): string;
  }
}

String.prototype.toSmallString = function (): string {
  return this.toLowerCase();
};
```

