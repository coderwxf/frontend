redux 代码 一般会存放到 `src/store`中



管理员

reducer 修改公共状态的

![image-20240520230005657](https://s2.loli.net/2024/05/20/4zPodETepFGNi9v.png) 

```jsx
import { createStore } from 'redux';

// 定义初始状态
const initialState = {
  count: 0
}

// 定义Reducer
// 参数一: 需要存储的公共状态
// 参数二: 通过dispatch派发的行为对象 「 所谓行为对象就是有type属性的对象，而这个type就是行为标识 」
const reducer = (state = initialState, action) => {
  // 基于行为标识修改状态
  // 整个reducer需要返回一个全新的状态，用于替换公共容器中存储的状态值
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        count: state.count + 1
      };
    case 'DECREMENT':
      return {
        ...state,
        count: state.count - 1
      };
    default:
      return state;
  }
};

// 创建Store --- 创建一个公共容器
const store = createStore(reducer);

export default store;
```

```jsx
import { createStore } from 'redux';

const initialState = {
  count: 0
}

/*
  初始执行的时候，公共存储区域内部的状态值是undefined

  redux会在内部自动执行一次dispatch
  dispatch({
    type: [随机数]
  })
  派发的行为标识是随机数，从而确保reducer中没有任何type与其相匹配
  此时就会执行switch-case中的default逻辑

  因此就相当于用initialState来初始化公共状态区域
*/
const reducer = (state = initialState, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        count: state.count + 1
      };
    case 'DECREMENT':
      return {
        ...state,
        count: state.count - 1
      };
    default:
      return state;
  }
};

const store = createStore(reducer);

export default store;
```

```jsx
import { createStore } from 'redux';

const initialState = {
  count: 0
}

/*
  后续派发，就类似于
  dispatch({
    type: 'INCREMENT',
    。。。。
  })
  此时就会匹配到对应的行为标识
  修改状态后，替换公共状态区域中的状态
*/
const reducer = (state = initialState, action) => {
  switch (action.type) {
    // 派发的事件名，一般全大写，多个单词之间使用下划线分割 -- 例如 USER_LOGIN ？？？
    case 'INCREMENT':
      // 为了接下来的操作中，我们操作state，不会直接修改容器中的状态，需要对原本state进行浅拷贝操作 ？？?
      // reducer 需要是一个纯函数 ？？？
      return {
        ...state,
        count: state.count + 1
      };
    case 'DECREMENT':
      return {
        ...state,
        count: state.count - 1
      };
    default:
      return state;
  }
};

const store = createStore(reducer);

export default store;
```

