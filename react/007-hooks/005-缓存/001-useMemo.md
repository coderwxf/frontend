可以使用`useMemo`模拟类似于计算属性的功能

```jsx
import { useState, useMemo } from 'react'

export default function Demo() {
  let [x, setX] = useState(10),
      [y, setY] = useState(20)

  // 普通函数调用
  const functionVal = () => {
    console.log('function call')
    return x * 10
  }

  // 计算属性 --- useMemo(handler, 依赖数组)
  // 如果依赖数组为空，则除了首次渲染时，useMemo的callback会被执行
  // 后续useMemo的callback将永远不会被执行，因为没有依赖项会发生改变
  const computedVal = useMemo(() => {
    console.log('useMemo')
    return x * 10
  }, [x])

  return (
    <div>
      <div>{functionVal()}</div>
      <div>{x} * 10  = {computedVal}</div>
      {/*
         修改x
         1. computedVal 因为依赖数组的改变，而重新计算
         2. UI rerender 导致 函数被重新执行
      */}
      <button onClick={() => setX(x + 1)}>add X</button>
      {/*
        修改y
        1. computedVal 因为依赖数组没有依赖项发生改变，而不会重新计算
        2. UI rerender 导致 函数被重新执行
      */}
      <button onClick={() => setY(y + 1)}>add Y</button>
    </div>
  )
}
```

