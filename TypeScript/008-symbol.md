```ts
let x:symbol = Symbol();
```



`symbol`类型包含所有的 Symbol 值

`unique symbol` 类型表示单个的、某个具体的 Symbol 值 

可以认为`unique symbol`是`symbol`的字面量值，所以`unique symbol`是`symbol`的子类型



```ts
// 正确
const x:unique symbol = Symbol();

// 报错
let y:unique symbol = Symbol();
```



`const`命令为变量赋值 Symbol 值时，变量类型默认就是`unique symbol`，所以类型可以省略不写。

```ts
const x:unique symbol = Symbol();
// 等同于
const x = Symbol();
```