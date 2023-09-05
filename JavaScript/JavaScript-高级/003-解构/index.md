ES6 允许按照一定模式，从数组和对象中提取值 并赋值给变量的行为, 被称为解构（Destructuring）

解构依赖的是 “模式匹配”，只要等号两边的模式相同，左边的变量就会被赋予对应的值



## 数组

```js
let [a, b, c] = [1, 2, 3];

let [foo, [[bar], baz]] = [1, [[2], 3]];
foo // 1
bar // 2
baz // 3

let [ , , third] = ["foo", "bar", "baz"];
third // "baz"

let [x, , y] = [1, 2, 3];
x // 1
y // 3

let [head, ...tail] = [1, 2, 3, 4];
head // 1
tail // [2, 3, 4]

let [x, y, ...z] = ['a'];
x // "a"
y // undefined
z // []
```



如果解构不成功，变量的值就等于`undefined`。

以下两种情况都属于解构不成功，`foo`的值都会等于`undefined`

```js
let [foo] = [];
let [bar, foo] = [1];
```



另一种情况是不完全解构，即等号左边的模式，只匹配一部分的等号右边的数组。这种情况下，解构依然可以成功。

```ts
let [x, y] = [1, 2, 3];
x // 1
y // 2

let [a, [b], d] = [1, [2, 3], 4];
a // 1
b // 2
d // 4
```



只要某种数据结构具有 Iterator 接口，都可以采用数组形式的解构赋值

```ts
let [x, y, z] = new Set(['a', 'b', 'c']);
x // "a"
```

```ts
function* fibs() {
  let a = 0;
  let b = 1;
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

let [first, second, third, fourth, fifth, sixth] = fibs();
sixth // 5
```

```js
// 报错
let [foo] = 1;
let [foo] = false;
let [foo] = NaN;
let [foo] = undefined;
let [foo] = null;
let [foo] = {};
```



### 默认值

解构赋值允许指定默认值。

只有当一个数组成员严格等于`undefined`，默认值才会生效。

```ts
let [x = 1] = [undefined];
x // 1

let [x = 1] = [null];
x // null
```



如果默认值是一个表达式，那么这个表达式是惰性求值的，即只有在用到的时候，才会求值。

```js
function f() {
  console.log('aaa');
}

// 因为x能取到值，所以函数f根本不会执行
let [x = f()] = [1];
```



默认值可以引用解构赋值的其他变量，但该变量必须已经声明。

```ts
let [x = 1, y = x] = [];     // x=1; y=1
let [x = 1, y = x] = [2];    // x=2; y=2
let [x = 1, y = x] = [1, 2]; // x=1; y=2
let [x = y, y = 1] = [];     // ReferenceError: y is not defined
```



## 对象

解构不仅可以用于数组，还可以用于对象。

对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定

而对象的属性没有次序，变量必须与属性同名，才能取到正确的值

如果解构失败，变量的值等于`undefined`。

```ts
let { bar, foo } = { foo: 'aaa', bar: 'bbb' };
foo // "aaa"
bar // "bbb"

let { baz } = { foo: 'aaa', bar: 'bbb' };
baz // undefined
```



