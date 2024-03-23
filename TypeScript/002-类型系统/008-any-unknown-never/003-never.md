## never

1. never表示不可能存在的类型
2. never是底层类型，也就是所有类型的子类型



作用

1. 不可能返回值的函数，返回值的类型就可以写成`never`

   + 死循环函数
   + 抛出异常的函数

2. 联合类型缩小时，放于最后，用于确保类型运算的完整性

   ```ts
   function fn(x:string|number) {
     if (typeof x === 'string') {
       // ...
     } else if (typeof x === 'number') {
       // ...
     } else {
       x; // never 类型
     }
   }
   ```



never作为底层类型，可以赋值给任意类型变量

```ts
function f():never {
  throw new Error('Error');
}

let v1:number = f(); // 不报错
let v2:string = f(); // 不报错
let v3:boolean = f(); // 不报错
```

