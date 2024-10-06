context => 上下文 

+ 适用于 嵌套层级较深的组件之间进行通信 => 类似于`provide/inject`
+ context可以理解为就是父子组件之间公用的状态管理容器



创建上下文对象  => 一般位于`src/context`目录下，文件命名规则一般为`xxxContext.js`

```jsx 
import { createContext } from 'react'

// 创建上下文对象
export default createContext()
```



## 类组件

```jsx
import { PureComponent } from 'react'
import CountContext from './context/CountContext'

class Count extends PureComponent {
  render() {
    // 通过上下文对象的消费者使用状态
    // 使用方式是插槽 => 是一个方法，参数是提供的状态，返回值是需要渲染的UI
    return <CountContext.Consumer>
      {
         ({ count }) => <h1>{ count }</h1>
      }
    </CountContext.Consumer>
  }
}

class Action extends PureComponent {
  // 设置静态属性contextType后，可以将上下文提供的状态挂载到组件实例的context属性上
  // 1. 只有类组件才有
  // 2. 多个context嵌套时, 只能挂载一个 「 context是一个属性 」
  static contextType = CountContext

  componentDidMount() {
    console.log(this.context) // => {count: 0, increment: ƒ}
  }

  render() {
    return <button onClick={this.context.increment}>click me</button>
  }
}

export default class Child extends PureComponent {
  state = {
    count: 0
  }

  increment() {
    this.setState({
      count: this.state.count + 1
    })
  }

  render() {
    return (
      // 通过生产者的value属性，向后代组件提供状态
      <CountContext.Provider value={{
        count: this.state.count,
        // 方法绑定为箭头函数，用于修正this指向
        increment: () => this.increment()
      }}>
        <Count count={this.state.count} />
        <Action increment={() =>  this.increment()} />
      </CountContext.Provider>
    )
  }
}
```



## 函数组件

```jsx
import { useState, useCallback, memo } from "react"
import CountContext from './context/CountContext'

const Count = memo(function () {
  return (
    <CountContext.Consumer>
      {
        ({ count }) => <h2>{count}</h2>
      }
    </CountContext.Consumer>
  )
})

const Action = memo(function() {
  return (
    <CountContext.Consumer>
      {
        ({ increment })  => <button onClick={increment}>click me</button>
      }
    </CountContext.Consumer>
  )
})

export default function Child() {
  const [ count, setCount ] = useState(0)

  const increment = useCallback(() => {
    setCount(count + 1)
  }, [count])

  return (
    <>
      <CountContext.Provider value={{ count, increment }}>
        <Count count={count} />
        <Action increment={increment}  />
      </CountContext.Provider>
    </>
  )
}
```



```jsx
import { useState, useCallback, memo, useContext } from "react"
import CountContext from './context/CountContext'

const Count = memo(function () {
  // useContext的返回值就是祖先组件传递过来的上下文对象
  const { count } = useContext(CountContext)
  return <h2>{count}</h2>
})

const Action = memo(function() {
  const { increment } = useContext(CountContext)
  return <button onClick={increment}>click me</button>
})

export default function Child() {
  const [ count, setCount ] = useState(0)

  const increment = useCallback(() => {
    setCount(count + 1)
  }, [count])

  return (
    <>
      <CountContext.Provider value={{ count, increment }}>
        <Count count={count} />
        <Action increment={increment}  />
      </CountContext.Provider>
    </>
  )
}
```



## 嵌套使用

上下文的嵌套使用 => 类组件和函数组件使用方式一致

```jsx
import { PureComponent } from 'react'
import CountContext from './context/CountContext'
import ThemeContext from './context/ThemeContext'

function Cpn() {
  return (
    <ThemeContext.Consumer>
      {
        ({ theme }) => (
          <CountContext.Consumer>
            {
              ({ count }) => (
                <>
                  <h2>theme: { theme }</h2>
                  <h2>count: { count }</h2>
                </>
              )
            }
          </CountContext.Consumer>
        )
      }
    </ThemeContext.Consumer>
  )
}

export default class Child extends PureComponent {
  render() {
    return (
      <ThemeContext.Provider value={{ theme: 'dark' }}>
        <CountContext.Provider value={{ count: 20 }}>
          <Cpn />
        </CountContext.Provider>
      </ThemeContext.Provider>
    )
  }
}
```



## 默认值

```jsx
// @ThemeContext
import { createContext } from 'react'

export default createContext({
  theme: 'dark'
})
```

```jsx
// @Child.jsx
import ThemeContext from './context/ThemeContext'

function Cpn() {
  return <ThemeContext.Consumer>
    {
      ctx => <h2>{ ctx?.theme ?? 'default theme' }</h2>
    }
  </ThemeContext.Consumer>
}

export default function Child() {
  return (
    <>
      {/* 提供默认值 - 渲染结果 light */}
      <ThemeContext.Provider value={{ theme: 'light' }}>
        <Cpn />
      </ThemeContext.Provider>
     
      {/*
        不是ThemeContext的后代，如果使用ThemeContext消费，使用默认值 - 渲染结果: dark
      */}
      <Cpn />
     
      {/*
        没有提供value，且使用ThemeContext的后代
        视为没有提供value -> 提供给后代的value属性值是undefined - 渲染结果为 default theme
      */}
      <ThemeContext.Provider>
        <Cpn />
      </ThemeContext.Provider>
    </>
  )
}
```

