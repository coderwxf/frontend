React组件分类

- 函数组件
  - 不具备“状态、ref、周期函数”等内容，第一次渲染完毕后，无法基于组件内部的操作来控制更新，因此称之为静态组件！
  - 但是具备自定义属性，父组件可以控制其更新渲染！
  - 这是未来的重点，函数逐渐取代类！
  - 基于FP（函数式编程）思想设计，提供更细粒度的逻辑组织和复用！

- 类组件
  - 具备“状态、ref、周期函数、属性、插槽”等内容，可以灵活的控制组件更新。基于钩子函数也可以灵活控制不同阶段处理不同的事情！
  - 这是当前最主流的组件构建模式！
  - 基于OOP（面向对象编程）思想设计，更方便实现继承等！

React Hooks 组件，就是基于 React 中新提供的 Hook 函数，可以让函数组件动态化！



Hook函数概览

Hook 是 React 16.8 的新增特性！并且只适用于函数组件中。

- 基础 Hook
  - useState 使用状态管理
  - useEffect 使用周期函数
  - useContext 使用上下文信息

- 额外的 Hook
  - useReducer 结合redux处理逻辑，管理复杂的状态和逻辑
  - useCallback 快捷处理函数依赖关系
  - useMemo 快捷处理依赖数据
  - useRef 使用保存DOM
  - useImperativeHandle 配合forwardRef 使用，控制实例
  - useLayoutEffect 类似useEffect，但会在所有的DOM变更之后同步调用

- 自定义 Hook



类组件

1. **语法冗长**：类组件的语法相对繁琐，需要编写更多的模板代码。
2. **this 关键字**：需要正确绑定 `this`，可能导致困惑和错误。
3. **代码复用困难**：类组件之间的逻辑复用通常依赖高阶组件或 render props，可能增加复杂性

函数组件

1. **简洁语法**：函数组件通常更简洁，代码更易读易写。
2. **无 this 绑定问题**：不需要处理 `this` 绑定的问题。
3. **更好的逻辑复用**：通过自定义 Hooks，可以更方便地复用逻辑



HOC -> withXxx -> withAuth`、`withRouter

HOOK -> useXxxx -> useState、 useEffect



React 的 Hook 函数必须在函数组件的顶层调用，不能在异步函数、循环、条件语句或嵌套函数中调用。这是因为 Hooks 的调用顺序必须保持一致，以确保 React 能够正确地管理组件的状态和生命周期。

具体来说，遵循以下规则可以确保 Hooks 正常工作：

1. **只在函数组件或自定义 Hook 中调用 Hooks**：不要在普通的 JavaScript 函数中调用 Hooks。
2. **保持 Hooks 的调用顺序一致**：不要在条件或循环中调用 Hooks。确保每次渲染时 Hooks 的调用顺序相同。


```jsx
import { useState } from 'react'

export default function Child(props) {
  // useState -> 在函数组件中使用状态
  // const [状态， 状态更新函数] = useState(初始值)
  const [count, setCount] = useState(0)
  
  // 组件更新时，状态count是更新后的状态 「 只有第一次才使用初始值 」
  // 但increment和setCount每次更新后，获取的都是新的函数，并不对之前的函数进行复用
  const increment =  () => setCount(count + 1)

  return (
    <>
      <div>{ count }</div>
      {/*
        1. setCount => 更新状态，并通知视图更新
        2. 函数组件不存在实例，所以不存在this
      */}
      <button onClick={increment}>increment</button>
    </>
  )
}
```

![image.png](https://s2.loli.net/2024/09/30/vzXB6MHQPVqJo35.png) 

每调用一次类组件，生成一个实例

类组件更新，不会产生新的实例，而是执行生命周期，重新执行render函数，并进行dom-diff



函数组件的每一次渲染（或者是更新），都是把函数（重新）执行，产生一个全新的“私有上下文”！
内嵌的代码也需要重新执行

+ 这与类组件更新机制不同（这些函数的作用域（函数执行时）上级上下文），是每一次执行DEMO产生的闭包）
+ 每一次执行DEMO函数，都会useState建立新的执行
+ 执行useState，只有第一次，设置的初始值会生效，其余以后再执行，获取的状态都是更新前的状态值「而不是初始值」
返回的状态修改的方法，每一次都是返回一个新的



在 React 函数组件中：

- **状态管理**: 状态由 `useState` 钩子管理，并保存在 React 内部的一个特殊地方。这使得即使组件函数重新执行，状态也能保持不变。

  => 实际的状态数据是保存在 React 内部的一个数据结构中，与组件实例绑定

  

- **函数执行**: 除了状态管理，组件的其余部分就是普通的 JavaScript 函数执行流程。每次组件渲染时，函数都会重新执行，生成新的执行上下文。

  => 内部事件处理函数，状态更新函数都是全新的，而他们的上级上下文都是全新的函数组件执行上下文

这种机制确保了组件的状态在多次渲染中能够持久化，而不是每次都重置为初始值。



```js
let _state; // 实际状态被存储在与vdom相关的一个数据结构中
function useState(initialValue) {
  	// null会被视为有效初始值
    if (typeof _state === "undefined") {
			_state = typeof initialValue === 'function' ? initialValue() : initialValue;
    }
		
  	// 所以状态更新函数每次都是全新的，并不复用
    return [_state, function setState(value) {
       newState = typeof value === 'function' ? value(_state) : value        
      
        // 如果新状态值和旧状态值完全一致，则停止更新「 没有更新必要, 类似于SCU的功能 」
      	if (Object.is(newState, _state)) return
      
        _state = newState;
        // 通知视图更新
      	// 1. 重新执行函数组件，生成新的vdom
        // 2. 进行DOM-DFF 获取Patch包
      	// 3. UI更新
    }];
}

let [num, setNum] = useState(0);
```



```jsx
const Demo = function Demo() {
    let [num, setNum] = useState(0);

    const handle = () => {
        setNum(100);
        setTimeout(() => {
          // 根据作用域链 => 定时器 -> handle -> Demo「 初次渲染 」
	        console.log(num); // 0 
        }, 2000);
    };

    return <div className="demo">
        <span className="num">{num}</span>
        <Button type="primary"
                size="small"
                onClick={handle}>
            新增
        </Button>
    </div>;
}

export default Demo;
```

![image.png](https://s2.loli.net/2024/09/30/njXqDvoWM5KtJEy.png) 



```jsx
import { useState } from 'react'

export default function Child(props) {
  const [point, setPoint] = useState({
    x: 0,
    y: 0
  })

  function changePoint() {
    // 推荐 => 一个状态对应一个useState => 因为useState并不支持部分状态更新 => 需要自己手动实现
    setPoint({
      ...point,
      x: 20
    })
  }

  return (
    <>
      <div>{ JSON.stringify(point) }</div>
      <button onClick={changePoint}>click</button>
    </>
  )
}
```

```js
const point1 = {x: 10}

const point2 = Object.assign(point, {
  x: 20
})

// Object.assign 不能触发 组件更新 => 直接修改第一个参数对象本身，并会生成全新的对象
console.log(point1 === point2) // => true
```





在 React18 中，我们基于 useState 创建出来的“修改状态的方法”，它们的执行也是异步的。
原理：等同于类组件中的 this.setState
+ 基于异步操作 & 更新队列，实现状态的批处理
在任何地方修改状态，都是采用异步编程的。



```js
const handle = () => {
  // 也可以传入函数，作用和setState传入函数的作用完全一致
  setX(prevX => prevX + 1)
  setX(prevX => prevX + 1)
  setX(prevX => prevX + 1)
}
```

```jsx
// 传入参数，计算初始值 -> 只会在第一次渲染时计算
let [x, setX] = useState(() => props.x + props.y);
```

















---

动画

useId