在函数组件中获取DOM的方式

1. 原生方式
2. 函数形式ref
3. `createRef`
4. `useRef`



## 函数形式ref

```jsx
import { useEffect } from "react"

export default function Child(props) {
  let divEl = null

  useEffect(() => {
    console.log(divEl) // => 函数形式获取 => 不推荐
  }, [divEl])

  return (
    <>
      <div ref={el => divEl = el}>lorem</div>
    </>
  )
}
```



## createRef

```jsx
import { createRef, useEffect, useState } from "react"

let prevRef = null

export default function Child(props) {
  let divEl = createRef()
  let [count, setCount] = useState(0)

  useEffect(() => {
    console.log(divEl.current) // => ref对象形式获取

    if (!prevRef) {
      prevRef = divEl
    } else {
      // 通过createRef生成的ref对象
      // 组件每次更新都会产生全新的ref对象
      console.log(prevRef === divEl) // => false
    }
  }, [divEl])

  return (
    <>
      <div ref={divEl}>{ count }</div>
      <button onClick={() => setCount(count + 1)}>click me</button>
    </>
  )
}
```



## useRef

```jsx
import { useEffect, useRef, useState } from "react"

let prevRef = 0

export default function Child(props) {
  const [count, setCount] = useState(0)
  let divRef = useRef(null) // => hook方式创建ref对象 => 参数是初始值
  console.log(divRef) // => { current: null }

  useEffect(() => {
    console.log(divRef.current) // => hook方式获取实例
  }, [divRef])

  useEffect(() => {
    if (divRef) {
      if (!prevRef) {
        prevRef = divRef
      } else {
        // useRef创建的ref对象和组件想关联
        // 在组件每次更新创建的全新函数执行上下文中，ref对象都是同一个
        console.log(prevRef === divRef) // => true
      }
    }
  })

  return (
    <>
      <div ref={divRef}>{count}</div>
      <button onClick={() => setCount(count + 1)}>click me</button>
    </>
  )
}
```



## createRef vs useRef

类组件 推荐 `createRef` 

=> 类组件更新只是重新执行`render`方法，其余逻辑并不会被执行，所以每次更新都是同一个ref对象

=> 类组件中不能使用`useRef`

```jsx
import { Component, createRef } from 'react'

export default class Child extends Component {
  h2Ref = createRef()

  // 组件每次更新，会重新执行render方法，但不会重新创建新的实例
  // => 组件每次更新，并不需要重新执行createRef()
  // => 也就不会每次更新创建出多个ref对象
  render() {
    return (
      <>
        <div ref={this.h2Ref}>lorem</div>
      </>
    )
  }
}
```





函数组件推荐`useRef` => 函数组件每次都是重新执行函数

=> 使用`createRef`创建ref对象，会导致每次都会生成一个全新的ref对象

=> 使用`useRef`在 `createRef`基础上进行了优化 => ref对象和组件进行绑定，和函数的执行上下文无关

=> 每次生成的都是全新的ref对象

```jsx
import { useRef } from "react"

export default function Child(props) {
  // 函数组件每次更新，是重新执行函数，并形成全新的函数执行上下文
  // 如果使用createRef会导致重复创建ref对象 => 浪费性能
  // 如果使用useRef会创建和组件关联的唯一ref对象 => 推荐使用
  let h2Ref = useRef(null)

  return (
    <>
      <h2 ref={h2Ref}>lorem</h2>
    </>
  )
}
```



## useImperativeHandle

```jsx
import { forwardRef, useEffect, useRef, useState } from "react"

const Count = forwardRef(function(props, ref) {
  const [count] = useState(0)

  return <>
    {/* 通过ref转发获取ref实例  */}
    <div ref={ref}>{ count }</div>
  </>
})

export default function Child(props) {
  const countRef = useRef(null)

  useEffect(() => {
    console.log(countRef.current) // => <div>0</div>
  })

  return (
    <>
     <Count ref={countRef} />
    </>
  )
}
```

```jsx
import { forwardRef, useEffect, useImperativeHandle, useRef, useState } from "react"

const Count = forwardRef(function(props, ref) {
  const [count, setCount] = useState(0)

  function increment() {
    setCount(count + 1)
  }

  // 将函数组件需要向外暴露的属性和方法挂载到ref上 
  useImperativeHandle(ref, () => ({
    count,
    increment
  }))

  return <>
    <div>{ count }</div>
  </>
})

export default function Child(props) {
  const countRef = useRef(null)

  useEffect(() => {
    console.log(countRef.current) // => {count: 0, increment: () => {} )
  })

  return (
    <>
     <Count ref={countRef} />
    </>
  )
}
```

