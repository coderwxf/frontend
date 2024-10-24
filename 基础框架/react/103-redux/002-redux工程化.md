工程化redux => 使用redux时分了模块

1. 减少redux状态的命名冲突 => 多个模块中可以存在同名模块
2. 减少代码合并冲突
3. 逻辑拆分，方便后期维护和更新



## 目录结构

```shell
store # redux代码根目录
  ├── action-types.js # 行为标识文件
  ├── actions # 行为对象模块目录
  │   ├── counter.js # 行为对象模块目录
  │   ├── index.js # 行为对象入口文件 => 合并行为对象
  │   └── userInfo.js
  ├── index.js # redux代码入口文件
  └── reducers # reducer模块目录
      ├── counter.js # reducer模块
      ├── index.js # reducer模块入口文件 => 合并reducer模块
      └── userInfo.js
```



## reducer模块

```js
// @reducer 模块文件
const initalState = {
  name: 'Klaus',
  age: 23
}

// 每个模块reducer有自己的state和action
export default function reducer(state = initalState, action) {
  switch(action.type) {
    case 'ADD_AGE':
     return {
        ...state,
        age: state.age + (action.payload?.age ?? 1)
     }
    default:
      return {
        ...state
      }
  }
}
```

```js
// @reducer 入口文件 => 合并reducer模块
import { combineReducers } from 'redux'
import counterReducer from './count'
import userInfoReducer from './userInfo'

/* 
  通过combineReducers 进行模块合并
  合并后为一个大的state =>
  {
    counter: { .... },
    userInfo: { .... }
  }
*/
export default combineReducers({
  counter: counterReducer,
  userInfo: userInfoReducer
})
```

```js
// 使用 --> 使用redux的组件
const store = useContext(StoreContext)
// 解构出 state模块
const { userInfo } = store.getState()
// 再从 state模块中 解构出 实际所需要使用的状态
const { name, age } = userInfo
```



## 行为标识

在派发时，会把事件池中所有匹配到的行为对象全部派发，而不是仅仅只执行第一个事件逻辑

这样操作很容易导致代码逻辑混乱，所以需要在使用工程化redux的时候，需要确保行为标识是唯一的

```js
// actions-type.js => 命名任意，最为常见的事actions-type

// 命名规则推荐为 模块名_行为标识 => 因为是常量，所以全大写
export const VOTE_SUP = "VOTE_SUP";
export const VOTE_OPP = "VOTE_OPP";

export const PERSONAL_SUP = "PERSONAL_SUP";
```

一般将所有行为标识定义在`actions-types`这一个文件中，且必须保证常量名和常量值保持一致

原因

1. 在一个文件中，变量名冲突，IDE会直接报错

   此时变量名和变量值保持一致，就可以确保实际派发的行为标识是唯一的

2. 使用以下方式进行使用

   ```js
   import * as types from '../action-types';
   ```

   使用时，直接通过`types.xxx`的形式来使用行为标识

   好处是:

   1. 在IDE中，存在代码提示
   2. 可以有效避免因为人为粗心，导致的代码错误



## 行为对象模块化

```js
// actions/userInfo.js
import * as TYPES from '../action-types';

// 行为对象模块化本质就是生成一个由action creator组成的对象
// action creator => 一个返回 action 对象的函数
const userInfoAction = {
  addAge(step) {
    return {
      type: types.USERINFO_INCREMENT,
      payload: {
        step: step ?? 1
      }
    }
  }
}

// 之所以分开导出 是为了便于以后看到变量名后见名知意，大致就知道该文件导出内容的主要作用是什么
// 也方便后期再countAction上进行二次扩展
export default userInfoAction
```

```js
// actions/index.js
import counterAction from './counter'
import userInfoAction from './userInfo'

// 手动实现action的模块化
const combineAction =  {
  counter: counterAction,
  userInfo: userInfoAction
}

export default combineAction
```

```jsx
{/* 实际使用 => dispatch(调用action creator) */}
<button onClick={ () => dispatch(combineAction.counter.addCount(10)) }>increment</button>
```



## 模拟实现counterReducers

combineReducers 是以s结尾的 ！！！=> 返回值是合并后的大reducer

```js
export function combineReducers(reducerMap = {}) {
  // 需要给大state一个默认值 => 空对象
  // redux 默认会在底层派发一次 => 此时 state[key]的值是undefined
  // 从而执行reducer模块的默认逻辑 => 从而实现大state的初始化 「 并且是模块化管理的 」
  return function(state = {}, action)  {
    let newState = {}

    Object.keys(reducerMap).forEach(key => {
      const reducer = reducerMap[key]
      newState[key] = reducer(state[key], action)
    })

    return newState
  }
}
```



### 存在的问题

#### 事件池是唯一的

Redux 的事件池是全局的，意味着每次状态改变时，所有订阅的回调函数都会被调用。这会导致以下问题：

- **不必要的渲染**：即使组件的状态没有更新，也可能会触发渲染，生成新的虚拟 DOM（VDOM），并进行 DOM-DIFF 操作。虽然最终没有实际的 DOM 更新，但生成 VDOM 和进行 DIFF 操作仍然消耗性能。
- **优化建议**：为了解决这个问题，建议组件实现 `shouldComponentUpdate` 或使用 `React.memo`，以避免在 `props` 和 `state` 未发生变化时重新生成 VDOM。



#### 状态可以直接修改

Redux 的 `getState` 方法返回的是当前状态的引用，而不是副本。这意味着状态可以被直接修改，这与 Redux 的不可变性原则相违背。

- **风险**：直接修改状态可能导致难以追踪的错误和不可预测的行为。

- **解决方案**：确保在使用状态时不直接修改它。使用 `redux-immutable` 或 `immer` 等库来帮助管理不可变数据。

  

#### 方法需要手动移除

在组件的生命周期中，订阅 Redux 状态变化的回调函数需要手动移除。

- **问题**：如果不手动移除订阅，可能会导致内存泄漏或组件卸载后仍然响应状态变化。
- **解决方案**：在组件卸载时（如 `componentWillUnmount`），确保调用 `unsubscribe` 方法来移除订阅。