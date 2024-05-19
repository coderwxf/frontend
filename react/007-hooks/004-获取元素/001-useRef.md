在函数组件中，可以基于`useRef`获取DOM元素

```jsx
import { useEffect, useRef } from 'react'

export default function App() {
  // useRef() 用于获取元素的引用 --- 参数是默认值
  const el = useRef(null)

  // 在componentDidMount中输出结果
  useEffect(() => {
   console.log(el.current)
  }, [])

  return (
    <div ref={el}>Hello React</div>
  )
}
```



## useRef vs createRef

+ `useRef` 在函数组件中使用，返回的引用对象在组件的整个生命周期内保持不变，`current` 属性会根据 DOM 元素的变化自动更新
+ `createRef` 在类组件中使用，每次重新渲染时都会返回一个新的引用对象



`useRef`

```jsx
import { useRef, useState } from 'react'

let preEl = null

export default function App() {
  const [num, setNum] = useState(0)
  const el = useRef(null)

  if (!preEl) {
    preEl = el
  } else {
    // useRef 每次都会返回相同的引用
    console.log(el === preEl) // => true
  }

  return (
   <>
    <div ref={el}>{num}</div>
    <button onClick={() => setNum(num + 1)}>increment</button>
   </>
  )
}
```



`createRef`

```jsx
import { createRef, useState } from 'react'

let preEl = null

export default function App() {
  const [num, setNum] = useState(0)
  const el = createRef(null)

  if (!preEl) {
    preEl = el
  } else {
    // React.createRef在函数组件中依然可以使用
    // 只不过createRef 每次渲染都会返回一个新的引用
    console.log(el === preEl) // => false
  }

  return (
   <>
    <div ref={el}>{num}</div>
    <button onClick={() => setNum(num + 1)}>increment</button>
   </>
  )
}
```



## 作用于类组件

```jsx
import { useEffect, useRef } from "react"

class Child extends React.Component {
  submit = () => {
    console.log('调用了子组件的submit方法！')
  }

  render() {
    return <div>Child</div>
  }
}

export default function Demo() {
  const box = useRef(null)

  useEffect(() => {
    // useRef作用于类组件，获取的也是完全的组件实例，可以调用组件的任何方法和属性
    console.log(box.current)
    box.current.submit()
  }, [])

  return (
    <div>
      <Child ref={box} />
    </div>
  )
}
```



## 作用于函数组件

```jsx
import { useEffect, useRef, forwardRef } from "react"

const Child = forwardRef((props, ref) => {
  // 函数组件没有实例, 如果要获取DOM元素, 依旧需要使用forwardRef转发
  return <div ref={ref}> Child </div>
})

export default function Demo() {
  const box = useRef(null)

  useEffect(() => {
    console.log(box.current)
  }, [])

  return (
    <div>
      <Child ref={box} />
    </div>
  )
}
```

