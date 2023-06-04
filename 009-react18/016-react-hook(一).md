Hook 是 React 16.8 的新增特性，Hook本质上就是一系列的函数

 Hook可以让我们在不编写class的情况下使用state以及其他的React特性

Hook的出现基本可以代替我们之前所有使用class组件的地方

如果是一个旧的项目，你并不需要直接将所有的代码重构为Hooks，因为它完全向下兼容，你可以渐进式的来使用它, 例如原本的组件依旧使用class组件，而新的组件使用hooks

`Hook只能在函数组件中使用，不能在类组件，或者函数组件之外的地方使用`

简而言之，hook的编写比类组件更为的简单方便，同时为函数式组件提供了状态管理和生命周期等扩展性新功能

![image.png](https://s2.loli.net/2022/09/12/yMTwJjEYHc6frei.png)



## 初体验 - 计数器

### 使用类组件来实现

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			count: 0
		}
	}

	changeCount(step) {
		this.setState({
			count: this.state.count + step
		})
	}

	render() {
		const { count } = this.state

		return (
			<div>
				<h1>{ count }</h1>
				<button onClick={() => this.changeCount(1)}>+1</button>
				<button onClick={() => this.changeCount(-1)}>-1</button>
			</div>
		)
	}
}

export default App
```



### 使用函数式组件+hook来实现

```jsx
import React, { memo, useState } from 'react'

const App = memo(() => {
  // 一个标准的hook 接收的参数 会被作为对应状态的初始值
  // 返回值是一个数组
  // 数组的第一项值 为 对应的状态初始值 - 不设置默认为undefined
  // 数组的第二项值 为 用于设置对应状态的方法

  // 当使用hook后，对应的状态会交给react内部来进行管理
  // 所以再次执行函数式组件的时候，对应的状态值会使用上一次存储下来的对应状态值
  // 当执行设置状态值的方法(如setCounter)的时候，react会修改对应的状态值，并重新执行对应的render方法
  const [counter, setCounter] = useState(0)

  return (
    <div>
      <div>{ counter }</div>
      <button onClick={ () => setCounter(counter + 1) }>+1</button>
      <button onClick={ () => setCounter(counter - 1) }>-1</button>
    </div>
  )
})

export default App
```



通过上述示例，我们可以明显的得出`函数式组件结合hooks让整个代码变得非常简洁,并且再也不用考虑this相关的问题`

使用Hook有两条基本原则：

1. `只能在函数最外层调用 Hook`。不要在循环、条件判断或者子函数中调用
2. `只能在 React 的函数组件中调用 Hook`。不能在其他 JavaScript 函数中调用
3. `React允许我们在自定义Hook函数中去调用其它hooks方法，但不允许在普通函数中去调用其它hooks函数`
   + tips: 当前仅当一个函数的函数名类似于useXXX的时候，React会将该函数看成是自定义hook



## State Hook

State Hook的API就是 useState， useState会帮助我们定义一个 state变量，useState 是一种新方法，它与 class 里面的 this.state 提供的功能完全相同

一般来说，函数组件在退出后变量就会”消失”，所以state 中的变量实质上会被 React 保留

也就是说在第一次的时候，会生成对应的状态，react会将对应状态进行存储

在下次渲染函数式组件的时候，react会自动取出对应状态以供渲染

一但我们调用设置state的方法，react会重新执行对应的函数式组件

并使用最新的状态生成新的ReactElement对象，并重新进行渲染



所以，在渲染过程中，seState 总是返回给我们当前的 state，所以Hook 的名字总是以 use 开头，因为我们往往都是在使用对应的hook所存储的值



## Effect Hook

Effect Hook 可以让你来完成一些类似于class中生命周期的功能

类似于网络请求、手动更新DOM、一些事件的监听，都是React更新DOM的一些副作用(Side Effects), 所以对于完成这些功能的Hook被称之为 Effect Hook



通过useEffect的Hook，可以告诉React需要在渲染后执行某些操作

useEffect要求我们传入一个回调函数，在React执行完更新DOM操作之后，就会回调这个函数

默认情况下，无论是第一次渲染之后，还是每次更新之后，都会执行这个回调函数

```js
import React, { memo, useState, useEffect } from 'react'

const App = memo(() => {
  const [count, setCount] = useState(0)

  // useEffect要求我们传入一个回调函数，在React执行完更新DOM操作之后，就会回调这个函数
  // 只要函数组件被重新渲染，useEffect内部的回调就会被再次执行
  useEffect(() => {
    document.title = count
  })

  // 组件本职就是获取状态去渲染对应的组件
  // 所以和组件渲染之外的所有操作，例如网络请求，额外的dom操作，事件监听等都属于组件的副作用
  return (
    <div>
      <div>{ count }</div>
      <button onClick={ () => setCount(count + 1) }>+1</button>
    </div>
  )
})

export default App
```



### 清除机制

每个 effect 都可以返回一个清除函数，该清除函数会在函数组件每次更新之前或者组件被卸载之前被回调

使用effect的清除机制，可以将添加和移除订阅的逻辑放在一起，增加对应的内聚性

```js
useEffect(() => {
  console.log('执行了副作用函数')

  return () => {
    console.log('执行effect的清除机制')
  }
})
```



### 多次调用

effect hook可以被多次调用，effect对应的回调会按照定义的顺序被依次调用

通过多次调用effect hook可以将相关的逻辑单独封装成一个独立的hook，另一个相关的逻辑单独封装成另一个hook

```js
useEffect(() => {
  console.log('执行了副作用函数1')
})

useEffect(() => {
  console.log('执行了副作用函数2')
})
```



### 模拟生命周期函数

某些代码我们只是希望执行一次即可，类似于componentDidMount和componentWillUnmount中完成的事情;(比如网 络请求、订阅和取消订阅)

对于这些函数多次只能可能会导致对应的性能问题，此时effect hook就给我们提供了第二个参数用于控制在什么情况下，需要执行对应的hook函数

```js
import React, { memo, useState, useEffect } from 'react'

const App = memo(() => {
  const [count, setCount] = useState(0)

  // 参数一 -> 回调函数，需要执行的副作用函数
  // 参数二 -> 数组，只有当数组中的状态发生改变的时候，才会重新执行对应的副作用函数
  //       -> 如果不传第二个参数 就表示 只要 函数组件重新渲染了 那么对应的副作用函数就需要被重新执行
  //       -> 如果第二个参数是空数组， 那么就表示只有初次渲染的时候才会执行对应的副作用函数 -- 用于模拟componentDidMount
  //          而对应的副作用函数的返回值函数 也只有在函数组件被卸载的时候才会被再次执行 -- 用于模拟 componentWillUnmount
  // 所以可以使用useEffect来模拟生命周期函数，而useEffect本身比生命周期函数更为的强大
  useEffect(() => {
    console.log('componentDidMount')

    return () => {
      console.log('componentWillUnmount')
    }
  }, [])

  return (
    <div>
      <div>{ count }</div>
      <button onClick={ () => setCount(count + 1) }>+1</button>
    </div>
  )
})

export default App
```



## Context Hook

Context Hook允许我们通过Hook来直接获取某个Context的值

当context依赖的值发生改变的时候，对应的组件也会被重新渲染

```js
import { createContext } from "react";

const UserContext = createContext()
const ThemeContext = createContext()

export {
  UserContext,
  ThemeContext
}
```

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { ThemeContext, UserContext } from './context'

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
   <ThemeContext.Provider value={{ color: 'red', fontSize: '12px' }}>
      <UserContext.Provider value={{ name: 'Klaus', age: 23 }}>
        <App />
      </UserContext.Provider>
   </ThemeContext.Provider>
);
```

```jsx
import React, { memo, useContext } from 'react'
import { ThemeContext, UserContext } from './context'

const App = memo(() => {
  const user = useContext(UserContext)
  const theme = useContext(ThemeContext)

  return (
    <div>
      <div>{ user.name } - { user.age }</div>
      <div>{ theme.color } - { theme.fontSize }</div>
    </div>
  )
})

export default App
```



## useReducer

useReducer仅仅是useState的一种替代方案，并不能完全取代redux

`不使用useReducer`

```jsx
import React, { memo, useState } from 'react'

const App = memo(() => {
  const [count, setCount] = useState(0)

  return (
    <div>
      <div>count: { count }</div>
      <button onClick={ () => setCount(count + 1) }>+1</button>
      <button onClick={ () => setCount(count - 1) }>-1</button>
    </div>
  )
})

export default App
```



`使用useReducer`

```jsx
import React, { memo, useReducer } from 'react'

// useReducer函数 本质上就是将所有useState的操作集中在了一起
// 例如 useState(age), useState(user) -> useState({ age, user })
// 并且当useReducer结合context的时候，也可以进行一些数据的共享
function reducer(state, action) {
  switch(action.type) {
    case 'increment':
      return { ...state, count: state.count + 1 }
    case 'decrement':
      return { ...state, count: state.count - 1 }
    default:
      return state
  }
}

const App = memo(() => {
  // useReducer
  //  参数一 --- reducer函数
  //  参数二 --- 初始值
  const [state, dispatch] = useReducer(reducer, {
    count: 0
  })

  return (
    <div>
      <div>count: { state.count }</div>
      <button onClick={ () => dispatch({ type: 'increment' }) }>+1</button>
      <button onClick={ () => dispatch({ type: 'decrement' }) }>-1</button>
    </div>
  )
})

export default App
```



## useCallback

useCallback实际的目的是为了进行性能的优化

useCallback会返回一个函数的 memoized(记忆的) 值

在依赖不变的情况下，多次定义的时候，返回的值是相同的

通常使用useCallback的目的是不希望子组件进行多次渲染，并不是为了函数进行缓存

所以如果我们需要将一个函数作为props传递给子组件的时候，最好将该函数使用useCallback进行包裹后再进行传递

```jsx
import React, { memo, useCallback, useState } from 'react'
import Cpn from './Cpn'

const App = memo(() => {
  const [count, setCount] = useState(0)
  const [ age, setAge ] = useState(18)

  // 对于函数组件 每次组件被重新渲染的时候，对应的函数会被重新定义
  // 所以对于传入useCallback的回调函数 或者 对于那些没有使用useCallback的函数
  // 在每次函数组件被重新渲染的时候，都会被再次定义一次
  // 而对于使用了useCallback的函数
  //  参数一 - 需要执行的回调函数
  //  参数二 - 依赖项 只有当依赖项中的数值发生了改变，才会将useCallback的回调函数重新赋值给对应的返回值(这里是increment)
  //  如果依赖项并没有发生任何的改变 会复用上一次的回调函数，这次的回调函数依旧会被定义，但是并不会被采用
  const increment = useCallback(() => {
    setCount(count + 1)
  }, [count])

  return (
    <div>
      <div>count: { count }</div>
      <div>age: { age }</div>
      <Cpn increment={increment} />
      {/*
        在这里 修改了age的值 但是并没有修改count的值
        所以此时传入给子组件Cpn的increment函数并不会被重新定义
        所以子组件中的props并没有发生改变
        所以此时子组件Cpn并不需要被再次渲染
      */}
      <button onClick={() => setAge(23)}>change age</button>
    </div>
  )
})

export default App
```



```jsx
// 当useCallback的第二个参数是空数组的时候，表示函数在第一个被赋值后，不在使用新定义的函数
// 此时count会从0转变为1，之后无论如何修改count的值，count的值都是不会再次改变的
// 因为在进行setCount(count + 1)的时候，其内部的count的值因为闭包的原因 所捕获到的count的值一定是0
// 所以对于该函数而言，每次所执行的操作都是 0 + 1
// 这种情况被称之为闭包陷阱
const increment = useCallback(() => {
  setCount(count + 1)
}, [])
```



但是实际上，在上述案例中，我们仅仅只是修改了count的值，对于increment函数，其内部逻辑本质上是一致的，并没有发生改变，所以没有必要再重新创建一个新的increment函数给子组件，从而避免子组件Cpn的不必要渲染

```jsx
import React, {
  memo,
  useCallback,
  useState,
  useRef
} from 'react'
import Cpn from './Cpn'

const App = memo(() => {
  const [count, setCount] = useState(0)

  // 无论函数组件被渲染几次，useRef返回的永远都是同一个值
  // 当count的值发生改变的时候，下述逻辑会被重新执行
  // 所以countRef的current属性所对应的值永远保存着最新的count值
  const countRef = useRef()
  // 和vue的ref使用value存储对应的值类似
  // react中的ref使用current存储对应的值
  countRef.current = count

  // 因为countRef所指向的引用值永远都是一致的
  // 所以并不需要根据依赖生成新的increment函数
  // 这样就确保了在不生成新的increment函数的同时
  // 可以使用正确的count值
  const increment = useCallback(() => {
    setCount(countRef.current + 1)
  }, [])

  return (
    <div>
      <div>count: { count }</div>
      <Cpn increment={increment} />
    </div>
  )
})

export default App
```



## useMemo

useMemo实际的目的也是为了进行性能的优化

useMemo返回的也是一个 memoized(记忆的) 值

在依赖不变的情况下，多次定义的时候，返回的值是相同的

```jsx
// useCallback是对传入的函数进行缓存
// useMemo是对回调函数的返回值进行缓存
// useCallback(fn, deps) === useMemo(() => fn, deps)
```



```js
import React, { memo, useMemo, useState } from 'react'

function calcTotal(num) {
  let i = 0;
  let arr = 0;

  for (i = 0; i <= num; i++) {
    arr += i
  }

  return arr
}

const App = memo(() => {
  const [count, setCount] = useState(0)

  return (
    <div>
      <div>{ count }</div>
      {/*
        只有当count的值发生改变的时候，才会重新计算calcTotal
        此时该功能类似于vue中的计算属性
       */}
      <div>{ useMemo(() => calcTotal(count), [count]) }</div>
      <button onClick={ () => setCount(count + 1) }>+1</button>
    </div>
  )
})

export default App
```



```jsx
import React, { memo, useMemo } from 'react'
import Cpn from './Cpn'

const App = memo(() => {
  // 如果组件重新渲染，对于复杂类型数据，会重新创建一个对应的值
  // 所以可以使用useMemo对其进行包裹，以避免重新创建一个相同的复杂类型值
  // 从而可以避免Cpn组件被多次无效渲染
  const userInfo = useMemo(() => ({
    name: 'Klaus',
    age: 23
  }), [])

  return (
    <div>
      <Cpn userInfo={userInfo} />
    </div>
  )
})

export default App
```