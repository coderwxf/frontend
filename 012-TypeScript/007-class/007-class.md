TypeScript 对 ES2015 中引入的 class 关键字提供了完整的支持



## 字段

```ts
class Point {
	// 字段声明会在类上创建一个公共可写属性 --- 与其他位置一样，类型注解是可选的，但如果未指定，则会被隐式推断为 any 类型
  x: number;
  
  // 字段也可以具有初始化程序；当类被实例化时，这些程序将自动运行，以便推断出字段的类型
  y = 0;
}
 
const pt = new Point();
pt.x = 0;
pt.y = 0;
```



### strictPropertyInitialization

在`tsconfig.json`中存在一个字段`strictPropertyInitialization`，用于控制类字段是否需要在构造函数中初始化

如果开启了`strictPropertyInitialization`, 则必须设置初始化值

> 在 TypeScript 中，类的字段需要在构造函数中初始化
>
> 因为子类实例化时，必须先调用父类的构造函数，并确保所有字段都已经正确地初始化
>
> 如果在构造函数中使用实例方法来初始化字段，那么这些方法可能会在子类中被重写，导致字段未被正确地初始化



### readonly

字段可以加上 readonly 修饰符，以表示该字段是只读属性，也就是说无法在构造函数之外对该字段进行赋值

```ts
class Greeter {
  readonly name: string = "world";
 
  constructor(otherName?: string) {
    // readonly属性只能在类成员变量声明的时候赋值，并在构造函数中被修改
    // 其余任何时候readonly属性都无法再被修改 即使是在类方法中
    if (otherName !== undefined) {
      this.name = otherName;
    }
  }
 
  err() {
    this.name = "not ok";
    // error: 无法分配到 'name'，因为它是只读属性。
  }
}
const g = new Greeter();
g.name = "also not ok";
// error: 无法分配到 'name'，因为它是只读属性。
```



### constructor

构造函数与函数非常相似。可以添加带有类型注解、默认值和重载的参数

```ts
class Point {
  x: number;
  y: number;
 
  // 带有默认值的正常签名
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
}
```

```ts
class Point {
  // 重载
  constructor(x: number, y: string);
  constructor(s: string);
  constructor(xs: any, y?: any) {
    // 待定
  }
}
```



类构造函数签名与函数签名之间只有一些微小的差别：

- 构造函数不能有类型参数(即泛型) --- 这些应该在外部类声明中被定义
- 构造函数不能有返回类型注解 --- 类实例类型始终是返回的类型



### super call

与 JavaScript 中一样，如果有一个基类，需要在构造函数体中使用任何 this 成员之前调用 `super()`, 以初始化父类构造函数



### methods

在 TypeScript 中，类中的函数属性称为方法，方法的类型注解方式和普通函数的类型注解方式一致

如果在方法体中访问类的属性或其他方法，需要使用 this 关键字来引用当前对象

如果在方法体中使用未限定名称，它将会在方法外部进行查找

```ts
class Point {
  x = 10;
  y = 10;
 
  scale(n: number): void {
    this.x *= n;
    this.y *= n;
  }
}
```



### Getters / Setters

```ts
class C {
  // 类也可以有访问器
  #length = 0;
  get length() {
    return this.#length;
  }
  set length(value) {
    this.#length = value;
  }
}
```

