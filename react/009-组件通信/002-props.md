`父组件`

```jsx
import { useCallback, useState } from 'react'
import Count from './Count'
import Opt from './Opt'

function App() {
  const [count, setCount] = useState(0)

  // 只要函数作为props传入，就需要使用useCallback
  const onChange = useCallback(function (step) {
    setCount(count + step)
    // 当count更新后会生成新function
    // 新function在新的执行上下文中，所以可以获取最新的count值
    // 如果依赖数组是空数组，永远是第一个执行上下文中的count，即count值永远为0
  }, [count])

  return (
    <>
      <Count count={count} />
      <Opt onChange={onChange} />
    </>
  )
}

export default App
```

`Count.jsx`

```jsx
import { memo, useMemo } from 'react'
import PropTypes from 'prop-types'

const Count = memo(({ count }) => {
  // 通过useMemo 实现计算属性
  const dblCount = useMemo(() => count * 2, [count])

  return (
    <>
      <div>count: { count }</div>
      <div>doubule count: { dblCount }</div>
    </>
  )
})

Count.defaultProps = {
  count: 0
}

Count.propTypes = {
  count: PropTypes.number
}

export default Count
```

`Opt.jsx`

```jsx
import { memo } from 'react'
import PropTypes from 'prop-types'

const Opt = memo(({ onChange }) => {
  return (
    <>
      <button onClick={() => onChange(1)}>+1</button>
      <button onClick={() => onChange(-1)}>-1</button>
    </>
  )
})

Opt.prototype = {
  onChnage: PropTypes.func.isRequired
}

export default Opt
```

