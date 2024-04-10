## 什么是node

1. 一个基于V8构建的JavaScript运行时环境
2. 可以让JavaScript在服务端运行
3. node有自己的事件循环机制，且API都是异步的 
   + 擅长处理 I/O 密集型任务，也就是需要频繁进行数据传输的任务 
   + 比如网络请求、文件系统操作等
4. JS是单线程的，因此不擅长计算密集型任务，也就是那些复杂业务逻辑



## 模块化

将程序分割成多个可重用的部分

1. 每个模块都有自己独立的作用域 -- 可以存在私有数据和方法
2. 允许你将复杂的代码逻辑分割成更小、更易于管理的部分（称为“模块”）
   + 提高代码的可维护性，还能促进代码复用



### 循环依赖

node无法处理循环依赖 「 即A引入B，B又引入A 」



## 基本使用

1. 出现是因为早期没有ESM
2. 每个文件都是一个模块 --- 文件模块
3. 导出通过`module.exports` 和 `exports`
   + `module`是当前模块实例 -- 实际通过`module.exports`导出
   + `exports`默认和`module.exports`指向一个对象，也可以导出
4. 通过`require`方法导入
   + require的参数是路径 
   + 第一次导入，以路径为key，模块执行结果为value 缓存起来
   + 第二次导入，根据key查找到对应模块执行结果，直接使用
   + 也就是说 每一个模块都是单例模式

```js
// 部分导出
exports.add = function(x, y) {
  return x + y;
}

exports.subtract = function(x, y) {
  return x - y;
}

// 没有导出，是私有函数
function print(x, y) {
	console.log(x, y)
}
```

```js
// 导入模块 -- 导入的就是 文件模块math 所导出的那个对象
const { add, subtract } = require("./math");

// 统一导出
// 此时 module.exports 指向了一个新对象，module.exports和exports不在相等
module.exports = {
  add,
  subtract,
};
```



## require.main

1. 任何一个模块的`require`方法都有一个属性`main`

2. 值是主模块实例，也就是入口模块实例
3. 可以通过这个属性判断当前模块是不是主模块

```js
if (require.main === module) {
  console.log("该模块直接运行");
} else {
  console.log("该模块被其他模块引用");
}
```



## mjs

- **ESM** 使用 `import` 和 `export` 语句来导入和导出模块。
- **CJS** 使用 `require()` 和 `module.exports` 或 `exports` 来导入和导出模块

在 Node.js 中，默认情况下，`.js` 文件被当作 CJS 模块处理

`.mjs` 扩展名，告诉 Node.js 这个文件应该作为 ESM 处理

```js
// 导出模块
export function greet(name) {
  console.log(`Hello, ${name}!`);
}
```

```js
// 导入模块
import { greet } from "./module.mjs";

greet("World"); // 输出: Hello, World!
```



## 缓存

解析后的模块会被缓存起来，也就是说CJS模块都是单例的

```js
// logger.js
class Logger {
  constructor() {
    this.logs = [];
  }

  add(log) {
    this.logs.push(log);
  }

  display() {
    console.log(this.logs);
  }
}

// 模块是单例模式
module.exports = new Logger();

// app.js
const logger1 = require("./logger");
const logger2 = require("./logger");

logger1.add("First log");
logger2.add("Second log");

logger1.display(); // 输出: ['First log', 'Second log']
```

```js
const math1 = require("./math");

// require.resolve(path) -- 将path解析为绝对路径
// require.cache -- 缓存所有模块的缓存
// 清除缓存
delete require.cache[require.resolve("./math")];

const math2 = require("./math");
```



## 文件夹模块

如果引入的是一个文件夹

1. 该文件夹下有没有`package.json`，且存在`main`字段，有则使用main字段的值作为入口

   例如，如果 `package.json` 有一个条目是 `"main": "lib/mainFile.js"`，那么 Node.js 将会加载 `lib/mainFile.js` 文件

2. 如果条件1不满足，则去加载文件夹下的`index.js`



## 第三方模块

```js
require('express')
```

1. express 是不是 node核心内置库
2. 当前目录下有没有`node_modules`，里面有没有express
3. 上一层目录有没有`node_modules`，里面有没有express
4. 。。。
5. 一直找到系统根目录

如果在任何一级的 `node_modules` 文件夹中找到了匹配的包，Node.js 就会停止搜索并加载该包

如果最终没有找到，Node.js 则会抛出一个错误



