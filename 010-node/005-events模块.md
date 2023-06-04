Node中的核心API都是基于异步事件驱动的

+ 在这个体系中，某些对象(发射器(Emitters))发出某一个事 件
+ 我们可以监听这个事件(监听器 Listeners)，并且传入的回调 函数，这个回调函数会在监听到事件时调用

所以可以将events模块看成是node中自带的事件总线

在node中发出事件和监听事件都是通过EventEmitter类来完成的，它们都属 于events对象

一些模块，例如stream和http都是继承自events，所以可以在这类模块上使用on，emit等方法

| 常见方法                           | 说明                                    |
| ---------------------------------- | --------------------------------------- |
| emitter.on(eventName, listener)    | 监听事件，也可以使用 addListener        |
| emitter.off(eventName, listener)   | 移除事件监听，也可以使用 removeListener |
| emitter.emit(eventName[, ...args]) | 发出事件，可以携带参数列表              |

```js
// 引入事件模块
const EventEmitter = require('events')

// 创建全局事件触发器
const emitter = new EventEmitter()

// 事件处理函数 - 单独抽离 便于添加事件和移除事件的时候对应的是同一个事件处理函数
const handler = (name, age) => console.log(name, age)

// 添加事件监听
emitter.on('custom', handler)

// 移除事件监听
emitter.off('custom', handler)

emitter.on('custom', handler)

// 触发事件
// 事件总线在进行参数传递的时候和vue一样
// 传递的是对应的参数列表，不是参数对象
emitter.emit('custom', 'Klaus', 23)
```



| 方法                                  | 说明                                                         |
| ------------------------------------- | ------------------------------------------------------------ |
| emitter.eventNames()                  | 返回当前 EventEmitter对象注册的事件字符串数组                |
| emitter.getMaxListeners()             | 返回当前 EventEmitter对象的最大监听器数量<br />可以通过setMaxListeners()来修改，默认是 10 |
| emitter.listenerCount(事件名称)       | 返回当前 EventEmitter对象某一个事件名称，监听器的个数        |
| emitter.listeners(事件名称)           | 返回当前 EventEmitter对象某个事件监听器上所有的监听器数组    |
|                                       |                                                              |
| emitter.once(eventName, listener)     | 事件监听一次                                                 |
| emitter.prependListener()             | 将监听事件添加到最前面                                       |
| emitter.prependOnceListener()         | 将监听事件添加到最前面，但是只监听一次                       |
| emitter.removeAllListeners(eventName) | 移除所有的监听器                                             |



