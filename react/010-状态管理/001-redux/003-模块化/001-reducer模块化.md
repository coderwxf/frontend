随着项目规模的扩大，我们需要对Redux的reducer进行模块化管理，以便于团队协作和代码维护

- **项目规模大**：项目越大，涉及的状态和操作就越多。如果所有的reducer都写在一个文件中，代码行数会非常多，不便于维护
- **团队协作**：多人协作开发时，如果所有人都修改同一个reducer文件，会导致频繁的代码冲突，增加合并代码的难度
- **代码管理**：模块化管理能够将代码按功能模块进行划分，每个模块管理自己的状态和reducer，便于维护和扩展



## 目录管理

1. 在`/src/store`下新建文件夹`reducers`
2. 在该文件夹下新建各个reducer模块 比如`rootReducer`, `personalReducer`等
3. 在`reducers`文件夹下创建一个`index.js`文件，用于合并各个模块的reducer



示例

`/src/store`的目录结构如下

```shell
.
├── consts.js
├── index.js
└── reducers
    ├── countReducer.js
    ├── index.js
    └── userInfoReducer.js
```



`const.js`

每次派发时，所有模块的 reducer 都会被查找一遍，找到匹配的行为标识后执行对应的逻辑

这就意味着如果 root 模块下有一个行为标识叫 VOTE_SUP，而 personal 模块下也有一个行为标识叫 VOTE_SUP，那派发时，两个模块的逻辑都会被执行

在大多数情况下，这不是我们期望的结果，因此，我们必须确保每个模块的行为标识是唯一的，避免冲突

```js
// 有时候推荐使用 模块名_action名的方式来定义 action事件名，从而减少冲突概率
// 在一个文件中，变量名不可能冲突，从而避免了action事件名冲突问题
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

