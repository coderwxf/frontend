```js
export default function combineReducers(reducers) {
  // 将多个reducer模块合并为一个新的公共的reducer
  return (state = {}, action) => {
    const keys = Object.keys(reducers)

    keys.forEach(key => {
      const reducer = reducers[key]

      // 1. reducer 执行后返回的是 新state
      // 2. 执行dispatch的时候，实际是将行为对象依次传递给每个子reducer, 让子reducer执行对应方法
      state[key] = reducer(state[key], action)
    })

    // reducer方法本身也要返回一个新的state
    return state
  }
}
```

