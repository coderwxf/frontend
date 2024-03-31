## 老版本setState

在React18中，一切setState都是异步操作

在React16中

+ 在像合成事件，生命周期等由React控制的上下文中，setState是异步的
+ 在像定时器，原生DOM事件监听等非React控制的上下文中，setState是同步的 「 立即更新状态并重新更新视图 」



## flushSync

`flushSync(() => {})`

1. 接收一个函数
2. 运行时，会同步加载回调函数
3. 等回调函数执行完毕后，立即更新状态并重新渲染视图

```jsx
import { Component } from 'react'
// flushSync 是从react-dom中引入的，不是从react中引入的
import { flushSync } from 'react-dom'

export default class App extends Component {
  state = {
    x: 10,
    y: 20,
    z: 0
  }

  render() {
    const { x, y, z } = this.state
    return (
      <>
        <div>{x} + {y} = {z}</div>
        <button onClick={() => this.change()}>change</button>
      </>
    )
  }

  change() {
    // 1. flushSync之前的setState是异步执行的
    this.setState({ x: this.state.x + 1 })
    console.log(this.state) // => { x: 10, y: 20, z: 0 }

    // 2. flushSync的回调函数会被同步执行，回调函数内部setState依旧是异步执行的
    // flushSync(fn) 等价于 1. fn() 2. 更新DOM 3. 视图更新
    flushSync(() => {
      this.setState({ y: this.state.y + 1 })
      console.log(this.state) // => { x: 10, y: 20, z: 0 }
    })

    // 3. flushSync回调执行完毕后，会立即更新状态并重新渲染视图
    console.log(this.state) // => { x: 11, y: 21, z: 0 }
  }

  componentDidUpdate() {
    console.log('componentDidUpdate')
  }
}
```



上述代码中

```jsx
flushSync(() => {
  this.setState({ y: this.state.y + 1 })
  console.log(this.state) 
})
```

会先同步执行回调，再回调执行完毕后，立即更新状态并重新渲染视图

从效果上等价于

```jsx
this.setState({ y: this.state.y + 1 })
console.log(this.state)
flushSync()
```

