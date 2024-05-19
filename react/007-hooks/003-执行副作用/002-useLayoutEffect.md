`useLayoutEffect`和`useEffect`的功能和函数签名完全一致

区别是:

- `useEffect` 在渲染结果已经提交到屏幕之后执行
- `useLayoutEffect` 在 DOM 更新之后但在浏览器绘制之前执行
  - 简单来说，`useLayoutEffect`会阻塞界面渲染



`useLayoutEffect`会延迟界面渲染，降低用户体验，所以如果有可能，应该优先使用`useEffect`
