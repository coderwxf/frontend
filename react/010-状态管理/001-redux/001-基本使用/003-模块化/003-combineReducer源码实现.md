```js
// 将多个reducer模块合并为一个新的公共的reducer
export default function combineReducers(reducers) {
  return (state = {}, action) => {
    const keys = Object.keys(reducers)

    keys.forEach(key => {
      const reducer = reducers[key]

      // 1. reducer 执行后返回的是 新state
      // 2. 执行dispatch的时候，实际是将行为对象依次传递给每个子reducer, 让子reducer执行对应方法
      state[key] = reducer(state[key], action)
    })

    // 返回的是合并后的模块化state对象
    return state
  }
}
```



通过上述伪代码可以得出:



行为对象每次被派发的时候，所有模块的 reducer 都会被查找一遍，并执行匹配逻辑

这就意味着如果 root 模块下有一个行为标识叫 VOTE_SUP，而 personal 模块下也有一个行为标识叫 VOTE_SUP，

那派发时，两个模块的逻辑都会被执行

在大多数情况下，这不是我们期望的结果，因此，我们推荐每个模块的行为标识是唯一的，以避免对应冲突

