```shell
this.setState([partialState | stateCallback], [callback])
```

+ `partialState` => 支持部分状态修改 => 会和原始state 执行 `Object.assign`合并
+ `callback`  => 如果`SCU`返回了false => `componentDidUpdate`不会执行 但 `callback`会被执行



`setState`更新是异步的 => 类似于Vue中的`nextTick`

vue中的一个更新周期 是一个事件循环周期 「 tick周期 」

![image.png](https://s2.loli.net/2024/09/28/FBkdLDsMq8KS1V2.png) 

```vue
<script setup>
import { ref, onUpdated } from 'vue'

const count = ref(0)

function add() {
  setTimeout(() => {
    count.value += 1
  }, 1000)

  setTimeout(() => {
    count.value += 1
  }, 1000)

  setTimeout(() => {
    count.value += 1
  }, 1000)
}

onUpdated(() => console.log('updated'))  // updated会输出三次
</script>

<template>
	<!-- count的值是3 -->
  <h1>{{ count }}</h1>
  <button @click="add">click</button>
</template>
```





React中批处理机制 和 Vue类似 => 但React的批处理机制确实是通过React自己来实现的

1. 在一个更新周期中:

   + 状态更新被加入到更新队列「 updater 」中

   + 回调函数被插入到回调队列 「 callbacks 」中

2. 一个更新周期执行结束
   + 通过`Object.assign`合并`updater`中的所有状态更新使其成为一次更新 
   + 批处理操作 => 减少渲染次数，提升更新性能
3. 更新状态和视图
4. 执行后续生命周期
5. 依次执行`callbacks`中的函数 => 可以访问更新后的 DOM 和状态

```jsx
import { Component } from 'react'

export default class Child extends Component {
  state = {
    count: 0
  }

  add() {
    setTimeout(() => {
      this.setState({ count: this.state.count + 1 })
    }, 1000)

    setTimeout(() => {
      this.setState({ count: this.state.count + 1 })
    }, 1000)

    setTimeout(() => {
      this.setState({ count: this.state.count + 1 })
    }, 1000)
  }

  componentDidUpdate() {
    console.log('updated') // 只会输出一次updated
  }

  render() {
    return (
      <>
        {/* count的值是1  */}
        <h1>{ this.state.count }</h1>
        <button onClick={() => this.add()}>click</button>
      </>
    )
  }
}
```



React对于非React控制回调实现批处理是依赖于React内容自己实现的调度器

调度器会收集可以同时执行的宏任务，但依旧会存在误差

例如如下情况，调度器可能有时会将他们视为一次更新，有时会识别为多次更新

```js
add() {
  setTimeout(() => {
    this.setState({ count: this.state.count + 1 })
    console.log(1)
  }, 1000)

  setTimeout(() => {
    this.setState({ count: this.state.count + 1 })
    console.log(2)
  }, 1001)

  setTimeout(() => {
    this.setState({ count: this.state.count + 1 })
    console.log(3)
  }, 1002)
}
```



`setState`同步异步问题:

+ 在React18中，任何`setState`都是异步操作 

+ 在React18之前
  1. React自己执行的`setState`都是异步 「 生命周期、合成事件 」
  2. 非React执行的`setState`都是同步「 定时器、原生DOM事件  」



`componentDidUpdate`会在任何状态更新完毕后会回调

`callback`只会在对应状态更新完毕后，才会被回调



```jsx
const { count } = this.state // => count -> 0

this.setState({ count: count + 1 })
this.setState({ count: count + 1 })
this.setState({ count: count + 1 })
// => state -> { count: 1 } => 被批处理了
```

```jsx
// 第一次 初始state => { count: 0 }
this.setState(preState => ({
  count: preState.count + 1 // => stateCallback的返回值是新state => 支持部分状态更新
}))
 
// 之后  上一次state =>  { count: 1 }
this.setState(preState => ({
  count: preState.count + 1
}))

// preState => { count: 2 }
this.setState(preState => ({
  count: preState.count + 1
}))

// 结果: state -> { count: 3 } 界面重新渲染一次 
```



## flushSync

在`react-dom`中，不在`react`中

```jsx
const { count } = this.state // => count -> 0
this.setState({ count: count + 1 })

// flushSync会在回调执行完毕后，立即刷新updater队列 => 即依次取出并执行updater中每一次的状态更新
flushSync(() => {
  this.setState({ count: count + 1 })
})

console.log(this.state.count) // => 1
```

```jsx
// 初始state => { count: 0 }

// 错误写法: 
// 	const { count } = this.state
// 	this.setState({ count: count + 1 }) => count一旦解构就不会实时更新了

this.setState({ count: this.state.count + 1 }) // => { count: 1 }
flushSync() // => 这是最为常用的使用方式
this.setState({ count: this.state.count + 1 }) // => { count: 2 }
flushSync()
this.setState({ count: this.state.count + 1 }) // => { count: 3 }

// 结果 state -> { count: 3 } => 界面重新渲染三次
```

