## Object

Object类型是一种复杂类型，也称为引用类型。

它与原始类型不同，原始类型表示的是单一的值，而Object类型可以包含多个值

Object类型将多种基本数据类型放到一个集合中

```js
let book = {
    name: "算法的力量",
    description: "人类如何共同生存",
    oldPrice: 98,
    newPrice: 73.5
};

let desc = 'description'
console.log(book.name); // "算法的力量"
console.log(book[desc]); // "人类如何共同生存"
console.log(book[oldPrice]); // 98
```



### 初始化

在开发中，我们有时需要初始化一个对象，而不确定其具体的值。

此时，我们可以将对象初始化为 `null`，而不是空对象 `{}`

主要原因如下

1. null所占内存和布尔值差不多，而空对象相对而言需要更多的内存空间
2. 如果我们使用空对象 `{}` 初始化，当我们进行逻辑判断时，它会被视为 `true`。而 `null` 在逻辑判断中会被视为 `false`。



## Null

- **Object类型** 用于表示复杂的数据结构，可以包含多个属性。
- **Null类型** 用于初始化对象为空。
- **Undefined类型** 表示变量已声明但尚未赋值。

005-