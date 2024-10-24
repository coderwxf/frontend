在React中，以`withXxxx`来描述高阶组件（Higher-Order Component，HOC）

+ HOC 是一个函数，接受一个组件作为参数，并返回一个新的组件

在React中，以`useXxxx`来描述Hook函数 

+ 在自定义hook函数中也可以使用hook函数，使用规则和函数组件内使用hook的规则一致

> 命名规则非强制性，属于推荐规则 => 方便Lint工具进行规则校验



## 示例

功能: 实现支持部分更新的hook函数

```jsx
import { useState } from "react"

// 自定义hook => 底层本质使用的还是 React内置提供的哪些hook函数
function usePartialState(initalState) {
  const [state, setState] = useState(initalState)

  function setPartialState(partialState) {
    setState({
      ...state,
      ...partialState
    })
  }

  // 使用自己更新状态的方法 替换 原生的更新状态方法
  return [state, setPartialState]
}

// TEST CODE
export default function Child(props) {
  const [state, setState] = usePartialState({
    x: 10,
    y: 20
  })

  return (
    <>
      <div>x: {state.x} - y: { state.y }</div>
      <button  onClick={() => setState({ x: state.x + 1 })}>add X</button>
    </>
  )
}
```

