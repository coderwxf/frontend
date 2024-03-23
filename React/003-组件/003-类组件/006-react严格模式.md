```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App'

const root = ReactDOM.createRoot(document.getElementById('root'));

// 通过React.StrictMode开启react的严格模式
// 和use strict一样，即可以给整个应用程序开启，也可以给部分应用程序开启
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```



开启react的严格模式后

1. 使用了将来可能移除的旧API或者不推荐的做法，React 会在控制台中显示红色警告

2. 在`开发模式`下，严格模式会导致

   + 对于类组件
     + 构造函数
     + render方法
     + 生命周期函数
   + 对于函数组件
     + 函数组件本身
     + 函数组件内部调用的hook函数

   被调用两次「可以简单理解为组件被挂载后再被卸载，然后重新挂载的过程」

   目的是 发现意料之外的那些副作用

> 在引入一些老版本的第三方库时，可能会因为这些第三方库内部使用了过期API，而导致控制台出现一系列红色警告，所以在实际开发中，可能会出现关闭react严格模式的情况



## 副作用

调用组件的目的是为了渲染UI，而渲染UI过程中触发的操作，都可以被视为组件的副作用行为

1. 开启定时器
2. 操作DOM
3. 触发网络请求
4. 。。。