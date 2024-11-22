`symbol`表示的是`symbol`类型

`unique symbol`是 `symbol` 的子类型，表示一个唯一的、不可变的符号值



因为 `unique symbol` 必须是唯一且不可变的，所以必须使用 `const` 关键字声明。

```ts
const x: unique symbol = Symbol(); // 正确
let y: unique symbol = Symbol();   // 报错
```

用 `const` 声明的 Symbol 值类型默认为 `unique symbol`。

```typescript
const x = Symbol(); // 类型为 unique symbol
```



`Symbol.for()`用于在全局注册表中查找或创建符号。

`Symbol.for()` 方法返回相同的 Symbol 值，TypeScript 不能识别这种情况。

```ts
const a: unique symbol = Symbol.for('foo'); // 报错
const b: typeof a = Symbol.for('foo'); // 报错 => 运行时相等，但类型系统中视为不同类型
```



## ==应用场景==

- **作为对象的属性名**：使用 `unique symbol` 可以确保属性名不与其他属性冲突。

  ```typescript
  const x: unique symbol = Symbol();
  
  interface Foo {
    [x]: string; // 使用 unique symbol 作为属性名
  }
  ```

- **作为类的 `readonly static` 属性**：在类中定义静态且只读的属性。

  ```typescript
  class C {
    static readonly foo: unique symbol = Symbol();
  }
  ```

