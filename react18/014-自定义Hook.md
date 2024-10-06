通过自定义Hook来实现自定义逻辑代码的封装和提取

+ 在React中，以`withXxxx`来描述高阶组件（Higher-Order Component，HOC）
  + HOC 是一个函数，接受一个组件作为参数，并返回一个新的组件

+ 在React中，以`useXxxx`来描述Hook函数 
  + 在自定义hook函数中也可以使用hook函数，使用规则和函数组件内使用hook的规则一致

这并不是强制性规则，是约定俗成的推荐性规则，但推荐使用约定规则进行编写，方便Lint工具更好的对使用规则进行校验



```jsx
// 示例: 实现可以支持部分修改的useState
import { useState } from "react"

// 底层本质使用的还是 React内置提供的哪些hook函数
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

