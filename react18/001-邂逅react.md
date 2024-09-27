```jsx
import React from 'react'; // react核心库 JSX -> VDOM
import ReactDOM from 'react-dom/client'; // react渲染库 VDOM -> DOM
import App from './App' //  根组件

// 定义根容器 => 挂载点是 div#root
const root = ReactDOM.createRoot(document.getElementById('root'));

// 渲染应用
root.render(<App />);
```



1. vue的挂载点一般为`div#app`, react的挂载点一般为`div#root`

   + vue 和 react 都不推荐 使用 `html`元素 或 `body`元素作为挂载点

2. 无论是vue还是react，都会用渲染后的应用替 换挂载点中原本存在的内容 

   「 挂载点中的内容是应用，挂载点本身不属于应用的一部分  」

3. React18开始渲染是通过新建根元素，再渲染根组件

   ```js
   const root = ReactDOM.createRoot(document.getElementById('root'));
   
   root.render(<App />);
   ```

   React18之前是，直接通过render方法进行渲染

   ```js
   React.render(<App />, document.getElementById('root'))
   ```

   
   
   当存在多个根节点需要渲染时，这多个根节点的渲染方式如下:
   
   + `createRoot + render` => 采用React的并发模式
   
   + `render` => 同步渲染模式



## react的渲染库

| 库                 | 说明                                        |
| ------------------ | ------------------------------------------- |
| `react-dom/client` | 构建WebApp => CSR「 VDOM -> DOM 」          |
| `react-dom/server` | 构建WebApp => SSR 「 VDOM -> Html String 」 |
| `react-native`     | 构建NativeApp => 「 VDOM -> APP原生控件 」  |



## react版本

React16 是个版本大更新 => 引入了fiber

React17 和 React16 基本用法保持一致  只是底层进行了性能优化

React18 是个版本大更新 => 引入了并发特性 和 hook函数
