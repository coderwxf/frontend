Redux 是一个全局状态管理工具，可以帮助我们在 React 组件之间共享数据，使得组件之间的通信变得更简单。

Redux 是一个独立的状态管理库，能够在多个框架中使用，如 Vue、React 等。在 React 中，通常结合 `react-redux` 来简化 Redux 的使用。

`Redux Toolkit (RTK)` 是 Redux 和 react-redux 的简化工具包，提供了更加简洁和现代化的语法，方便开发者更快速地配置和管理 Redux。

一般情况下，仅推荐将需要在多个组件之间复用的状态存入 Redux 中，并不建议将所有的状态信息都存入 Redux。对于简单的父子组件关系，优先考虑使用 props 进行状态传递。当组件之间的关系较远或状态共享较复杂时，才考虑使用 Redux 来管理状态。



一般情况下，redux相关的文件会被存放到`src/store`目录下



下面是如何使用 Redux 的几个简单步骤：

1. **创建全局状态容器（Store）**： Store 是一个用来存储应用程序中所有状态的容器。
2. **全局注入store**: 方便在所有组件中获取并使用store对象
3. **获取数据**： 用 `store.getState()` 方法获取当前存储在 Store 里的状态。
4. **订阅状态变化**： 使用 `store.subscribe()` 方法，可以在状态变化时执行某些操作，比如更新界面。
5. **更新状态**： 使用 `dispatch` 方法可以更新状态，当 Store 更新时，订阅的回调函数就会让组件重新渲染。



## store的不可变性

不能直接修改store，而是应该通过`dispatch`派发一个行为对象 (action)，让`reducer`来修改状态

`reducer`被要求是一个纯函数，接收store并返回新的store对象

```js
import { createStore } from 'redux';

// reducer -> 更新store的纯函数「 管理员 」  enhance -> 中间件
const store = createStore(reducer, enhancer);
export default store;
```

这么做的目的是确保操作store都是产生一个全新的store，而不会修改旧store

Redux DevTools 能够更好地追踪每次状态的变化，并支持操作回溯（时间旅行调试）



### 不直接修改store的好处

1. 全局状态修改和使用组件解耦，方便修改逻辑的复用
2. 状态修改都统一由reducer组件，组件仅仅只是派发任务 => 方便后期维护和扩展
3. reducer是纯函数 => 状态的修改都是可预测的 => 因为不存在任何的副作用，确定的输入一定有确定的输出
4. 方便实现redux devtool



### 行为对象

执行一次`dispatch`方法的本质 就是执行一次`reducer`方法并获取新的`store`

```js
// dispatch方法的参数被称之为action对象 => 行为对象
// 行为对象必须存在type属性 => 行为标识
store.dispatch({
  type: 'INCREMENT', // 行为标识会在组件和redux等多处被使用，可以定义为常量并抽离
  step: 10 // 其余值表示参数 => 随便传
})
```



## reducer

```js
// consts.js
export const INCREMENT = 'INCREMENT'
export const DECREMENT = 'DECREMENT'
```

```js
import { createStore } from 'redux'
import { INCREMENT, DECREMENT } from './consts'

const initalState = {
  count: 0
}

function reducer(state = initalState, action) {
  // 根据行为标识做不同的事情
  switch (action.type) {
    case INCREMENT:
      // 获取参数并执行派发事件
      return {
        ...state,
        count: state.count + action.step ?? 1
      }
    case DECREMENT:
      return {
        ...state,
        count: state.count - 1
      }
    default:
      // 原封不动的返回参数，不违反纯函数原则
      return state
  }
}

const store = createStore(reducer)
export default store
```



### 首次执行

`redux`内部会默认执行一次派发, 用于初始化状态

```js
dispatch({
  type: <hash值>
})
```

此时在`reducer`中没有任何行为标识与其匹配，会执行默认逻辑，也就是`default`中的逻辑

又因为初始`state`值是`undefined`，所以此时`state`会等于`initalState`



### 后续执行

根据行为对象中的行为标识，执行不同的逻辑代码块

如果行为标识不存在，则什么都不处理，将state原封不动的返回



## 全局注入

store在多个组件中都需要使用，所以作为全局context传递给每一个组件，然后组件再通过上下文对象去获取即可

直接在组件中引入store并使用也是可以的，但是使用上下文对象可以实现组件和全局状态之间的解耦

```js
import { createContext } from "react";
export default createContext();
```

```jsx
<ReduxContext.Provider value={store}>
  <App />
</ReduxContext.Provider>
```



## 类组件中使用

```js
import { PureComponent } from 'react'
import StoreContext from './contetxt/StoreContext'

export default class ClassChild extends PureComponent {
  static contextType = StoreContext

  state = {
    dateStamp: 0
  }

  render() {
    const store = this.context
    // 获取store state 并解构
    const { count } = store.getState()

    return (
      <h1>class: { count }</h1>
    )
  }
  
  // 事件更新方法不需要重复添加 => 只要在组件渲染完成后添加一次即可
  
  // 不可以将这部分逻辑添加到构造函数中
  // => 因为context对象是在render函数执行前才会被初始化 「 和state以及props一起初始化 」
  // => 所以在构造方法中，this.context的值是undefined
  componentDidMount() {
    const store = this.context

    // subscribe的回调会被插入store state的事件池中
    // 当状态改变时，会依次执行事件池中的回调
    store.subscribe(() => {
      this.setState({
        // 每次执行都会获取一个全新的时间戳，更新状态
      	// 组件重新渲染 => 重新执行render方法，获取最新redux state
      	// 所以这里的本质是需要更改状态触发组件重新渲染，所以使用this.forceUpdate()其实也是一样的
        dateStamp: +new Date()
      })
    })
  }
}
```



## 函数组件中使用

```jsx
import { useContext, useState } from 'react'
import StoreContext from './contetxt/StoreContext'

export default function FunctionChild() {
  const store = useContext(StoreContext)
  const { count } = store.getState()
  const [_, forceUpdate] = useState(0)
  
  store.subscribe(() => {
    forceUpdate(+new Date())
  })

  return (
    <h1>
      function: { count }
    </h1>
  )
}
```

在函数组件中，`subscribe`还可以修改为

```js
import { useContext, useEffect, useState } from 'react'
import StoreContext from './contetxt/StoreContext'

export default function FunctionChild() {
  const store = useContext(StoreContext)
  const { count } = store.getState()

  const [num, setNum] = useState(0)

  useEffect(() => {
    // const unSubscribe = subscribe(callback)
    // 1. callback中的回调会被插入redux store的事件池中
    // 2. subscribe方法的返回值unSubscribe也是一个函数
    //    执行unSubscribe方法会从redux store事件池中将callback移除
    const unSubscribe = store.subscribe(() => {
      setNum(num + 1)
    })

    // 每次触发副作用前，先将上一次的callback从事件池中移除，避免callback的重复添加
    return () => unSubscribe()
  }, [num])

  return (
    <h1>
      function: { count }
    </h1>
  )
}
```



## 事件派发

```js
import { useContext } from 'react'
import StoreContext from './contetxt/StoreContext'
import { INCREMENT, DECREMENT } from './store/consts'

export default function Action(props) {
  const { dispatch } = useContext(StoreContext)

  return (
    <>
    	{/* 派发事件并传递参数 */}
      <button 
    		onClick={ () => dispatch({ type: INCREMENT, step: 10})}
			>increment</button>
      <button onClick={ () => dispatch({ type: DECREMENT }) }>decrement</button>
    </>
  )
}
```



## 源码实现 - 伪代码

```js
/* 实现redux的部分源码 */
export function createStore(reducer) {
  if (typeof reducer !== 'function') {
    throw new TypeError('The reducer must be a function')
  }

  // state 状态对象 => 初始值 undefined
  let state
  // 事件池数组
  let listeners = []

  function dispatch(action) {
    // 确保行为对象action 是一个纯对象
    if (Object.prototype.toString.call(obj) !== "[object Object]") {
      throw new TypeError('The action must be a plain object.')
    }

    if (!action.type) {
      throw new TypeError('The action must have a type property.')
    }

    // 状态更新逻辑交给使用者编写
    // redux所需要做的仅仅只是调用reducer并传入state和action
    // 同时将reducer的返回值替换现有state即可
    state = reducer(state, action)

    // 通知观察者
    listeners.forEach(listener => listener())

    // dispatch方法的返回值是行为对象 「 一般不用而已 」
    return action
  }

  // 派发一个不可能被reducer匹配的行为对象 => 初始化redux state
  dispatch({
    type: Symbol()
  })

  return {
    getState: () => state,
    dispatch,
    subscribe(callback) {
      if (typeof callback !== 'function') {
        throw new TypeError('The subscribe parameter must be a function.')
      }

      let index = listeners.findIndex(listener => listener === callback)

      // 如果更新回调函数已经在事件池中，就不需要重复添加 「 判断依据是更新回调函数的引用地址，简称事件指针或事件引用 」
      if (index === -1) {
        listeners.push(callback)
        index = listeners.length - 1
      }

      // 返回一个从事件池中，移除方法的函数
      return () => listeners.splice(index, 1)
    }
  }
}
```

其中，为state赋予初始值使用的是

```js
dispatch({
  type: Symbol()
})
```

但redux中，为了兼容ES6之前语法。实际并不是这么实现的

```js
function randomString() {
   // toString(n) => 转换为n进制数值格式字符串
   // n的取值范围是 [2, 36] => 之所以是36，是0-9「10个数值」 + a-z「 26个字母 -> 进制中字母不区分大小写 」
  
   // str.substring(start, end) => 截取子串, start是开始索引，end是结束索引 => [start, end)
   // end不传默认值是str.length => 即截取到最后
   return Math.random().toString(36).substring(7).split('').join('.');
};

dispatch({
	type: `@@redux/INIT` + randomString()
})
```



### 纯对象 和 原生对象

1. **原生对象**：

   - `typeof obj // object`：这表示 `obj` 是一个对象类型。
   - `obj` 不是数组也不是 `null`：这意味着 `obj` 是一个普通的对象，而不是特殊类型（如数组或 `null`）。

2. **纯对象（Plain Object）**：

   - 是一种特殊的原生对象。

   - 纯对象只有 `Object.prototype` 或 `null` 作为原型，没有其他自定义的原型链

     - 这意味着纯对象是通过 `new Object()` 或对象字面量 `{}` 创建的，或通过`Object.create(null)`创建出来的

       => `Object.create(null)` 创建的顶层对象可以被认为是一种特殊的纯对象

     - 纯对象只有`Object`或自身添加的属性和方法，没有因为继承，没有通过原型链继承的其他属性或方法

   - 判断对象是否是纯对象的方式

     ```js
     // 简单写法
     Object.prototype.toString.call(obj) === "[object Object]" || Object.getPrototypeOf(obj) === null
     ```

     ```js
     // 完整写法
     function isPlainObject(obj) {
       // 只要是普通对象 
       // => Object.prototype.toString.call(obj)返回值都是 [object Object]
       if (Object.prototype.toString.call(obj) !== "[object Object]") {
         return false
       }
     
       const proto = Object.getPrototypeOf(obj)
       
       // 通过Object.create(null) 创建的 纯对象
       if (!proto) {
         return true
       }
     
       // new Object 或 字面量创建出的对象
       // => 对象.hasOwnProperty(属性) => 属性是否是对象自身的属性
       // => Reflect.has(属性， 对象) <=> 属性 in 对象 => 属性在对象自身或在对象的原型链上
       if (proto.hasOwnProperty('constructor') && proto.constructor === Object) {
         return true
       }
     }
     ```
     
     

### 观察者设计模式

观察者设计模式的核心思想就是让多个观察者对象能够订阅 「 监控、监听 」 一个被观察的对象（通常称为“主题”或“被观察者”）。

当被观察者的状态发生变化时，它会通知所有订阅的观察者，并触发它们的回调方法。

观察者最常见于基于事件驱动的编程语言中



具体实现通常包括以下几个步骤：

1. **注册观察者**：观察者对象向被观察者注册，表示它们希望接收通知。
2. **维护观察者列表**：被观察者维护一个观察者列表（或事件队列），记录所有已注册的观察者。
3. **状态改变通知**：当被观察者的状态发生变化时，它会遍历观察者列表，通知每一个观察者，并调用它们的回调方法。
4. **注销观察者**：观察者可以选择注销，以停止接收通知。



原生redux和事件总线就是观察者模式的具体实现。

它们维护者一个事件队列，观察者会将当被观察对象改变后需要执行的逻辑封装为方法存入事件队列中，

当被观察对象发生更新时，会依次执行事件队列中的各个回调

```js
function subscribe(callback) {
  // 将回调函数添加到监听器列表
  listeners.push(callback);
}

// 通知所有监听器
listeners.forEach(listener => listener());
```



`InteractionObserver`是另一种观察者模式的实现，当观察者需要订阅被观察者时，会将观察者存入观察者列表

被观察者自身维护者一个事件，当被被观察者发生更新时，会执行被观察者自身维护的事件，并将观察者列表作为参数传入

```js
const callback = (entries, observer) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element is in view:', entry.target);
    } else {
      console.log('Element is out of view:', entry.target);
    }
  });
};
```

