`useEffect`是React提供的用于处理副作用的Hook‘

语法规则: `useEffect([callback],[dependencies])`



在组件中，一般通过生命周期函数来添加或清理组件渲染过程中需要执行的副作用

因此`useEffect`可以在函数组件中模拟类组件中的生命周期方法



## 没有依赖

没有依赖的`useEffect`会在组件 第一次渲染完 && 每一次更新完被回调 => 等同于 `componentDidMount && componentDidUpdate`

```jsx
useEffect(() => {
  console.log('useEffect');
})
```



## 空依赖

如果`useEffect`的依赖是空数组，则只会在组件 第一次渲染完被回调 => 等同于 `componentDidMount`

```jsx
useEffect(() => {
  console.log('useEffect');
}, [])
```



## 依赖数组

第一次渲染完 以及 依赖的状态发生改变时被回调

```jsx
useEffect(() => {
  console.log('useEffect');
}, [num])
```



## 卸载函数

返回的函数将在 组件卸载前 被调用，等同于 `componentWillUnmount`

```jsx
useEffect(() => {
    return () => {
      console.log('组件卸载前被回调');
    };
  }, []);
```



## 清理函数

如果有依赖项，则useEffect返回的是该依赖项对应的清理函数

1. 当状态更新后，先执行清理函数，再执行副作用回调
2. 组件被卸载时，也会执行一遍清理函数

```jsx
useEffect(() => {
    return () => {
      console.log('num状态的清理函数');
    };
  }, [num]);
```



## 异步请求

`useEffect`的返回值只能是一个函数或者`undefined`, 不能是一个promise

所以以下写法存在错误

```jsx
useEffect(async () => {
  let result = await queryData();
  setData(result);
  console.log(result);
}, []);
```

应该修改为

```jsx
useEffect(() => {
  const next = async () => {
    let result = await queryData();
    setData(result);
    console.log(result);
  };
  next();
}, []);
```



## 闭包问题

状态更新，会产生新的执行上下文，每个执行上下文都是互相独立的

在定时器中的打印，其上层作用域是旧的闭包，并不是新的闭包环境

所以输出的结果是旧的状态值，而不是新的状态值

```jsx
let [num, setNum] = useState(0);

useEffect(() => {
  // 输出的结果是0
  const timer = setTimeout(() => console.log(num), 2000);
  return () => clearTimeout(timer);
}, [num]);
```



## 基本原理

当函数组件在渲染或更新期间遇到useEffect操作时，React会将该操作视为副作用，并将其注册到一个effect链表中

这个链表包含了回调函数（callback）和依赖项（dependencies）



在视图渲染完成后，React会按顺序执行这些副作用。在这个过程中，可能遇到如下情况

1. React会检查依赖项的值是否有更新
   + 如果依赖项有更新，对应的回调函数会被执行
   + 如果没有更新，React会继续处理下一个副作用
2. 如果依赖项是一个空数组，副作用只会在组件首次渲染完成后执行
3. 如果没有设置依赖项，副作用会在每次渲染完成后都执行
