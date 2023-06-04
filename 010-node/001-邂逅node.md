Node.js是一个基于V8 JavaScript引擎的JavaScript运行时环境

也就是说Node.js是一个程序，这个程序内部会调用V8引擎来执行对应的JS代码，可以被用来执行JS，是JS的一种宿主环境



node中包含V8，但node中不仅仅有v8：

V8可以嵌入到任何C ++应用程序中，只要一个应用程序嵌入了V8，那么该程序就拥有了JS代码的执行能力

事实上无论是Chrome还是Node.js，都是嵌入了V8引擎来执行JavaScript代码

但是在Chrome浏览器中，还需要解析、渲染HTML、CSS等相关渲染引擎，另外还需要提供支持浏览器操作的API、浏览器自己的事件循环等

而在Node.js中我们也需要进行一些额外的操作，比如文件系统读/写、网络IO、加密、压缩解压文件等操作

所以V8只是node的一个组成部分，而不是node的全部，所以node是使用JS，C，C++ 等多种语言编写的应用程序

![image.png](https://s2.loli.net/2022/07/12/o87iN1ncGXYUWT9.png) 

`node和浏览器的差异:`

1. 在浏览器中，浏览器会先解析对应的html文件，如果在解析过程中，遇到了JS文件，就会转交给V8来进行执行

   而对于node而言 一般是直接解析对应的JS代码

2. node中的用于进行底层操作的中间层是使用C语言编写的libuv，ibuv提供了事件循环、文件系统读写、网络IO、线程池等等内容



`node架构`

![image.png](https://s2.loli.net/2022/07/12/8z6p1dyeYrnqGvF.png) 

1. 我们自己编写的JS应用程序会被V8进行解析，在解析过程中会将解析后的对应操作传递给libuv
2. libuv会帮助我们去执行和操作系统相关的底层操作，如文件读写，网络请求等
3. 因为node中绝大多数的API都是异步的，所以libuv中提供了对应的事件队列，libuv会将处理好的异步结构存放到对应的事件队列中
4. V8会在合适时间的去事件队列中读取结构并返回给我们，供我们进行调用



## 版本

Node.js是在2009年诞生的，主要分为如下两种版本
| 版本类型                               | 说明                                 |
| -------------------------------------- | ------------------------------------ |
| LTS版本（Long-term support, 长期支持） | 相对稳定一些，推荐线上环境使用该版本 |
| Current版本                            | 最新的Node版本，包含很多新特性       |



## node的版本工具

### n

```shell
# 安装
npm install n -g

# 查看版本
n --version

# 再安装和查看的时候 推荐在对应的命令之前添加sudo 以获取足够的权限

# 安装
n <版本号|latest(最新版)|lts>

# 查看 切换 使用 node版本
n
```



### nvm

```shell
# 下载 需要去github下载对应的安装包

# 查看版本
nvm version

# 查看所有安装的node列表
nvm list

# 安装
nvm install <版本号|latest(最新版)|lts>

# 切换版本(切换版本时候需要加上sudo或者在window系统中使用管理员身份打开对应的命令行工具)
nvm use <版本号|latest(最新版)|lts>
```



## REPL

REPL是Read-Eval-Print Loop的简称，翻译为“读取-求值-输出”循环

REPL是一个简单的、交互式的编程环境, 列如浏览器的控制台就是浏览器提供的REPL环境

如果我们直接输入node + 回车 就会自动进入node的REPL环境，可以供我们进行对应的测试



## 执行

```shell
# 执行脚本
node <文件名>

# 执行并监听脚本 在脚本改动时，自动重新执行脚本 --- 需要安装第三方工具 nodemon
nodemon <文件名>
```



### 输入和输出

`输入`

```shell
# node <文件名> 以空格进行分割的参数列表
node index.js name=Klaus 23
```



`输出`

```js
// process表示当前脚本进程的相关信息 比如版本、操作系统
// 我们传入的参数会被存放到其中名为argv的属性中

// argv会存放着我们传入的参数列表 -- 本质是个可以使用forEach和for-of方法的可迭代伪数组对象
// argv默认会存在两个值 分别为 node命令行工具所在的文件目录 以及 执行文件所在的文件目录
// 我们自主传入的参数会依次被插入到对应的默认参数后边

// 在node环境中 也可以使用console.log(), console.clear(), console.trace()等进行数据的输出操作
console.log(process.argv)
/*
  [
    '/usr/local/bin/node',
    '/Users/klaus/Desktop/demo/index.js',
    'name=Klaus',
    '23'
  ]
*/
```



`argv`

在C/C++程序中的main函数中，实际上可以获取到两个参数：

1. argc：argument counter的缩写，传递参数的个数
2. argv：argument vector（向量、矢量）的缩写，传入的具体参数
   + vector翻译过来是矢量的意思，在程序中表示的是一种数据结构
   + 在C++、Java中都有这种数据结构，是一种数组结构
   + 在JavaScript中也是一个数组，里面存储一些参数信息



## 全局对象

Node中给我们提供了一些全局对象，方便我们进行一些操作



### global

global是node中的全局对象，类似于浏览器中的window， process、console、setTimeout等都被挂载到global对象上

在node中的globalThis指向的就是global

```js
console.log(global === globalThis) // => true
```

```js
var foo = 'foo'

// global和window最大的不同是

// 在浏览器中 凡是使用var声明的变量都会默认被挂载到window对象上

// 但是在node环境中，使用var声明的变量并不会被默认挂载到global对象上
// 而是仅仅作为当前模块中的一个变量，并不会被挂载到全局
console.log(global.foo) // => undefined
console.log(foo) // => foo
```



### 特殊全局对象

这些全局对象实际上是模块中的变量，每个模块中都存在，而在node中每一个js文件都是一个个对应的模块，所以这些拜年了看来像是全局变量

但是这些对象并不可以在命令行交互工具中使用

node中的特殊全局对象有`__dirname、__filename、exports、module、require()`

```js
// __dirname：获取当前文件所在的路径 --- 不包括后面的文件名
console.log(__dirname) // => /Users/klaus/Desktop/demo

// __filename：获取当前文件所在的路径和文件名称 --- 包括后面的文件名称
console.log(__filename)  // => /Users/klaus/Desktop/demo/index.js
```



### 其它全局对象

| 对象名  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| process | process提供了Node进程中相关的信息<br />比如Node的运行环境、参数信息、环境变量等相关信息 |
| console | node中用于输出的对象                                         |
| 定时器  | node中的定时器对象                                           |



`node中的定时器对象`

| 定时器方法                              | 清除定时器方法 | 说明                                  |
| --------------------------------------- | -------------- | ------------------------------------- |
| setTimeout(callback, delay[, ...args])  | clearTimeout   | callback在delay毫秒后执行一次         |
| setInterval(callback, delay[, ...args]) | clearInterval  | callback每delay毫秒重复执行一次       |
| setImmediate(callback[, ...args])       | clearImmediate | callbackI / O事件后的回调的“立即”执行 |
| process.nextTick(callback[, ...args])   | 无             | 添加到下一次tick队列中                |



