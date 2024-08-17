window在浏览器中有两种用法

+ GO
+ 浏览器窗口



如果作为GO

+  作用域链的顶层
+ 一些全局属性和方法会被挂载到GO上
+ var声明变量和隐式全局变量会被挂载到window 「 node不会 」
+ 存在指向自身的属性，理论上可以无限递归

```js
console.log(window.window.window) // window
```



 **globalThis**

不同JavaScript宿主环境 「 运行环境 」 的GO名不一样

+ 浏览器 -- window
+ node -- global
+ web worker -- self

所以为了统一，GO统一叫做globalThis，其会根据实际宿主环境去指向实际的GO对象