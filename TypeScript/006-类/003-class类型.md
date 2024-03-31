## 实例类型

```ts
class Color {
  name:string;

  constructor(name:string) {
    this.name = name;
  }
}

// 类名 表示的是 类的实例类型
const green: Color = new Color('green');
```

类名作为类型使用，可以被认为是给实例对象起别名，

因此TypeScript 有三种方法可以为对象类型起名：type、interface 和 class



## 类类型

```ts
class Color {
  name:string;

  constructor(name:string) {
    this.name = name;
  }
}

// typeof Color表示类类型本身
const color: typeof Color = Color
console.log(new color('red'))
```



类本质就是构造函数 所以Color可以使用以下两个方式替代

```ts
new (name: string) => Color
```

```ts
{
  new (x:number, y:number): Point
}
```

