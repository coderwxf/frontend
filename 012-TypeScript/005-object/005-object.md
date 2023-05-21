在 JavaScript 中，我们组织和传递数据的基本方式是通过对象

在 TypeScript 中，我们通过对象类型来表示它们，以下是描述对象类型的三种方式

```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

```ts
interface Person {
  name: string;
  age: number;
}
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```

```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```



## 属性修饰符

```ts
interface PaintOptions {
  shape: Shape;
  xPos?: number; // ? 表示可选
  yPos = 0; // 设置默认值，因为后续yPos可能会发生改变，所以typeof yPos -> number
  readonly zPos?: number;  // readonly 表示只读
}
```

```ts
interface Person {
  name: string;
  age: number;
}

interface ReadonlyPerson {
  readonly name: string;
  readonly age: number;
}

let writablePerson: Person = {
  name: "Person McPersonface",
  age: 42,
};

// 检查两个类型是否兼容时，tsc并不考虑属性修饰符是否兼容
let readonlyPerson: ReadonlyPerson = writablePerson;

console.log(readonlyPerson.age); // prints '42'
writablePerson.age++;
console.log(readonlyPerson.age); // prints '43'
```



## 索引签名

有时候事先不知道类型的所有属性名称，但知道值的形状(也就是只的数据结构)，在这种情况下，可以使用索引签名来描述可能值的类型

在 TypeScript 中，索引签名属性只能使用字符串、数字、symbol来作为索引值，或结合模板字符串以及联合类型进行使用

```ts
interface StringArray {
 	// key为number value为string 的键值对类型
  // 在索引签名中也可以使用属性修饰符
  [index: number]: string; // -> 索引签名
}
```

```ts
// 可以同时存在字符串索引和数值索引 --- 因为数值索引会被转换为字符串索引，所以数值索引对应的值必须是字符串索引的子类型
interface Foo {
  // index索引的类型必须是key索引类型的子类型
  [index: number]: string;
  // 在使用字符串索引签名时，所有属性的类型必须与索引签名返回的类型相匹配
  [key: string]: string | number;
  // name对应的值也必须是key索引类型对应的子类型
  name: string;
}
```



## 接口继承

使用 extends 关键字可以让我们扩展其他类型，并添加新的成员，从而避免了重复声明相同的属性

接口也可以从多个类型扩展，这对于在不同的类型之间建立关系非常有用

在接口中使用 extends 关键字可以让我们有效地从其他命名类型中复制成员，并添加任何新成员

这可以用于减少我们需要编写的类型声明样板代码的数量，并表示多个相同属性的不同声明可能是相关的

```ts
interface BasicAddress {
  name?: string;
  street: string;
  city: string;
  country: string;
  postalCode: string;
}

interface AddressWithUnit extends BasicAddress {
  unit: string;
}
```

接口也可以从多个类型扩展

```ts
interface Colorful {
  color: string;
}

interface Circle {
  radius: number;
}

interface ColorfulCircle extends Colorful, Circle {}

const cc: ColorfulCircle = {
  color: "red",
  radius: 42,
};
```



## 泛型对象

泛型对象类型通常是一种容器类型，它可以独立于它所包含的元素类型工作。这使得数据结构可以在不同的数据类型之间重用

例如 `Type[]`本质上就是`Array<Type>`的简写，同样的还有`Map<K, V>`、`Set<T>`和`Promise<T>`

```ts
interface Box<Type> {
	contents: Type;
}
```

