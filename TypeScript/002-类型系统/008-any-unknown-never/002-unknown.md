## unknown

unknown --- `严格版any`

1. unknown 是 顶层类型 --- 是其余任意类型的父类型 -- unknown 只能赋值给 unknown类型 或 any类型

   ```ts
   let x:unknown;
   
   // 父类型 -> 子类型 success
   x = true; // 正确
   x = 42; // 正确
   x = 'Hello World'; // 正确
   ```

   相应的，任何其余类型都不可以直接赋值给`unknown`类型

   ```ts
   let v:unknown = 123;
   
   let v1:boolean = v; // 报错
   let v2:number = v; // 报错
   ```

2. 不能直接调用`unknown`类型变量的方法和属性

   ```ts
   let v1:unknown = { foo: 123 };
   v1.foo  // 报错
   ```

   可以和 unknown 一起使用的运算符，也只有那些类型缩小运算符

   ```ts
   let a:unknown = 1;
   
   a + 1 // 报错
   a === 1 // 正确
   ```

   在使用unknown之前，必须通过类型缩小，将其缩小为更具体的类型

   ```ts
   let a:unknown = 1;
   
   if (typeof a === 'number') {
     let r = a + 10; // 正确
   }
   ```

   > 1. “类型缩小”，即将一个不确定的类型缩小为更明确的类型，可以简单理解为 类型缩小就是 父类型 转变为 子类型
   >
   > 2. 一般来说，凡是需要设为`any`类型的地方，通常都应该优先考虑设为`unknown`类型

