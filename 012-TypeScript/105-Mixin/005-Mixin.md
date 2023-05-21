Mixin是一种设计模式，它允许一个类通过继承另外一个类的方法和属性来扩展自己的功能，同时又不会破坏类之间的继承关系

Mixin类本身并不是一个完整的类，它被设计用来被其他类继承，从而为这些类提供额外的功能

Mixin和继承类似，唯一的区别是mixin中继承的类不是父类，而是需要进行功能扩展的类，HOC就是mixin的一种具体实现

以下是实现mixin的两个例子：

```ts
// 通用构造器类型
type GConstructor<T = {}> = new (...args: any[]) => T;

// 传入的类必须有setPos方法 并且符合对应规则
type Positionable = GConstructor<{ setPos: (x: number, y: number) => void }>;

class Sprite {
  name = "";
  x = 0;
  y = 0;

  setPos(x: number, y: number) {
    console.log(x, y)
  }
}

// 通过HOC 实现mixin功能
// TBase extends Positionable --- 对传入的类进行类型约束
function Jumpable<TBase extends Positionable>(Base: TBase) {
  // mixin形成的类不能使用private/protected修饰符， 但是可以使用#修饰符来定义私有属性
  return class Jumpable extends Base {
    jump() {
      this.setPos(10, 20);
    }
  };
}

const Jump = Jumpable(Sprite)
const jumpInstance = new Jump()
jumpInstance.jump()
```



```ts
// 这是实现mixin的另一种实现方式

// 需要扩展的功能类
class Jumpable {
  jump() {}
}

class Duckable {
  duck() {}
}

// 基准类
class Sprite {
  x = 0;
  y = 0;
}

// mixin 是在运行时完成的，编译时 TSC并不能确定扩展类型, 需要手动对类类型进行类型扩展
interface Sprite extends Jumpable, Duckable {}

// 实现mixin
applyMixins(Sprite, [Jumpable, Duckable]);

let player = new Sprite();
player.jump();
console.log(player.x, player.y);

// 通过原型实现mixin
function applyMixins(derivedCtor: any, constructors: any[]) {
  constructors.forEach(baseCtor => {
    Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
      Object.defineProperty(
        derivedCtor.prototype,
        name,
        Object.getOwnPropertyDescriptor(baseCtor.prototype, name) ||
          Object.create(null)
      );
    });
  });
}
```





mixin --- 限制 