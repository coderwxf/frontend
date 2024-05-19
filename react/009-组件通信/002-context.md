`ThemeContext.js`

```js
import { createContext } from 'react'

// 创建上下文对象
const ThemeContext = createContext()
export default ThemeContext
```



`App.jsx`

```jsx
import { useCallback, useState } from 'react'
import ThemeContext from './ThemeContext'
import Count from './Count'
import Opt from './Opt'

function App() {
  const [count, setCount] = useState(0)

  const onChange = useCallback(function (step) {
    setCount(count + step)
  }, [count])

  return (
    <ThemeContext.Provider value={{
      count,
      onChange
    }}>
      <Count count={count} />
      <Opt onChange={onChange} />
    </ThemeContext.Provider>
  )
}

export default App
```

`Count.jsx`

```jsx
import React, { memo, useContext, useMemo } from 'react'
import ThemeContext from './ThemeContext'
import PropTypes from 'prop-types'

const Count = memo(() => {
  // 函数组件只要通过useContext 这个hook就可以直接获取context
  const { count } = useContext(ThemeContext)
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
import { memo, useContext } from 'react'
import ThemeContext from './ThemeContext'
import PropTypes from 'prop-types'

const Opt = memo(() => {
  const { onChange } = useContext(ThemeContext)

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

