**接口（Interface）** 是 TypeScript 中用于定义对象的类型结构的一种方式。多个属性之间可以使用换行符，逗号或分号进行分割。

- **普通属性**：接口中定义的属性默认是必需的。

  ```ts
  interface Person {
    firstName: string;
  }
  ```

- **可选属性**：在属性名后加上 `?` 表示该属性是可选的。

  ```ts
  interface Person {
    lastName?: string;
  }
  ```

- **只读属性**：使用 `readonly` 关键字定义的属性是只读的，不能被修改。

  ```ts
  interface Person {
    readonly age: number;
  }
  ```

- **索引签名**：允许对象有额外的属性，但这些属性必须符合指定的类型。

  ```ts
  interface Person {
    [prop: string]: number;
  }
  ```




## 属性类型

TypeScript 提供了一种方式可以通过方括号运算符获取属性的类型。

```ts
type Age = Person['age']; // 这里 Age 的类型是 number
```



## 定义方法

在接口中，可以有多种方式定义对象的方法：

- **直接定义**：

  ```ts
  interface A {
    f(x: boolean): string;
  }
  ```

- **使用函数类型**：

  ```ts
  interface B {
    f: (x: boolean) => string;
  }
  ```

- **使用对象字面量**：

  ```ts
  interface C {
    f: { (x: boolean): string };
  }
  ```




## 重载

接口中的方法可以进行重载，即同名方法可以有不同的参数类型和返回类型。

```ts
interface A {
  f(): number;
  f(x: boolean): boolean;
  f(x: string, y: string): string;
}
```



## 继承

接口可以通过继承（extends）来进行接口扩展

```ts
interface Shape {
  name: string;
}

interface Circle extends Shape {
  radius: number;
}
```

**解释**：`Circle` 接口继承了 `Shape` 接口，因此它包含 `name` 和 `radius` 两个属性。



TypeScript 允许接口实现多继承，这意味着一个接口可以同时继承多个接口。

```ts
interface Style {
  color: string;
}

interface Shape {
  name: string;
}

interface Circle extends Style, Shape {
  radius: number;
}
```

**解释**：`Circle` 接口继承了 `Style` 和 `Shape` 接口，因此它包含 `color`、`name` 和 `radius` 三个属性。



继承中出现的同名属性必须满足类型兼容性

```ts
interface Foo {
  id: string;
}

interface Bar extends Foo {
  id: number; // 报错
}
```

```ts
interface Foo {
  id: number | string;
}

interface Bar extends Foo {
  id: number; // 合法
}
```

```ts
interface Foo {
  id: string;
}

interface Bar {
  id: number;
}

interface Baz extends Foo, Bar {
  type: string;
} // 报错
```



## 继承自其他类型

1. **继承自类型别名**：

   类型别名如果是对象类型的别名，接口可以继承它。

2. **继承自类**：

   - 接口可以继承类的所有公共成员。

     ```ts
     class A {
       x: string = '';
     
       y(): boolean {
         return true;
       }
     }
     
     interface B extends A {
       z: number;
     }
     
     const b: B = {
       x: '',
       y: function() { return true; },
       z: 123
     }
     ```

     

   - 如果接口继承类的私有或受保护成员，则该接口不能直接用于对象实现，只能由类的子类实现。

     ```ts
     class A {
       private x: string = '';
       protected y: string = '';
     }
     
     interface B extends A {
       z: number;
     }
     
     // const b: B = { /* ... */ } // 报错
     
     class C extends A implements B {
       z = 2;
     }
     ```



## 接口合并

多个同名接口会自动合并成一个接口，属性会合并在一起。

```ts
interface Box {
  height: number;
  width: number;
}

interface Box {
  length: number;
}
```

**合并结果**：`Box` 接口包含 `height`、`width` 和 `length` 属性。



如果合并的接口中同名属性的类型冲突，会导致错误。

```ts
interface A {
  a: number;
}

interface A {
  a: string; // 报错
}
```

**解释**：这里 `a` 属性的类型冲突，导致无法合并。



### 接口方法合并与重载

- **方法重载**：合并时，存在同名方法，这些方法会进行重载，后定义的优先级更高。
- **字面量类型优先级**：在方法参数中，字面量类型的优先级最高。

```ts
interface Cloner {
  clone(animal: Animal): Animal;
}

interface Cloner {
  clone(animal: Sheep): Sheep;
}

interface Cloner {
  clone(animal: string): Sheep;
  clone(animal: '大黄'): Sheep;
}

interface Cloner {
  clone(animal: '小黑'): Sheep;
}

interface Cloner {
  clone(animal: Dog): Dog;
  clone(animal: Cat): Cat;
}
```

**合并后重载顺序**：`小黑、大黄、Cat、Dog、string、Sheep、Animal`



如果两个接口组成联合类型，且有同名属性，该属性的类型为联合类型。

```ts
interface Circle {
  area: bigint;
}

interface Rectangle {
  area: number;
}

declare const s: Circle | Rectangle;

s.area; // 类型为 bigint | number
```

**解释**：`s.area` 的类型是 `bigint | number`，因为 `Circle` 和 `Rectangle` 的 `area` 属性类型不同



