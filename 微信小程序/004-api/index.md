



在浏览器中有一个名为window的顶层对象，在小程序中也有一个功能类似的顶层对wx

该对象上挂载了宿主环境提供给我们的一系列API，通过调用这些API，可以很方便的使用微信开放给我们的能力



小程序中可以将 API 分为三类

1. 事件监听API

   这些API以on开头，用于在某些时间触发的时候执行对应的回调逻辑 

   例如 `wx.onWindowResize(callback)`

2. 同步API

   + 以`Sync`结尾
   + 通过函数返回值直接获取结果
   + 执行出错函数会抛出异常
   + 例如`wx.setStorageSync(key, value)`

3. 异步API

   + 通过传入包含回调函数的配置对象来接收结果

   + 配置对象中存在属性`success`，`fail`, `complete`，他们的值都是函数

     + 执行成功 结果会作为`success`回调的参数被传入

     + 执行失败，错误信息回座位`fail`回调的参数被传入

     + 无论是否执行成功，都会在调用`success`和`fail`之后

       统一再执行一次`complete`已进行收尾工作

   + 例如`wx.request`







































---

skyline 如何 开启 是什么 和webview的区别

