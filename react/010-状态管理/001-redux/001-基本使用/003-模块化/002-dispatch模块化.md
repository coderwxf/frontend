在`/src/store`目录下，新建文件夹`actions`

现在新目录结构如下:

```shell
.
├── actions
│   ├── countAction.js
│   ├── index.js
│   └── userInfoAction.js
├── consts.js
├── index.js
└── reducers
    ├── countReducer.js
    ├── index.js
    └── userInfoReducer.js
```



`countAction.js`

```js
import { INCREMENT, DECREMENT } from '../consts'

export const countAction = {
  // 这些生成实际行为对象的方法被称之为 Action Creator(动作创建函数)
  // 之所以被定义为方法，目录是为了方便通过组件传入参数
  increment(step) {
    return {
      type: INCREMENT,
      payload: {
        step
      }
    }
  },
  decrement(step) {
    return {
      type: DECREMENT,
      payload: {
        step
      }
    }
  }
}
```



`userInfoAction.js`

```js
import { CHANGE_NAME, CHANGE_AGE } from '../consts'

export const userInfoAction = {
  changeName(name) {
    return {
      type: CHANGE_NAME,
      payload: {
        name
      }
    }
  },
  changeAge(age) {
    return {
      type: CHANGE_AGE,
      payload: {
        age
      }
    }
  }
}
```



`actions/index.js`

```js
import { countAction } from './countAction'
import { userInfoAction } from './userInfoAction'

// action模块化
const actions =  {
  count: countAction,
  userInfo: userInfoAction
}

export default actions
```



`countReducer.js`

```js
import { INCREMENT, DECREMENT } from '../consts'

const countReducer =  (state = { count: 0 }, action) => {
  switch(action.type) {
    case INCREMENT:
      return {
        ...state,
        // 使用参数
        count: state.count + action.payload.step
      }
    case DECREMENT:
      return {
        ...state,
        count: state.count + action.payload.step
      }
    default:
      return state
  }
}

export default countReducer
```



`userInfoReducer.js`

```js
import { CHANGE_NAME, CHANGE_AGE } from '../consts'

const userInfoReducer = (state = { name: 'Klaus', age: 23 }, action) => {
  switch(action.type) {
    case CHANGE_NAME:
      return  {
        ...state,
        name: action.payload.name
      }
    case CHANGE_AGE:
      return {
        ...state,
        age: action.payload.age
      }
    default:
      return state
  }
}

export default userInfoReducer
```



`store/index.js`

```js
import { createStore } from 'redux'
import rootReducer from './reducers'
import actions from './actions'

const store = createStore(rootReducer)

export { actions }
export default store
```



`App.jsx`

```jsx
import React, { memo, useEffect, useState } from 'react'
import store, { actions } from './store'

const App = memo(() => {
  const [_, setUpdate] = useState(0)

  const { count: { count } } = store.getState()

  useEffect(() => {
    store.subscribe(() =>  {
      setUpdate(new Date())
    })
  }, [])

  return (
    <>
      <div>{count}</div>
    	{/* 
    		actions.count.increment(5) 生成的是action对象
    		所以依旧需要使用store.dispatch来进行派发
      */}
      <button onClick={() => store.dispatch(actions.count.increment(5))}>+5</button>
      <button onClick={() => store.dispatch(actions.count.increment(-5))}>-5</button>
    </>
  )
})

export default App
```

