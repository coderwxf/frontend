Rollup 是一个 JavaScript 模块化打包工具，它的主要功能是将多个 JavaScript 模块打包成一个或多个大的文件，以便在浏览器或 Node.js 中使用

Rollup通过将多个小的 JavaScript 模块组合在一起，可以形成一个更大、更复杂的库或应用程序

Rollup和绝大多数构建工具一样支持通过命令行界面使用或通过JavaScript Api来进行使用

Rollup的默认入口文件是`src/main.js`, 出口文件名为`bundle.js`

```shell
npm install --global rollup
```



## 特点

### ESM

Rollup使用官方的ESM模块标准进行构建，而不是以前的 CommonJS 和 AMD 等特殊解决方案。所以默认情况下，rollup只能识别ESM模块，但是许多第三方模块是基于CommonJS模块语法编写的

为此Rollup官方提供了[@rollup/plugin-commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs)插件，使用这个插件可以让Rollup识别CommonJS模块并将其转换为ESM模块，从而确保可以在Rollup中正常使用CJS模块



Rollup可以让我们使用最新的模块化标准进行项目的编写，在构建的时候rollup会将其转换为我们所需要的具有高度兼容性的旧模块化语法，如 CommonJS 模块、AMD 模块和 IIFE 样式脚本等



如果使用rollup构建对应的库，为了确保编写的库可以被正常运行在基于CJS的构建环境中(如webpack)，推荐将ESM模块编译为UMD格式或通过如下方式配置`package.json`

```json
{
  "name": "项目名",
  "version": "版本号",
  "main": "CJS模块下，会自动引入这个字段对应的路径",
  "browser": "浏览器环境下 会自动引入这个字段对应的路径",
  "module": "ESM模块下，会自动引入这个字段对应的路径"
}
```



### 摇树 (Tree-Shaking)

因为ESM模块之间的依赖关系在编译期间就已经确定，所以Rollup可以基于`import`和`export`关键字在`静态分析代码时`就识别出模块之间的依赖关系，从而删除未被使用的代码，从而确保Rollup 只包含最基本的内容，以生成更轻、更快、更简单的库或应用程序



## 初体验

```js
// src/foo.js
export default 'hello world!';

// src/main.js --- 主入口文件
import foo from './foo.js';

// Rollup需要一个默认导出的可执行代码块(如函数、对象、类等)，以便正确识别和构建依赖树
export default function () {
	console.log(foo);
}
```

```shell
# rollup <入口文件> -f <编译输出的模块化格式> -o <出口文件>
# 1. -f 是参数 --format的简写 如果省略输出格式 默认就是ESM
# 2. -o 是参数 --output的简写 如果省略输出文件 默认就是输出到stdout(控制台)

# 注意rollup的CLI参数相对的路径都是基于当前工作路径
rollup src/main.js -f cjs -o bundle.js
```



## 配置文件

rollup默认情况下使用项目根目录下的配置文件`rollup.config.js`

Rollup 的配置文件默认情况下是使用 Node.js 进行解析的，而不是交给 Babel 等工具进行解析。因此，配置文件必须可以在没有任何编译转换的情况下可以在当前node环境下正确运行

最新版本的 Node.js 已经支持直接在 Node.js 中编写 ES Modules (ESM) 模块，因此 Rollup 的配置文件可以使用 ESM 或 CommonJS (CJS) 两种方式编写

```js
// 配置文件中的路径相对的是配置文件所在的路径
export default {
  // 入口文件
	input: 'src/main.js',
	output: {
    // 出口文件
		file: 'bundle.js',
    // 编译格式
		format: 'cjs'
	}
};
```

```shell
# -c 是 --config的简写 用来使用配置文件
# 如果不指定 -c 参数，则默认使用 rollup.config.js 文件
rollup -c

# 可以使用命令行参数覆盖配置文件中的配置，因为命令行参数的优先级更高
rollup -c -o bundle-2.js

# 如果需要使用不同的配置文件，可以通过 --config 参数来指定，
rollup --config rollup.config.dev.js
rollup --config rollup.config.prod.js
```



## 插件

随着你需要打包更复杂的代码，通常需要更灵活的配置，例如导入使用 NPM 安装的模块、使用 Babel 编译代码、处理 JSON 文件等等

此时我们就需要使用[插件](https://github.com/rollup/awesome)来在捆绑过程的关键点更改 Rollup 的行为

插件一般都是开发依赖，因为插件的本质是在构建过程中添加或更改rollup的默认行为，并不需要任何的运行时



这里以 [@rollup/plugin-json](https://github.com/rollup/plugins/tree/master/packages/json)为例，目的是允许 Rollup 从 JSON 文件中导入数据

```shell
npm install @rollup/plugin-json -D
```

```ts
// rollup.config.js

// 导入插件
import json from '@rollup/plugin-json';

export default {
	input: 'src/main.js',
	output: {
		file: 'bundle.js',
		format: 'cjs'
	},
  // 插件列表
	plugins: [json()]
};
```

```ts
// rollup.config.js
import json from '@rollup/plugin-json';
import terser from '@rollup/plugin-terser';

export default {
	input: 'src/main.js',
  // 可以同时输出多种格式的构建脚本
	output: [
		{
			file: 'bundle.js',
			format: 'cjs'
		},
		{
			file: 'bundle.min.js',
			format: 'iife',
			name: 'version',
      // 这个是对输出内容进行优化的插件
			plugins: [terser()]
		}
	],
  // 这里是构建过程中使用的插件
	plugins: [json()]
};
```



## 分包

代码分割(分包)是指有些情况下 Rollup 会自动将代码拆分成块，例如动态加载或多个入口点

还可以通过 [`output.manualChunks`](https://cn.rollupjs.org/configuration-options/#output-manualchunks) 选项显式地告诉 Rollup 需要将哪些模块拆分成单独的块

```ts
// 惰性加载
export default function () {
  // 动态导入
	import('./foo.js').then(({ default: foo }) => console.log(foo));
}
```



动态导入的包将会单独创建一个新包，为了让 Rollup 知道在哪里放置第二个块，我们不使用 `--file` 选项，而是使用 `--dir` 选项设置一个输出文件夹

```shell
rollup src/main.js -f cjs -d dist
```

这将创建一个名为 `dist` 的文件夹，其中包含两个文件，`main.js` 和 `chunk-[hash].js`, 其中 `[hash]` 是基于内容的哈希字符串

你可以通过指定 [`output.chunkFileNames`](https://cn.rollupjs.org/configuration-options/#output-chunkfilenames) 和 [`output.entryFileNames`](https://cn.rollupjs.org/configuration-options/#output-entryfilenames) 选项来提供自己的命名模式



如果我们不使用 `--dir` 选项，Rollup 将再次将块打印到 `stdout`，并添加注释以突出显示块的边界

```ts
//→ main.js:
'use strict';

function main() {
	Promise.resolve(require('./chunk-b8774ea3.js')).then(({ default: foo }) =>
		console.log(foo)
	);
}

module.exports = main;

//→ chunk-b8774ea3.js:
('use strict');

var foo = 'hello world!';

exports.default = foo;
```

通过代码分割的技术，我们可以把一些比较大或者比较耗时的模块拆分成单独的文件，只有在需要用到这些模块的时候才去加载它们，这样可以让应用程序更快地启动和运行，并提高用户体验



代码分割的另一个用途是能够指定共享一些依赖项的多个入口点

```shell
# 指定两个入口点
rollup src/main.js src/main2.js -f cjs
```

输出为

```js
//→ main.js:
'use strict';

function main() {
	Promise.resolve(require('./chunk-b8774ea3.js')).then(({ default: foo }) =>
		console.log(foo)
	);
}

module.exports = main;

//→ main2.js:
('use strict');

var foo_js = require('./chunk-b8774ea3.js');

function main2() {
	console.log(foo_js.default);
}

module.exports = main2;

//→ chunk-b8774ea3.js:
// 这是被公用的模块 所有被单独抽出以实现代码复用
('use strict');

var foo = 'hello world!';

exports.default = foo;
```











----

UMD

SystemJS