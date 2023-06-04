## useRef

useRef返回一个ref对象，返回的ref对象再组件的整个生命周期保持不变



使用场景

+  获取dom元素进行操作
+ 保存一个数据，这个对象在整个生命周期中可以保存不变

```jsx
import React, { memo, useRef, useEffect } from 'react'

const App = memo(() => {
  const titleRef = useRef()

  useEffect(() => {
    console.log(titleRef.current)
  }, [])

  return (
    <div>
      <h2 ref={ titleRef }>hello world</h2>
    </div>
  )
})

export default App
```



## useImperativeHandle

useImperativeHandle这个hook的功能和vue中子组件的defineExpose功能很像

useImperativeHandle决定了父组件可以对子组件进行那些操作，那些操作是不被允许的

````jsx
import React, { memo, useRef, useImperativeHandle, forwardRef } from 'react'

const Child = memo(forwardRef((props, ref) => {
  const inputEl = useRef()

  // useImperativeHandle 在父组件和子组件之间进行了一层拦截
  // 参数一 - 父组件传入到子组件的ref对象
  // 参数二 - 一个需要返回对象的回调函数
  //       - 所返回的对象，即是父组件可以操作的所有方法
  //       - 也就是说所返回的对象会作为inputEl的current属性值
  useImperativeHandle(ref, () => ({
    focus() {
      inputEl.current.focus()
    },
    setValue(v) {
     inputEl.current.value = v
    }
  }))

  return <input ref={inputEl} />
}))

const App = memo(() => {
  const inputEl = useRef()

  function handle() {
    // 如果父组件所执行的操作并不被允许，那么会静默失败
    inputEl.current.setValue('Klaus')
  }

  return (
    <div>
      <Child ref={inputEl} />
      <button onClick={ () => handle() }>handle</button>
    </div>
  )
})

export default App
````



## useLayoutEffect

useLayoutEffect和useEffect的功能很像

useEffect 是在 组件被挂载后回调， 也就是说useEffect这个hook 会在dom挂载完毕后再被回调

useLayoutEffect 是在 虚拟DOM创建完毕 虚拟DOM被挂载到真实DOM之前 被回调

如果我们希望在某些操作发生之后再更新DOM，那么应该将这个操作放到useLayoutEffect

官方更推荐优先使用useEffect， 只有当useEffect出现问题的时候，再去使用useLayoutEffect

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4bf6c99b665f4acaa481d50bddf542e0~tplv-k3u1fbpfcp-zoom-in-crop-mark:3024:0:0:0.awebp)

`useEffect`

 ```jsx
 import React, { memo, useEffect, useState } from 'react'

 const App = memo(() => {
   const [count, setCount] = useState(100)

   // 因为界面先渲染成0之后，再重新改变界面 将内容渲染成一个随机数
   // 所以界面会出现闪动效果
   useEffect(() => {
     if (count === 0) {
       setCount(Math.random() + 100)
     }
   }, [count])

   return (
     <>
       <div>{ count }</div>
       <button onClick={ () => setCount(0) }>+1</button>
     </>
   )
 })

 export default App
 ```



`useLayoutEffect`

```jsx
import React, { memo, useLayoutEffect, useState } from 'react'

const App = memo(() => {
  const [count, setCount] = useState(100)

  // 此时在界面渲染之前已经将count修改成了真正所需要渲染的那个随机数
  // 所以此时界面就不会出现闪烁的情况
  useLayoutEffect(() => {
    if (count === 0) {
      setCount(Math.random() + 100)
    }
  }, [count])

  return (
    <>
      <div>{ count }</div>
      <button onClick={ () => setCount(0) }>+1</button>
    </>
  )
})

export default App
```



自定义Hook本质上只是一种函数代码逻辑的抽取，严格意义上来说，它本身并不算React的特性



需求： 在组件被挂载或者被销毁的时候，打印对应的日志信息

```jsx
import React, { memo, useState, useEffect } from 'react'

const Child = memo(() => {
  useEffect(() => {
    console.log('组件被挂载')

    return () => {
      console.log('组件被销毁')
    }
  }, [])

  return (
    <h2>Child</h2>
  )
})

const App = memo(() => {
  const [isShow, setIsShow] = useState(true)

  useEffect(() => {
    console.log('组件被挂载')

    return () => {
      console.log('组件被销毁')
    }
  }, [])

  return (
   <>
    <h2>App</h2>
    <button onClick={() => setIsShow(!isShow)}>change</button>
    {isShow && <Child />}
   </>
  )
})

export default App
```

可以看到每个组件的useEffect操作其实都是重复的，所以我们需要对这部分代码进行抽取

对于类组件，我们抽取公共代码的方式是使用HOC

而对于函数组件，我们抽取公共代码的方式是使用hook

```jsx
import React, { memo, useState, useEffect } from 'react'

// hook本质上就是使用use开头的函数
function useLog(cName) {
  useEffect(() => {
    console.log(cName + '组件被挂载')

    return () => {
      console.log(cName +'组件被销毁')
    }
  }, [cName])
}

const Child = memo(() => {
  useLog('Child')

  return (
    <h2>Child</h2>
  )
})

const App = memo(() => {
  const [isShow, setIsShow] = useState(true)

  useLog('App')

  return (
   <>
    <h2>App</h2>
    <button onClick={() => setIsShow(!isShow)}>change</button>
    {isShow && <Child />}
   </>
  )
})

export default App
```



### 示例

`共享context`

```jsx
import { useContext } from 'react'
import { TokenContext, UserInfoContext } from "../context";

function useUserContext() {
  const token = useContext(TokenContext)
  const userInfo = useContext(UserInfoContext)

  // hook的返回值 一般是数组，(也可以返回其它的数据结构)
  return [userInfo, token]
}

export default useUserContext
```




`获取滚动位置`

```js
import { useState, useEffect } from 'react'

function useScrollPosition() {
  // 对应的数据需要在组件渲染完成后才可以获取，且这些数据是会实时修改的
  // 所以将对应的数据使用useState来进行定义
  const [scrollX, setScrollX] = useState(0)
  const [scrollY, setScrollY] = useState(0)

  // dom监听是组件渲染之外的额外操作，所以是组件的副作用
  useEffect(() => {
    function handler() {
      setScrollX(window.scrollX)
      setScrollY(window.scrollY)
    }

    // 监听页面滚到 监听window 和 document都是可以的
    // window先滚动，随后会传递给document
    window.addEventListener('scroll', handler)

    return () => {
      window.removeEventListener('scroll', handler)
    }
  }, [])

  return [scrollX, scrollY]
}

export default useScrollPosition
```



`localStorage数据存储`

```js
import { useEffect, useState } from "react"

/*
  需求: localStorage和state保持同步
  1. 取出localStorage中的值 并作为state返回
  2. 当外界修改state中的值的时候，自动更新localStorage
*/
function useLocalState(key) {
  // 如果useState中的初始值逻辑过于复杂
  // 可以将一个函数作为参数传递给useState
  // 对应的函数会立即执行, 并将函数的返回值作为useState的初始值
  const [data, setData] = useState(() => {
    // localStorage.getItem(undefined) -> error
    // localStorage.getItem('') -> error
    // localStorage.getItem(null) -> null
    const item = localStorage.getItem(key)
    if (!item) return ''
    return JSON.parse(item)
  })

  useEffect(() => {
    // JSON.stringify('') -> '""'
    // JSON.stringify(undefined) -> undefined -> 注意返回的不是字符串，而是undefined
    // JSON.stringify(null) -> 'null'
    localStorage.setItem(key, JSON.stringify(data))
  }, [key, data])

  return [data, setData]
}

export default useLocalState
```