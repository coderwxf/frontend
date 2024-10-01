`useEffect(callback, [])`   => 参数1 是回调函数「 一般被诚之为effect回调 」，参数2 是可选的依赖数组

1. 模拟生命周期，用于执行副作用操作 
2. 可以被多次添加，多个回调纳入数组并被依次回调 「 [示例代码](https://codesandbox.io/p/live/718ff372-2136-4927-87d2-9fcd6e544c2f) -- Child1 」

```jsx
 useEffect(() => {
    // 可以获取元素 => effect回调是在组件初次渲染完成或组件更新完成后被回调
    console.log(document.getElementById("foo"));
  });
```





## 基本使用

> 「 [示例代码](https://codesandbox.io/p/live/718ff372-2136-4927-87d2-9fcd6e544c2f) -- Child2 」

1. `useEffect(callback)`:

   - 当没有提供依赖数组时，`useEffect` 会在每次组件渲染后执行 => 包括首次渲染和每次更新
   - （相当于 `componentDidMount` 和 `componentDidUpdate`）。

   

2. `useEffect(callback, [])`:

   - 依赖数组为空时，`useEffect` 只在组件的首次渲染后执行一次（相当于 `componentDidMount`）。

   

3. `useEffect(callback, [x, y])`:

   - 依赖数组中包含变量时，`useEffect` 会在首次渲染后执行，
   - 并且当依赖数组中的任意一个变量发生变化时再次执行
   - （相当于 `componentDidMount` 和特定依赖的 `componentDidUpdate`）。

   

4. 清理函数（effect回调可以返回一个函数，被称之为清理函数）: 

   - 清理函数会在组件卸载时执行（相当于 `componentWillUnmount`）。
   - 如果依赖数组中的任意一项发生变化，清理函数会在执行新的 `effect` 之前执行。
     1.  可以理解为上一次渲染的组件被释放了 
     2. 会先执行清理函数，再执行新执行上下文中的effect回调 「 [示例代码](https://codesandbox.io/p/live/718ff372-2136-4927-87d2-9fcd6e544c2f) -- Child3 」

   ```jsx
   import { useEffect, useState } from 'react'
   
   export default function Child(props) {
     const [x, setX] = useState(0)
     const [y, setY] = useState(0)
   
     // 组件渲染完毕
     // 1. 点击`change X`按钮 => effect回调返回的函数会执行, X的值是之前状态值0
     // 2. 点击 `change Y`按钮 => effect回调返回的函数不会执行, 返回小函数也不会执行
     useEffect(() => {
       return () => {
         // 因为返回函数是上一次组件渲染产生的作用域中的清理函数
         // 根据闭包特性，x获取到的值是上一次闭包中的x值 => 获取的是之前的状态
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



## 基本原理

1. **执行代码并加入 `effect` 队列**：
   - 当组件渲染时，所有的 `useEffect` 钩子会被执行，并且它们的回调函数会被加入到一个 `effect` 队列中。
   - 每次组件更新时，都会产生新的 `effect` 队列。
   
2. **渲染视图完成后**：
   - **清理函数的执行**：
     - 如果不是首次渲染，React 会先执行上一个 `effect` 队列中的清理函数（如果存在）。
     - 这些清理函数是在上一次渲染时定义的，因此它们捕获的上下文是上一次渲染时的闭包环境，获取的值是旧值。
   - **执行 `effect` 回调**：
     - React 会依次检查当前 `effect` 队列中的每个回调，判断它们的依赖是否发生了变化。
     - 如果依赖发生了变化（或没有提供依赖数组），则执行对应的回调函数；如果没有变化，则跳过执行。



## 网络请求

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

`useEffect`返回值只能是清理函数，而`async`方法的返回值一定是一个新promise  => console中出现警告

解决方法

```jsx
useEffect(() => {
  const fetchData = async () => {
    queryData()
      .then(data => console.log('成功: ', data))
    	.catch(error => console.error('获取数据失败: ', error))
  };

  fetchData();
}, []);
```

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

1. **执行时机**：

   - `useLayoutEffect` 在浏览器完成布局和绘制之前同步调用
     - 这意味着如果在 `useLayoutEffect` 中进行状态更新，UI 渲染会被阻塞，直到状态更新完成。
     - 内存中的DOM已经被创建，但是UI还没有被渲染
   - `useEffect` 在浏览器完成布局和绘制之后异步调用。因此，状态更新不会阻塞初始的 UI 渲染。
     - 等UI渲染完成后，再被执行

2. **适用场景**：

   - `useLayoutEffect` 适用于需要在 DOM 更新后立即运行的代码，例如测量 DOM 元素的尺寸或位置，或者防止用户看到 UI 闪烁。

     「 [示例代码](https://codesandbox.io/p/live/718ff372-2136-4927-87d2-9fcd6e544c2f) -- Child4 」

   - `useEffect` 更适合大多数场景，因为它不会阻塞浏览器绘制，通常用于数据获取、事件订阅等不会直接影响布局的操作。

     「 [示例代码](https://codesandbox.io/p/live/718ff372-2136-4927-87d2-9fcd6e544c2f) -- Child5 」

   

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



