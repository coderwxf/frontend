## 数据分类

根据是否参数界面渲染，可以分为

1. 数据 --- 不参与界面渲染的数据
2. 状态 --- 参与界面渲染的数据，当数据更新时，需要通过react进行UI更新



## state

1. react规定，所有的状态都必须定义在实例属性`state`中
2. 更新状态后，通过`setState`方法通过react进行UI更新
3. 如果没有设置`state`，则`this.state`的默认值是`null`



```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
	// 通过类的成员属性初始化state, 可以省略构造函数
  state = {
    name: 'John',
    age: 23
  }

  render() {
    const { name, age } = this.state
    return (
      <>
        {name} - {age}
      </>
    )
  }
}
```



## setState

在vue中，对视图状态进行了数据劫持

- 使用状态，会触发get操作，对应视图部分会作为状态的依赖被收集
- 修改状态，会触发set操作，该状态对应的所有依赖会被同时进行更新

而react中，并没有对状态进行任何的数据劫持, 因此直接修改state中的状态并不能触发界面更新

在react中，如果需要在修改状态的同时，进行界面刷新，必须通过setState方法

```jsx
addAge() {
    // 这种方式修改状态 只能单纯的修改状态值 无法触发视图更新
    // 会导致视图和状态不一致，不推荐
    // this.state.age+= 1

    // setState(obj)
    // 1. 修改状态 --- 新旧state执行Object.assign操作，生成新state
    // 2. 触发视图更新
    this.setState({ age: this.state.age + 1 })

    // setState是异步更新 不能立即获取到最新的state
    console.log(this.state.age) // => 23
  }
}
```



## forceUpdate

```jsx
addAge() {
    this.state.age+= 1

  	// 强制使用最新的state重新渲染UI
    this.forceUpdate()

    // 界面强制刷新是同步的
    console.log(this.state.age) // => 24
  }
```

