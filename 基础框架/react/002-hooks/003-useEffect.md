`useEffect`的作用`模拟生命周期` => 用于执行对应副作用操作

`useEffect(callback, [])`

1. `callback` => 更新回调 「 effect回调 」
2. `[]` => 依赖数组

`useEffect`可以被多次调用，对应effect回调会被加入数组，并在合适的时候被依次回调



## 基础语法

1. `useEffect(callback)` => `componentDidMount` 和 `componentDidUpdate`
1. `useEffect(callback, [])` => `componentDidMount`
1. `useEffect(callback, [x, y])` => `componentDidMount` 和特定依赖的 `componentDidUpdate`



### 清理函数

effect回调可以返回一个函数，被称之为清理函数

清理函数会在effect回调被执行之前和`componentWillUnmount`时被回调

```jsx
import { useEffect, useState } from 'react'

export default function Child(props) {
  const [x, setX] = useState(0)
  const [y, setY] = useState(0)

  useEffect(() => {
    return () => {
      console.log(x) // => 0
    }
  }, [x])

  return (
    <>
      <h2>{x} - {y}</h2>
      <button onClick={() => setX(x + 1)}>change X</button>
      <button onClick={() => setY(y + 1)}>change Y</button>
    </>
  )
}
```

点击`change X` => 状态`x`发生了改变，`useEffect`回调被执行 => `useEffect`回调被执行之前先执行清理函数

此时因为清理函数属于上一个函数执行上下文，也就是上一个闭包，因为`x`的值就是`0`

点击`change Y` => 状态`y`发生了改变 => useEffect回调不被执行 => 清理函数不被触发



## 实现原理

每次组件渲染时，会生成一个新的`effect队列`

执行函数逻辑，将符合条件的effect回调加入effect队列 

在组件渲染完成后，依次回调effect队列中的事件

+ 如果effect回调存在清理函数，则会在执行effect回调执行前先执行清理函数
+ 清理函数是之前执行上下文中产生的，所以其获取的状态值都是上一个执行上下文中对应的状态值



## 异步处理

```jsx
useEffect(async () => {
 try {
    let data = await queryData();
    console.log('成功: ', data);
  } catch (error) {
    console.error('获取数据失败: ', error);
  }
}, []);
```

这是错误的，因为useEffect的返回值只能是清理函数 「 是同步函数 」

而`async函数`的返回值是`promise`，不是同步函数



解决方法

```jsx
useEffect(() => {
  const fetchData = async () => {
    try {
      let data = await queryData();
      console.log('成功: ', data);
    } catch (error) {
      console.error('获取数据失败: ', error);
    }
  };

  fetchData();
}, []);
```



## useLayoutEffect

**执行时机**：

- `useLayoutEffect` 在 DOM更新后，浏览器进行实际绘制前执行
- `useEffect` 在DOM更新后 且 浏览器实际绘制完成后执行

**使用场景**：

- `useEffect` 适合非阻塞渲染操作

  - 例如网络请求或添加事件回调等

- `useLayoutEffect` 适合于同步操作

  - 例如实际渲染前对DOM进行操作「避免布局抖动（layout thrashing）」或读取DOM信息「`getComputedStyle`」

  

```jsx
import { useEffect, useLayoutEffect, useState } from "react";

export default function Child() {
  const [count, setCount] = useState(0);

  /*
    useLayoutEffect的输出一定早于useEffect的输出
    且他们在执行的时候DOM都已经生成完毕，可以被获取了
  */
  useLayoutEffect(() => {
    console.log("useLayoutEffect");
    console.log(document.getElementById("count"));
  }, [count]);

  useEffect(() => {
    console.log("useEffect");
    console.log(document.getElementById("count"));
  }, [count]);

  return (
    <div>
      <h2 id="count">{count}</h2>
      <button onClick={() => setCount(count + 1)}>click me</button>
    </div>
  );
}
```

