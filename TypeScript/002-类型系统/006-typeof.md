在JavaScript中

typeof 运算符是一个一元运算符，返回一个字符串，代表操作数的类型

typeof 运算符 一共可能存在八种情况

```js
typeof undefined; // "undefined"
typeof true; // "boolean"
typeof 1337; // "number"
typeof "foo"; // "string"
typeof {}; // "object" ---> typeof null => "object" 
typeof parseInt; // "function"
typeof Symbol(); // "symbol"
typeof 127n // "bigint"
```



在TypeScript中， typeof 用于获取当前值的类型

```ts
const a = { x: 0 };

type T0 = typeof a;   // { x: number }
type T1 = typeof a.x; // number
```



TS中的 typeof 返回的是类型，不是值 => 只能用于类型运算，不能用于 值运算

```ts
let a = 1;
let b: typeof a; // 这个typeof 是类型运算 --- 编译后会被类型擦除

if (typeof a === 'number') { // 这个typeof 是类型判断 --- 编译后依旧会保留
  b = a;
}
```

`编译后`

```js
let a = 1;
let b;
if (typeof a === 'number') {
    b = a;
}
```



TS是静态类型检测，不会涉及任何运行时值运算

```ts
// Date() 需要调用后，才可以知道类型，属于运行时值运算
type T = typeof Date(); // error
```



typeof 是 获取 值的类型

```ts
type Age = number;
type MyAge = typeof Age; // error --- typeof 获取的是 值的类型 --- 不是类型的类型
```

