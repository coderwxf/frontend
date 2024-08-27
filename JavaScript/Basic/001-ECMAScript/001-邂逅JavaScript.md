### 计算机语言 vs 编程语言

**计算机语言** 是用于表达计算机程序和控制计算机系统的各种语言的总称。它包括标记语言（如HTML）、样式语言（如CSS）、编程语言（如JavaScript）等。

**编程语言** 是一种特殊的计算机语言，专门用于编写可以被计算机执行的程序。



编程语言通常具备以下特征：

1. 提供原生的**数据类型和数据结构**，用于存储和操作数据。
2. 支持**指令和流程控制**，允许开发者定义程序的逻辑。
3. 提供**引用和复用机制**，支持模块化和函数重用。
4. 通常**支持某种编程哲学或范式**，如面向对象编程、函数式编程等。



### 计算机语言的发展

1. **机器语言**
   - 由0和1组成，直接与计算机硬件通信。
   - 运行效率极高，因为计算机可以直接识别和执行。
   - 由于不同硬件有各自的指令集，机器语言无法跨平台。
   - 纯二进制代码可读性差，维护难度极大。

2. **汇编语言**
   - 使用助记符代替复杂的二进制指令，使编程稍微简化。
   - 虽然比机器语言更易读，但汇编语言依旧无法跨平台使用，因为它与硬件架构紧密相关。
   - 助记符种类繁多，学习和记忆仍然具有一定难度。

3. **高级语言**
   - 更接近人类自然语言，简单、易用、易于理解。
   - 程序具有可移植性，可以在不同硬件或操作系统上运行。
   - 需要通过编译或解释，转化为机器能够理解的二进制代码，才能执行。
   - 不同领域有不同的编程语言，没有一种统一的语言能够满足所有需求。

### 早期与现代语言的对比
- **早期语言** 更接近硬件层面，虽然执行效率高，但复杂难以维护。
- **现代语言** 更符合人类思维，编写和理解更容易，但需要经过编译或解释，因此存在性能上的一定开销。



### 简介

- JavaScript（简称 JS）是一种高级编程语言。
- 它基于原型实现。「 灵感来源于self 」
- 支持头等函数的概念。 「 灵感来源是schema 」
- 作为一种多范式语言，JavaScript 既支持面向对象编程，也支持函数式编程。
- 它是一种高级编程语言。
- JavaScript 采用半解释半编译的方式进行执行，常用 JIT（即时编译）技术来优化性能。



### 组成

| 组成部分   | 说明                                             |
| ---------- | ------------------------------------------------ |
| ECMAScript | 基本核心语法，无论在何种宿主环境运行都支持的语法 |
| DOM        | 用于操作文档（网页界面）的API                    |
| BOM        | 用于操作浏览器的API                              |

ECMAScript 「 ECMA-262 」 是一种规范，而 JavaScript 是这种规范的一种实现。

此外，还有其他实现，如 ActionScript 和 JScript 等。



### 浏览器内核

浏览器内核 = 渲染引擎 + JavaScript 引擎（例如，Webkit = WebCore + JavaScriptCore）

| 内核        | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| **Gecko**   | Firefox 使用的内核                                           |
| **Trident** | Internet Explorer 使用的内核（已废弃）                       |
| **Webkit**  | Safari、WebView、小程序使用的内核（小程序本质上依赖于 WebView 运行） |
| **Blink**   | Chrome、Edge、Opera 等浏览器使用的内核（Webkit 的一个分支）  |



### JavaScript 引擎

JavaScript 引擎用于加载和运行 JavaScript，将其转换为机器码并交给 CPU 执行。

| 引擎               | 说明                                               |
| ------------------ | -------------------------------------------------- |
| **SpiderMonkey**   | Firefox 使用的引擎，世界上第一个 JavaScript 引擎   |
| **Chakra**         | Internet Explorer 使用的引擎（已废弃）             |
| **JavaScriptCore** | Safari、WebView、小程序使用的引擎                  |
| **V8**             | Blink 内核浏览器（如 Chrome）和 Node.js 使用的引擎 |



### JavaScript 编写位置

1. **元素上**
   - 可以在 HTML 元素的事件属性中直接编写 JavaScript 代码。例如：

     ```html
     <a href="https://www.google.com" onclick="alert('Hello World')">click me</a>
     ```

   - 你也可以在 `a` 元素的 `href` 属性内编写 JavaScript 代码，但必须以 `javascript:` 开头。例如：

     ```html
     <a href="javascript:alert('Hello World')">click me</a>
     ```

2. **`<script>`元素内**
   - 页面中可以存在多个 `<script>` 标签，每个标签内可以包含 JavaScript 代码。例如：

     ```html
     <script>
         alert('This is script 1');
     </script>
     
     <script>
         alert('This is script 2');
     </script>
     ```

3. **引入外部JavaScript文件**
   - 可以通过 `<script>` 标签的 `src` 属性引入外部 JavaScript 文件。例如：

     ```html
     <script src="path/to/yourfile.js"></script>
     ```



### noscript

`<noscript>` 标签用于在浏览器不支持或禁用 JavaScript 时显示其中的内容 「 一般因测试而关闭 」

`<script>`标签则是在浏览器可以运行JavaScript时，执行其中的内容

浏览器会根据是否支持 JavaScript 来决定显示 `<noscript>` 内的内容还是执行 `<script>` 内的内容

```html
<noscript>
    <!-- 当浏览器不支持JavaScript或禁用JavaScript时显示的内容 -->
    <p>您的浏览器不支持JavaScript，或者已禁用JavaScript功能。请启用JavaScript以正常浏览此页面。</p>
</noscript>
<script>
    // 正常运行的JavaScript代码
    console.log('JavaScript is enabled and running.');
</script>
```



### 注意事项

1. JavaScript严格区分大小写。
2. script标签不能写成自闭合标签，即使内容为空，也应该使用闭合标签`</script>`
   + 虽然在HTML5中，某些浏览器可能可以处理自闭合的 `<script />`，但这并不是推荐的做法，兼容性不佳
3. 引入外部JS代码时，如果在script标签内编写了代码，内部代码会被忽略。
4. script标签的默认type属性是`text/javascript`，不需要显式声明
   + 此外可以写成`text/babel`, 让代码在被浏览器解析之前先经过babel进行转换
5. JS代码从上到下逐行解析，因此如果JS代码需要操作DOM元素，建议将其放在body标签的末尾，或者在script标签中使用`defer`属性来确保DOM已加载。



### `defer` 和 `async`

JavaScript 会阻塞 DOM 的渲染。由于 JavaScript 可以操作 DOM，因此在渲染完成后再修改 DOM 比较消耗性能。不如直接下载完 JS 后，在内存中操作完毕，再进行渲染。

但是如果 JavaScript 脚本过重，页面白屏的时间就会过长。因此，可以为 `script` 标签添加 `defer` 或 `async` 属性。



#### `defer`

1. `defer` 脚本不会阻塞 DOM 树的构建，会与 DOM 构建并行下载。
2. 下载完成后，`defer` 脚本会被存入队列。
3. 在 DOM 树构建完成，并在 `DOMContentLoaded` 事件之前，依次从队列中取出并执行脚本。

**注意：**

1. 推荐将 `defer` 脚本放置在 `head` 元素中。
2. `defer` 属性只对外部脚本生效，内部脚本的 `defer` 属性会静默失效。



#### `async`

1. `async` 脚本不会阻塞 DOM 树的构建，会与 DOM 构建并行下载。
2. 一旦下载完成，`async` 脚本会立即执行，不等待 DOM 完成构建。
3. `async` 属性只对外部脚本生效，内部脚本的 `async` 属性会静默失效。



#### 总结

- **`defer`**：适用于需要在 DOM 树构建完成后执行的脚本，推荐放在 `head` 中，只对外部脚本有效。
- **`async`**：适用于独立的、不依赖 DOM 的脚本，下载完成后立即执行，执行顺序不确定。



#### 浏览器优化

当浏览器加载过程中遇到普通的 `script` 标签（没有 `async` 或 `defer` 属性），为了避免长时间白屏，浏览器不会等待脚本加载完毕，而是会将已经解析完的部分 DOM 渲染出来，然后再去加载并执行 JavaScript 脚本。

```html
<div>这是遇到JS脚本之前的内容</div>

<script>
  // 界面已经渲染了 "这是遇到JS脚本之前的内容" 但并没有渲染 "这是遇到JS脚本之后的内容"
  debugger
</script>

<div>这是遇到JS脚本之后的内容</div>
```



### 交互方式

| 交互方式         | 功能                                                         | 参数个数 |
| ---------------- | ------------------------------------------------------------ | -------- |
| `alert`          | 弹窗显示一条消息。<br />参数会自动转为字符串，并显示在弹窗中。 | 一个     |
| `console.log`    | 在浏览器控制台输出消息。<br />可以传入多个参数，打印时，多个参数通过空格隔开。 | 多个     |
| `prompt`         | 在浏览器显示一个提示框，要求用户输入值。<br />传入的参数会转为字符串，作为提示信息显示在对话框中。<br />如果用户取消输入，返回 `null`。 | 一个     |
| `document.write` | 向文档中写入 HTML 内容。<br />如果文档流是新开的，它会替换之前所有的 HTML。<br />如果文档流已经存在，它会追加内容。<br />「在控制台调试时，第一次调用时开启新文档流，之后的调用会沿用之前的文档流」<br />1. `document.writeln` -- 插入内容后，自动添加一个换行符。<br />2. `document.write` 可以传入多个参数，效果相当于连续多次调用。<br />例如：`document.write('Hello', 'World') // => HelloWorld` | 多个     |



### 语句和表达式

**语句**（Statement）：是一条完整的可执行指令，通常用于执行某种操作。

**表达式**（Expression）：是指运行后会产生值的内容。「 可以放在return后的都是表达式 」



#### 自动插入分号（ASI）

JavaScript 中使用分号表示一条语句的结束，但分号可以省略。

JavaScript Engine会自动识别并在合适位置插入分号，这被称之为自动插入分号 

「  automatic semicolon insert 」



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