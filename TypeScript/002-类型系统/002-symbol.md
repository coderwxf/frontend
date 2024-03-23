`Symbol` 是一种基本数据类型，它代表了一个唯一且不可变的值

1. 通过`Symbol()`创建
2. 无法直接获取字面量值，只能通过变量来引用对应值

```ts
// symbol变量的类型 就是 symbol
let x:symbol = Symbol();
let y:symbol = Symbol();

console.log(x === y) // false
```



## unique symbol

`symbol`  => 所有的symbol值

`unique symbol` => 单个的、某个具体的 Symbol 值

> 1.  `symbol 和 unique symbol`之间的关联 就类似于 `string 和 'hello'`之间的关系
> 2. symbol必须通过变量引用，不存在symbol字面量，但可以将unique symbol类比为symbol字面量类型
> 3. `unique symbol `是`symbol`的子类型



`unique symbol`表示独一无二的symbol字面量，修改他的值没有意义，因此unique symbol对应变量必须被定义为常量

```ts
// 正确
const x:unique symbol = Symbol();

// 报错
let y:unique symbol = Symbol();
```



两个unique symbol属于两个不同的类型，想要获取同一类型的symbol只能通过typeof运算符

```ts
const a: unique symbol = Symbol();
const b: unique symbol = a; // 报错
const c: typeof a = a; // 正确
```



TS无法识别`symbol.for`创建的相同symbol值

```ts
const a:unique symbol = Symbol.for('foo');
const b:unique symbol = Symbol.for('foo');

const c: typeof a = b // error
```



### 作用

1. 对象key的类型必须是`unique symbol`，不能是`symbol`

   ```ts
   const x: unique symbol = Symbol();
   const y: symbol = Symbol();
   
   interface Foo {
     [x]: string; // 正确
     [y]: string; // 报错
   }
   ```

   

## 类型推断

1. 变量 --- symbol，常量 --- unique symbol

   ```ts
   let x = Symbol(); // 类型为 symbol
   
   const y = Symbol(); // 类型为 unique symbol
   ```

2. `symbol`赋值给常量, 推导为`symbol`

   ```ts
   let x = Symbol();
   
   const y = x; // 类型为 symbol --- 不是unique symbol的原因是 父类不能赋值给子类
   ```

3. `unique symbol`赋值给变量，推导为`symbol`

   ```ts
   const x = Symbol();
   
   let y = x; // 类型为 symbol --- 子类->父类 --- 类型兼容性
   ```

   