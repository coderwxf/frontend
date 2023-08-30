从ECMAScript 2015开始，符号（symbol）是一种原始数据类型，就像数字（number）和字符串（string）一样。

符号值通过调用Symbol构造函数创建

```ts
let sym1 = Symbol();
let sym2 = Symbol("key"); // optional string key
```



符号是不可变的（immutable）和唯一的（unique）

```ts
let sym2 = Symbol("key");
let sym3 = Symbol("key");
sym2 === sym3; // false, symbols are unique
```



与字符串类似，符号可以用作对象属性的键。

```javascript
const sym = Symbol();
let obj = {
  [sym]: "value",
};
console.log(obj[sym]); // "value"
```



符号还可以与计算属性声明结合使用，以声明对象属性和类成员。

```javascript
const getClassNameSymbol = Symbol();
class C {
  [getClassNameSymbol]() {
    return "C";
  }
}
let c = new C();
let className = c[getClassNameSymbol](); // "C"
```



