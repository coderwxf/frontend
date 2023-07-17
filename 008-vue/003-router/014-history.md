History API 是浏览器提供的一组 JavaScript 接口，用于管理浏览器的会话历史记录栈

使用这些接口，开发者可以以编程方式添加、修改或删除浏览器的历史记录

这使得开发者可以在**不刷新页面的情况下**改变浏览器的 URL 地址和页面所渲染的内容



`history`是一个全局对象，所以可以通过`window.history`或`history`来访问`history`对象

```ts
console.log(history === window.history) // => true
```



可以通过以下两个方式触发`history`记录栈的改变

1. `history.pushState/history.replaceState`
2. 浏览器自带的前进和后退按钮



## 属性和方法

### 属性 

`history`有唯一属性`length`，表示**历史记录栈**中历史记录的个数



### 方法

#### go(n)

接受一个整数作为`go`方法的参数

go方法的参数缺省值是`0`

| 参数值   | 说明            | 备注                      |
| -------- | --------------- | ------------------------- |
| `0`      | 页面刷新        | 等价于`location.reload()` |
| `正整数` | 页面前进n条记录 | 如果前进失败，静默失效    |
| `负整数` | 页面后退n条记录 | 如果后退失败，静默失效    |

```
history.go()
```



#### forward()

`forward`是个无参方法

```ts
history.forward()
// 等价于
history.go(1)
```



#### back()

```js
history.back()
// 等价于
history.go(-1)
```



#### pushState

在不进行页面刷新的前提下，向浏览器会话的历史记录栈中添加一条新记录

 ![pushState](https://s2.loli.net/2023/06/28/atr3PJqWoIUp4OV.png) 

































---

https://juejin.cn/post/7210301826938683448
https://juejin.cn/post/6948746074504986655
https://juejin.cn/post/6844903773266001933#heading-4
https://juejin.cn/post/6844903602641862663
