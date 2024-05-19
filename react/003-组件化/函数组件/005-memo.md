通过`memo`函数来实现类似于`PureComponent`中的`shouldComponentUpdate`

```jsx
import { useState } from "react"

// 函数组件的另一种定义方式
const Child = () => {
  console.log('child render')
  return <div>Child</div>
}

export default function App() {
  const [count, setCount] = useState(0)

  function increment() {
    setCount(count + 1)
  }

  return (
    <>
      <div>{count}</div>
      <Child />
      <button onClick={increment}>increment</button>
    </>
  )
}
```

默认情况下，点击按钮，状态值发生改变

重新执行App函数，生成一个新的函数执行上下文

因为重新渲染，父组件和其对应子组件都会被重新执行

但实际上Child组件并没有发生任何变更，重新渲染会导致性能浪费 



```jsx
import { useState, memo } from "react"

const Child = memo(() => {
  console.log('child render')
  return <div>Child</div>
})

export default function App() {
  const [count, setCount] = useState(0)

  function increme  nt() {
    setCount(count + 1)
  }

  return (
    <>
      <div>{count}</div>
      <Child />
      <button onClick={increment}>increment</button>
    </>
  )
}
```

如果函数组件使用`memo`方法包裹，那么除非传递给函数组件的props发生了更新

否则函数组件将不进行任何的渲染流程，从而提升更新性能



## 结合forwardRef

```jsx
// memo 和 forwardRef 一起使用的时候 必须使用memo包裹forwardRef
// 因为forwardRef需要将ref作为第二个参数传递给组件
// 如果使用forwardRef包裹memo, memo是不会将ref进行转发的，导致实际组件无法获取ref参数
const Child = memo(forwardRef((prop, ref) => <div ref={ref}>Child</div>))
```

