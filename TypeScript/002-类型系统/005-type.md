`type`命令用来定义一个类型的别名

1. 使类型 更加具有 语义化
2. 简化 复杂类型的使用方式

```ts
type Age = number;

let age: Age = 55;
```



类型别名不允许重复

```ts
type Color = 'red';
type Color = 'blue'; // 报错
```



别名的作用域和变量一致，都是块级作用域

```ts
type Color = 'red';

if (Math.random() < 0.5) {
  type Color = 'blue';
  
  let color: Color = 'red' // error --- color的类型是 字面量类型 blue 不是 red
}
```



别名支持类型表达式，即通过模板字符串引用另一个别名

```ts
type World = "world";
type Greeting = `hello ${World}`;
```

