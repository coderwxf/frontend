## 计算机语言 vs 编程语言

计算机语言是能被计算机读取和识别的语言，通过计算机语言，人可以控制计算机完全对应功能

计算机语言有很多，如标记语言 「 HTML 」、样式语言「 CSS 」、编程语言 「 JavaScript 」



编程语言是特殊的计算机语言，相比其它计算机语言，编程语言需满足如下条件

1. 有自己的数据类型和数据结构 『 可以存储并操作数据 』
2. 有指令和流程控制 『 可以发出指令完成业务逻辑 』
3. 存在引用和复用机制 『 减少代码冗余，提升代码可读性和可维护性 』
4. 有自己的编程哲学 『 编程风格，或编程思想，如面向对象，函数式编程等 』



## 计算机语言发展

1. 机器语言
   + 0和1 组成
   + 机器可以直接识别，运行效率高
   + 不能跨平台，不同硬件有自己的指令集
   + 可读性差，不易维护
2. 汇编语言
   + 通过助记符，减少指令集
   + 依旧无法跨平台
   + 助记符繁多，难以记忆
3. 高级语言
   + 简单、易用、易于理解
   + 一个程序还可以在不同的机器上运行，具有可移植性
   + 需要编译或解释称二进制后，才能被计算机所执行
   + 不同领域的开发需要使用不同的编程语言，并没有一个统一的编程语言可以实现所有的功能

早期更接近硬件，执行效率高但复杂难以维护

现代更符合人类思维，但需要转换为二进制，存在中间环节的性能开销



## 简介

1. 简写JS
2. 基于原型
3. 头等函数
4. 多范式 「 执行 面向对象编程 和 函数式编程 」
5. 编程语言 「 高级语言 」
6. 半解释 半编译型 「JIT」



## 组成

| 组成部分   | 说明                                                  |
| ---------- | ----------------------------------------------------- |
| ECMAScript | 基本核心语法 「 无论在那种宿主环境运行都支持的语法 」 |
| DOM        | 用于操作文档(网页界面)的API                           |
| BOM        | 用于操作浏览器的API                                   |

ECMAScript是一种规范，而JavaScript是这种规范的一种实现 「 除此之外还有ActionScript和JScript等 」



## 浏览器内核

浏览器内核 = 渲染引擎 + JavaScript引擎「 如 Webkit = WebCore + JavaScriptCore 」

| 内核        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| **Gecko**   | FireFox                                                      |
| **Trident** | IE 「 已废弃 」                                              |
| **Webkit**  | Safari，webview，小程序「 小程序运行本质依旧借助于webview 」 |
| **Blink**   | chrome、Edge、Opera等 「 webkit的一个分支 」                 |



## JavaScript引擎

用于加载JavaScript，并运行JavaScript 「 翻译官，将JS转换为机器码并交给CPU来执行 」

| 引擎               | 说明                                     |
| ------------------ | ---------------------------------------- |
| **SpiderMonkey**   | FireFox<br />世界上第一个JavaScript 引擎 |
| **Chakra**         | IE「 已废弃 」                           |
| **JavaScriptCore** | Safari，webview，小程序                  |
| **V8**             | 使用Blink内核浏览器 「 Chrome 」 和 node |



## 编写位置

1. 元素上
2. script元素内 「 页面中可以存在多个script标签 」
3. script引入外部JavaScript文件



**元素上**

a元素的href属性内也可以编写JavaScript代码，但必须以`javascript:`开头

```html
<a href="https://www.google.com" onclick="alert('Hello World')">click me</a>

<a href="javascript: alert('Hello World')">click me</a>
```



## noscript

当浏览器不支持JavaScript或因测试关闭JavaScript运行时，执行`noscript`标签内部内容，否则则执行`scirpt`标签内部内容



## 注意事项

1. JavaScript严格区分大小写

2. script标签不能写成单标签，即便内部没有内容 「 一定要使用单标签时必须严格闭合，且兼容性不佳 」

3. 引入外部JS代码的script内部编写的JS代码会被忽略

   ```html
   <script src="./index.js">
     // 这里的代码会被忽略
     alert('Hello JavaScript')
   </script>
   ```

4. script的默认type属性是`text/javascipt`，也可以显示设置为其它的，如`text/babel`

5. JS代码从上到下逐行解析，所以如果JS代码要操作DOM，推荐写到body元素的最后，或加上defer属性



## defer 和 async

JavaScript会阻塞DOM的渲染 「 JS可以操作DOM，渲染完成后再修改DOM比较消耗性能，不如直接下载完JS，并在内存中操作完毕后，再进行渲染 」

但JavaScript脚本过重，就会导致页面白屏出现时间过长，此时可以为script脚本添加defer或async属性



### defer

1. JavaScript脚本不阻塞DOM Tree构建，并行下载
2. 下载完的JavaScript脚本会被存入队列
3. 在DOM树构建完成，`DOMContentLoaded`事件被回调之前，依次从队列中取出并执行

注意:

1. `defer script` 推荐放置在`head`元素中
2. `defer script` 只对外部script脚本生效，内部script脚本的defer属性会静默失效



### async

1. JavaScript脚本不阻塞DOM Tree构建，并行下载
2. 下载完成就执行对应脚本



### 总结 

- **`defer`**：适用于需要在DOM树构建完成之后执行的脚本，推荐放在 `head` 中，只对外部脚本有效。
- **`async`**：适用于独立的、不依赖DOM的脚本，下载完成后立即执行，执行顺序不确定。



### 浏览器优化

浏览器加载过程中遇到了普通script脚本「 没有加async属性 和 defer属性的script脚本 」

为了避免出现白屏，并不会等待script脚本加载完毕，而是会将已经解析完的部分DOM渲染出来，在去加载JavaScript脚本

```html
<div>这是遇到JS脚本之前的内容</div>

<script>
  // 界面已经渲染了 "这是遇到JS脚本之前的内容" 但并没有渲染 "这是遇到JS脚本之后的内容" 
  debugger
</script>

<div>这是遇到JS脚本之后的内容</div>
```



## 交互方式

| 交互方式       | 功能                                                         | 参数个数 |
| -------------- | ------------------------------------------------------------ | -------- |
| alert          | 弹窗查看<br />参数会自动转字符串                             | 一个     |
| console.log    | 在浏览器控制台查看<br />可以传入多个参数，打印时，多个参数通过空格隔开 | 多个     |
| prompt         | 在浏览器接受用户输入 <br />参数转字符串<br />用户取消时，返回null | 一个     |
| document.write | 如果文档流是新开的，替换之前所有html<br />如果可以延续之前文档流，则是追加html<br />「 控制台调试时，第一次调用时开启新文档流，之后都是沿用之前的文档流 」<br />1. `document.writeln` -- 插入内容后，插入换行「 这个换行会被浏览器解析为空格 」<br />2. `document.write`可以传入多个参数，可以简单理解为多次调用<br />`document.write('Hello', 'World') // => HelloWorld` | 多个     |



## 语句和表达式

一条完整可执行指令就是语句

表达式是运行后会有返回值的内容，一般可以放在return后



## ASI

JavaScript中使用分号表示一条语句的结束，但是可以省略

JS Engine会自动识别并在合适位置插入分号，这被称之为自动插入分号 「  automatic semicolon insert 」



但以下条件时，ASI会出现问题

1. 一行以 `[ 、{ 、（`开头
2. 多条语句位于同一行编写『 多见于压缩后的代码 』



解决方法

1. 前一条语句结尾或当前语句开头 加上 分号
2. 在当前语句开头加上 `!、+、-` 『 将当前行识别为表达式，从而和之前的语句分割开来 』



## 注释

1. 单行注释
2. 多行注释
3. 文档注释
   + HTML对文档注释提示不好，推荐在JS文件中编写，以获得更好的提示信息
