定义对象结构有三种方式

1. type  --- 单纯只是定义类型结构
2. interface  --- 单纯只是定义类型结构
3. class -- 定义类的同时，定义了对应的类型结构



## type VS interface

1. type可以给非对象类型起别名，interface只能定义对象结构

2. interface通过继承扩展属性，type通过交叉类型扩展属性

3. 同名interface可以合并，同名type直接报错 「 interface 是开放的，可以添加属性，type 是封闭的，不能添加属性，只能定义新的 type 」

4. `interface`不能包含属性映射， type可以

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

5. Interface 中可以使用this，type不行

   ```ts
   // 正确
   interface Foo1 {
     add(num:number): this;
   };
   
   // 报错
   type Foo2 = {
     add(num:number): this;
   };
   ```

6. type 可以扩展原始数据类型，interface 不行

   ```ts
   // 正确
   type MyStr1 = string & {
     type: 'new'
   };
   
   // 报错
   interface MyStr2 extends string {
     type: 'new'
   }
   ```

7. `interface`无法表达某些复杂类型（比如交叉类型和联合类型），但是`type`可以

   ```ts
   type A = { /* ... */ };
   type B = { /* ... */ };
   
   type AorB = A | B;
   type AorBwithName = AorB & {
     name: string
   };
   ```

   