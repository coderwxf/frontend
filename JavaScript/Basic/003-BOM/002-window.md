浏览器中，window即使全局对象，也是浏览器窗口



## 作为全局对象

+ 在window对象上的所有属性都可以被访问

+ 使用var定义的变量或者隐式全局变量会被添加到window对象中 「 node不会 」

+ window默认给我们提供了全局的函数和类:setTimeout、Math、Date、Object等

+ window 有window属性指向自身，理论上可以无限递归

  ```js
  console.log(window.window.window) // window
  ```

  



## 作为浏览器窗口

因为window代表整个浏览器窗口，所以浏览器窗口的滚动条是window的，不是html和body的


| 属性                                                         | 说明                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `screenX`                                                    | 返回浏览器窗口的左上角相对于屏幕左边缘的水平距离。           |
| `screenY`                                                    | 返回浏览器窗口的左上角相对于屏幕顶部边缘的垂直距离。         |
|                                                              |                                                              |
| `scrollX`                                                    | 返回当前文档在水平方向上滚动的距离（也叫 `pageXOffset`）。   |
| `scrollY`                                                    | 返回当前文档在垂直方向上滚动的距离（也叫 `pageYOffset`）。   |
|                                                              |                                                              |
| innerWidth、innerHeight                                      | 获取浏览器窗口的视口宽度和高度（包括滚动条，但不包括工具栏等浏览器UI元素） |
| outerWidth、outerHeight                                      | 获取浏览器窗口的整个宽度和高度（包括工具栏、菜单栏、开发者工具等） |
|                                                              |                                                              |
| `documentElement.clientHeight`、`documentElement.clientWidth` | html元素的内部宽高。<br />包含内容区域和内边距，不包括边框和滚动条的尺寸。 |
| `documentElement.offsetHeight`、`documentElement.offsetWidth` | html元素的布局宽高。<br />包括内容、内边距、边框和滚动条的尺寸。 |



| 事件             | 说明                                  |
| ---------------- | ------------------------------------- |
| focus            | 浏览器获取焦点                        |
| blur             | 浏览器失去焦点                        |
| open             | 打开一个新网页                        |
| close            | 关闭使用open方法开启的新网页          |
| hashChange       | url的hash值发生了改变                 |
| load             | 页面所有内容加载完毕                  |
| DOMContentLoaded | 页面DOM加载完毕，外部资源没有加载完毕 |



>  Window类 也是继承自 EventTarget类，所以window实例也拥有addEventListener、removeEventListener、dispatchEvent等方法



## 打开新窗口

### window.open

```js
const newWindow = window.open(url, targetOrName, configString);
```

- **参数一** -- 需要打开的页面的 URL

  

- **参数二** -- 打开方式 或 新窗口名

  - **打开方式** -- 使用哪种方式打开页面，默认值是`_blank`。

  - **新窗口名** -- 如果指定了窗口名，会为新窗口起一个标识名。

    下次打开窗口的时候，如果已经存在对应标识窗口，则会刷新已有窗口。

    注意：新窗口名仅限于同源页面。

    

- **参数三** -- 对新窗口的配置字符串（不一定会生效，取决于浏览器。绝大多数浏览器倾向于新开标签页，而不是新开窗口，以减少对用户的干扰）

  - 多个值之间使用逗号隔开。
  - 如果值类型是数字，不需要单位，默认单位是像素（px）。

```js
const newWindow = window.open('/foo.html', 'foo', 'width=600,height=400,resizable=yes');

// opener表示当前窗口是由那个窗口打开的
// 如果是通过链接跳转或直接输入链接进入的窗口，opener的值是null
console.log(newWindow.opener)
```

- **返回值** -- 新打开窗口的 `window` 实例。

  

### window.close

通过 `window.open` 打开的窗口可以使用返回的 `window` 实例的 `close` 方法来关闭：

```js
newWindow.close();
```



**注意**: `window.close()` 方法可以关闭任何由脚本打开的窗口，但不能关闭用户通过点击链接或直接输入 URL 打开的窗口，除非用户授权浏览器允许该操作。尝试关闭未授权关闭的窗口可能不会生效。

```js
window.close();
```
