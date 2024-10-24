React-Redux 是 Redux 的官方库，方便在react中使用redux

1. 不需要自己创建全局上下文，用于注入`store`

   ```jsx
   // 全局注入
   import store from './store'
   import { Provider } from 'react-redux'
   
   export default function App() {
     return (
       <Provider store={store}>
         {/* ... */}
       </Provider>
     )
   }
   ```

   

2. 在组件中使用connect，不再需要通过`getState`和`subscribe`

   ```shell
   const 实际渲染的组件 = connect(mapStateToProps, mapDispatchToProps)(我们要渲染的组件)
   ```

   ` mapStateToProps`和`mapDispatchToProps` 都是函数

   + `mapStateToProps`返回需要注入到组件中的`state`
   + `mapDispatchToProps`返回需要注入到组件中的`dispatch`
   + 如果不需要可以传递`null`，进行占位



给组件注入状态

```js
import { connect } from 'react-redux'

function UserInfo({ name, age }) {
  return (
    <h1>
      { name } --- { age }
    </h1>
  )
}

// state.userInfo 本身其实就是一个对象
// 一般推荐使用了什么状态，就注入什么状态 => react-redux只会在注入的状态发生改变时才会刷新组件
export default connect(state => state.userInfo)(UserInfo)
```



给组件注入状态

**标准写法**

```jsx
import { connect } from 'react-redux'
import combineAction from './store/actions'

function Action({ counterAction, addAgeAction }) {
  return (
    <>
      <button onClick={ () => counterAction(10) }>increment</button>
      <button onClick={ addAgeAction }>add age</button>
    </>
  )
}

export default connect(null, dispatch => ({
  counterAction(step) {
    dispatch(combineAction.counter.addCount(step))
  },
  addAgeAction() {
    dispatch(combineAction.userInfo.addAge())
  }
}))(Action)
```



**简化写法**

只要传入的是`Record<string, action creator>` => react-redux内部会自动使用`bindActionCreators`转换为标准写法

```jsx
import { connect } from 'react-redux'
import combineAction from './store/actions'

let actionObj = {}

Object.values(combineAction).forEach(action => {
  actionObj = {
    ...actionObj,
    ...action
  }
})

function Action(props) {
  const { addCount, addAge } = props

  return (
    <>
      <button onClick={ () => addCount(10) }>increment</button>
      <button onClick={ addAge }>add age</button>
    </>
  )
}

// 简写方式仅执行 Record<string, action creator> => 仅支持action模块
// 不支持 { string: Record<string, action creator> } => 不支持大action
export default connect(null, actionObj)(Action)
```

```js
import { bindActionCreators } from 'redux';

bindActionCreators(action creator obj, action)
```



## 模拟实现

