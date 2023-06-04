在TypeScript中，this的默认类型是any

如果我们不希望使用隐式的（模糊的）any类型，可以使用tsconfig.json来修改typescript的默认编译配置

```shell
# 生成ts的编译配置文件 tsconfig.json
# 如果没有这个配置文件，ts会使用其内置的默认配置选项进行配置
tsc --init
```



在生成的默认配置文件中，会开启严格模式`strict:true` ，而该配置中，包含了`noImplicitAny: true`

也就是说开启该配置项后，TypeScript会根据上下文推导this，但是在不能正确推导时，就会报错，需要我们明确的指定 this

```ts
const user = {
  name: 'Klaus',
  sayName() {
    // 这里的this就会通过上下文被推导出来
    console.log(this.name)
  }
}

user.sayName()

function foo() {
  // 这里的this并不能被明确的推导出来，所以这里的this是any
  // 而noImplicitAny不允许存在类型为any的this，所以会报错
  console.log(this.name)
}
```



但是根据上下文推导的时候，所推导出的this类型只是可能性最大的this类型，而不一定是正确的this类型

```ts
const user = {
  name: 'Klaus',
  sayName() {
    console.log(this.name)
  }
}

// 此时推导出的this类型是正常的
user.sayName()

// 此时对应的this类型是错误的
// 但是此时typescript并不能发现对应的错误
user.sayName.call({})
```



所以我们可以手动的指定对应的this类型

```ts
const user = {
  name: 'Klaus',
  // 函数的第一个参数 必须是一个名为this的参数，而对应的类型就是我们所需要指定的this类型，this类型在编译后会被抹除
  // 所以我们可以从第二个参数开始传递我们实际所需要传递的对应的参数列表
  sayName(this: { name: string }) {
    console.log(this.name)
  }
}

user.sayName()

// 我们指定了实际所需要传递的this类型，所以此时ts就会发现对应的错误
user.sayName.call({})
```



## 内置工具

Typescript 提供了一些工具类型来辅助进行常⻅的类型转换，这些类型全局可用



### ThisParameterType

ThisParameterType用于提取一个函数类型Type的this的参数类型

如果这个函数类型没有this参数返回unknown

```ts
function foo(this: { name: string }) {
  console.log('Hello World')
}

/* 
  此时thisType的类型为 {
    name: string
  }
*/
type thisType = ThisParameterType<typeof foo>
```



### OmitThisParameter

OmitThisParameter用于移除一个函数类型Type的this参数类型, 并且返回当前的函数类型

```ts
function foo(this: { name: string }) {
  console.log('Hello World')
}

// pureFnType ==> () => void
type pureFnType = OmitThisParameter<typeof foo>
```



### ThisType

这个类型不返回一个转换过的类型，它被用作标记一个上下文的this类型

```ts
interface IState {
  name: string
  age: number
}

interface IStore {
  state: IState
  printName: () => void
  printAge: () => void
}

const store: IStore = {
  state: {
    name: 'Klaus',
    age: 23
  },

  printName() {
    console.log(this.name)
  },

  printAge() {
    console.log(this.age)
  }
}

// 此时js逻辑是正确的，但是ts无法通过上下文准确推测出正确的this指向
store.printName.call(store.state)
store.printAge.call(store.state)
```

所以，第一种解决方式，我们可以给每一个函数都指定正确的this指向

```ts
interface IState {
  name: string
  age: number
}

interface IStore {
  state: IState
  printName: () => void
  printAge: () => void
}

const store: IStore = {
  state: {
    name: 'Klaus',
    age: 23
  },

  printName(this: IState) {
    console.log(this.name)
  },

  printAge(this: IState) {
    console.log(this.age)
  }
}

store.printName.call(store.state)
store.printAge.call(store.state)
```

但是这样必然会存在重复性的代码，为此TS提供了ThisType来指定当前上下文中的this指向

```ts
interface IState {
  name: string
  age: number
}

interface IStore {
  state: IState
  printName: () => void
  printAge: () => void
}

// ThisType<IState> -> 对象store中的所有this的类型都是IState
const store: IStore & ThisType<IState> = {
  state: {
    name: 'Klaus',
    age: 23
  },

  printName() {
    console.log(this.name)
  },

  printAge() {
    console.log(this.age)
  }
}

store.printName.call(store.state)
store.printAge.call(store.state)
```

