类型别名用于为类型创建一个新的名称。它在编译时会被删除，不影响运行时。

```ts
type Age = number;

let age: Age = 55;
```

**解释**：`Age` 是 `number` 类型的别名，可以用于声明变量。



类型别名不能在同一作用域内重复声明。

```ts
type Color = 'red';
type Color = 'blue'; // 报错
```

**解释**：`Color` 已经被声明为 `'red'`，不能再次声明为 `'blue'`。



类型别名是块级作用域的，这意味着它们在不同的块中可以有相同的名称，但这通常不推荐。

```ts
type Color = 'red';

if (Math.random() < 0.5) {
  type Color = 'blue'; // 不推荐
}
```



类型别名可以嵌套使用，即一个别名可以使用另一个别名。

```ts
type World = "world";
type Greeting = `hello ${World}`;
```

**解释**：`Greeting` 是一个模板字符串类型，使用了 `World` 别名。



## interface vs type

1. **表示类型的范围**：

   - `type` 可以表示任何类型，包括基本类型、联合类型、交叉类型等。
   - `interface` 主要用于表示对象类型。

2. **继承与扩展**：

   - `interface` 可以通过 `extends` 继承其他接口。
   - `type` 不能直接继承，但可以通过交叉类型 `&` 实现类似效果。

3. **同名合并**：

   - `interface` 支持同名接口自动合并。
   - `type` 不允许同名重复定义，会报错。

4. **属性映射**：

   - `type` 支持映射类型。
   - `interface` 不支持映射类型。

   ```ts
   interface Point {
     x: number;
     y: number;
   }
   
   // 正确
   type PointCopy1 = {
     [Key in keyof Point]: Point[Key];
   };
   
   // 报错
   interface PointCopy2 {
     [Key in keyof Point]: Point[Key];
   };
   ```

5. **`this` 类型**：

   - `interface` 可以使用 `this` 类型。
   - `type` 中使用 `this` 会报错。

   ```ts
   // 正确
   interface Foo {
     add(num: number): this;
   }
   
   // 报错
   type FooType = {
     add(num: number): this;
   };
   ```

6. **扩展原始类型**：

   - `type` 可以扩展原始数据类型。
   - `interface` 不能扩展原始数据类型。

   ```ts
   // 正确
   type MyStr = string & {
     type: 'new';
   };
   
   // 报错
   interface MyStr extends string {
     type: 'new';
   }
   ```

   

一般情况下，推荐优先使用接口，因为接口会自动合并。

但如果需要实现一些类型运算，优先使用 `type`。