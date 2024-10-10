项目变大后，需要对reducer进行分模块处理

1. 减少代码冲突
2. 方便后期维护和管理
3. 减少redux store的状态冲突



## 思路

1. reducer进行分模块处理，一般位于`src/store/reducers`下
2. 在`reducers`下存在`index.js`导入所有模块reducer并合并为一个统一的`reducer`



## 基本实现

`模块reducer`

```js
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



`reducers/index.js`

```js
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



`使用`

```js
import { useContext, useState } from 'react'
import StoreContext from './contetxt/StoreContext'

export default function UserInfo() {
  const store = useContext(StoreContext)
  const { userInfo } = store.getState()
  const { name, age } = userInfo

  const [num, setNum] = useState(0)

  store.subscribe(() => {
    setNum(+new Date())
  })

  return (
    <h1>
      { name } --- { age }
    </h1>
  )
}
```

```js
import { useContext } from 'react'
import StoreContext from './contetxt/StoreContext'

export default function Action() {
  const { dispatch } = useContext(StoreContext)

  return (
    <>
      <button onClick={ () => dispatch({ type: 'INCREMENT', payload: { step: 10 } }) }>increment</button>
      <button onClick={ () => dispatch({ type: 'ADD_AGE' }) }>add age</button>
    </>
  )
}
```



## 存在的问题

事件池是唯一的，所以任何状态改变，事件池中对应方法会依次被回调并执行

这就导致很多组件状态即使没更新，也会因为加入事件池中的方法被执行，而生成全新的VDOM，并进行DOM-DIFF

虽然最终因为没有PATCH所以不会进行实际更新，但是DOM-DIFF时，VDOM是全量生成，所以依旧有损性能

为此推荐所有组件都实现类似`shouldComponentUpdate`的效果，从而在props和state不发生更新时，连VDOM都不再继续生成
