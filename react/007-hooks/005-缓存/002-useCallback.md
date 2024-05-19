组件触发更新, 生成新的执行上下文, 每个执行上下文中的函数都是全新的

导致传递给子组件的prop的引用也是全新的, 进而导致props的浅比较始终返回false

```jsx
import { useState, memo } from 'react'

const Child = memo(() => {
  console.log('child render')
  return <div>Child</div>
})


export default function App() {
  const [count, setCount] = useState(0)

  function increment() {
    setCount(count + 1)
  }

  function foo() {
    console.log('foo')
  }

  return (
    <>
      <Child foo={foo} />
      <button onClick={increment}>increment</button>
    </>
  )
}
```



为了解决这种情况，可以使用`useCallback`

```jsx
import { useState, memo, useCallback } from 'react'

const Child = memo(() => {
  console.log('child render')
  return <div>Child</div>
})


export default function App() {
  const [count, setCount] = useState(0)

  function increment() {
    setCount(count + 1)
  }

  // useCallback(callback, 依赖数组)
  // 1. 首次渲染，获取callback的引用并缓存
  // 2. 再次渲染
  //    + 如果依赖数组中有任何一项发生改变，重新执行callback获取最新引用
  //    + 否则，直接返回上一个callback对应的引用地址
  const foo = useCallback(() => console.log('foo'), [])

  return (
    <>
      <Child foo={foo} />
      <button onClick={increment}>increment</button>
    </>
  )
}
```



## 示例

```jsx
import { useState, memo, useCallback } from 'react'

const Child = memo(({ increment }) => {
  return <button onClick={increment}>increment</button>
})


export default function App() {
  const [count, setCount] = useState(0)

  // 依赖于count, 当count改变，获取最新函数的引用

  // 不能写成 useCallback(() => setCount(count + 1), [])
  // 否则获取到的永远是第一个callback的引用，根据闭包规则，获取的永远是第一个闭包中的count
  // 也就是说count的值永远是0

  const increment = useCallback(() => setCount(count + 1), [count])

  return (
    <>
      <div>{ count }</div>
      <Child increment={increment} />
    </>
  )
}
```

