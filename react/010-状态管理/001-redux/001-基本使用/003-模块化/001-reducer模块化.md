

`const.js`

```js
export const INCREMENT = 'INCREMENT'
export const DECREMENT = 'DECREMENT'

export const CHANGE_NAME = 'CHANGE_NAME'
export const CHANGE_AGE = 'CHANGE_AGE'
```



`countReducer.js`

```js
import { INCREMENT, DECREMENT } from '../consts'

const countReducer =  (state = { count: 0 }, action) => {
  switch(action.type) {
    case INCREMENT:
      return {
        ...state,
        count: state.count + 1
      }
    case DECREMENT:
      return {
        ...state,
        count: state.count - 1
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
        name: 'Alex'
      }
    case CHANGE_AGE:
      return {
        ...state,
        age: state.age + 1
      }
    default:
      return state
  }
}

export default userInfoReducer
```



`reducers/index.js`

```js
import { combineReducers } from 'redux'
import countReducer from './countReducer'
import userInfoReducer from './userInfoReducer'

// 参数对象中 key是模块名 值是reducer对象
export default combineReducers({
  count: countReducer,
  userInfo: userInfoReducer
})
```



`store/index.js`

```js
import { createStore } from 'redux'
import rootReducer from './reducers'

const store = createStore(rootReducer)

console.log(store.getState())
/*
  => { 模块名: 状态对象，模块名：状态对象，... }
    {
      count: {
        count: 0
      },
      userInfo: {
        age: 23
        name: "Klaus"
      }
    }
*/

export default store
```

