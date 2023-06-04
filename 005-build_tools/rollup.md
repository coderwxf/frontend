Rollup是一个JavaScript的模块化打包工具，可以帮助我们编译小的代码到一个大的、复杂的代码中，比如一个库或者一个应用程序

1. Rollup也是一个模块化的打包工具
   + rollup默认情况下只能对ESM进行打包
2. Rollup的核心系统是插件系统
   + 通过rollup的插件，rollup可以打包CJS模块，css文件等
3. Rollup可以打包JS库和应用程序，但更多的还是用于构建JS库
4. rollup支持tree shaking
5. 相比webpack，rollup构建后的文件更为的精简



应用场景

1. 构建JS库，如dayjs，vue，react等
2. vite在构建阶段 底层所使用的构建工具就是rollup



## 安装

```shell
npm install rollup -g

npm install rollup
```



## 运行

### CLI

```shell
# npx rollup <入口文件> -o <出口文件>
# 在rollup构建中需要指定运行环境，即输出代码需要运行在那种环境下
# 否则rollup并不会对代码进行构建，例如如下指令，rollup会直接将输入的index.js原封不动的输出的bundle.js
$ npx rollup ./src/index.js -o ./dist/bundle.js
```

```shell
# 可以通过-f 参数 来指定输出环境

# -f cjs --- 运行在commonjs环境 如node
# -f iife --- 运行在browser环境
# -f es --- 运行在ESM环境
# -f amd --- 运行在AMD环境
# -f umd --- UMD(Universal Module Definition)模式，也就是通用模式
#        --- 可以直接运行在CJS, browser和AMD模式下，但不支持ESM模式

# 如果输出模式为iife 需要为导出内容取一个全局名称
# 这个代码的功能是为了使用这可以在浏览器中使用模块
# 这个全局名称的功能类似于 jQuery中的$和jQuery 或者说是 lodash中的_
# 如果输出模式为umd，那么就必须为导出内容取一个全局名称，否则会报错
$ npx rollup ./src/index.js -f umd  -o ./dist/bundle.js --name=util

# 新版的执行代码如下
$ rollup ./src/index.js --file ./dist/bundle.js --format umd --name "util"
```

```js
// UMD导出文件格式 --- 伪代码
(function (global, factory) {
  typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
  typeof define === 'function' && define.amd ? define(['exports'], factory) :
  (global = typeof globalThis !== 'undefined' ? globalThis : global || self, factory(global.util = {}));
})(this, (function (exports) { 'use strict';

  function foo() {
    console.log('foo');
  }

  exports.foo = foo;

}));

// 上述代码等价于
const baz = function (global, factory) {
  // 存在exports和module 所以是CJS环境
  typeof exports === 'object' && typeof module !== 'undefined' ? factory(exports) :
  // 存在define所以是AMD环境
  typeof define === 'function' && define.amd ? define(['exports'], factory) :
  // UMD支持CJS, ESM, AMD, 所以这里剩下的只有ESM

  // 如果现代浏览器支持globalThis，global就是globalThis
  // 如果老版浏览器不执行globalThis，那么就是实际运行环境中的this

  // 此时factory函数的参数exports就是global.util = {}
  // 这里的util是指定的name，也就是说在浏览器中执行的时候，对应的函数和变量会被挂载到全局对象util中
  (global = typeof globalThis !== 'undefined' ? globalThis : global || self, factory(global.util = {}));
}

// 1. this就是运行环境的globalThis
// 2. 回调函数
baz(this, (function (exports) {
  'use strict';

  function foo() {
    console.log('foo');
  }

  exports.foo = foo;

}));
```



###  配置文件

```js
// rollup的配置文件 - rollup.config.js
// 一般位于项目根目录

// 虽然rollup主要编译的代码是ESM
// 但是rollup在构建的时候 是运行在node环境中
// 所以配置文件的编写格式是CJS
module.exports = {
  // 入口文件
  input: './src/index.js',
  // 输出到
  output: {
    // 编译格式
    format: 'umd',
    // 全局对象的名称
    name: 'util',
    // 文件名
    file: './dist/bundle.js'
  }
}
```

```shell
# 运行配置文件
# 通过-c参数 告诉rollup去加载对应的配置文件
# -c 后边可以指定自定义的配置文件名称和路径
# 如果不指定 默认就是 项目根目录下的rollup.config.js
npx rollup -c <config_file_name>
```

```js
// rollup的配置文件 - rollup.config.js
module.exports = {
  input: './src/index.js',
  // output的值也可以是数组, 以便于输出运行在不同环境下的多个构建文件
  // 方便用户进行按需引入
  output: [
    {
      format: 'umd',
      name: 'util',
      // 在文件名中加上umd以区分运行在不同环境中的文件
      file: './dist/bundle.umd.js'
    },
    {
      format: 'iife',
      name: 'util',
      file: './dist/bundle.iife.js'
    },
    {
      format: 'amd',
      file: './dist/bundle.amd.js'
    },
    {
      format: 'cjs',
      file: './dist/bundle.cjs.js'
    }
  ]
}
```



## [插件](https://github.com/rollup/awesome)

### 库相关

#### @rollup/plugin-commonjs

默认情况下，rollup并不会处理CJS代码，而我们所编写的库可能会使用到一些基于CJS的库，例如lodash

所以为了我们可以正确使用CJS编写的库，就需要一个插件`@rollup/plugin-commonjs`

```js
const commonjs = require('@rollup/plugin-commonjs')

module.exports = {
  input: './src/index.js',
  output: {
    format: 'umd',
    name: 'util',
    file: './dist/bundle.js'
  },
  // 配置插件
  plugins: [
    // rollup中的每一个插件本质上都是一个个的函数
    commonjs()
  ]
}
```




#### @rollup/plugin-node-resolve

ESM中对于模块路径的解析规则和CJS中对于模块路径的解析规则是不同的

所以为了我们可以正确解析CJS编写的模块，我们还需要使用另一个插件`@rollup/plugin-node-resolve`

```js
const commonjs = require('@rollup/plugin-commonjs')
const nodeResolve = require('@rollup/plugin-node-resolve')

module.exports = {
  input: './src/index.js',
  output: {
    format: 'umd',
    name: 'util',
    file: './dist/bundle.js'
  },
  plugins: [
    commonjs(),
    nodeResolve()
  ]
}
```



此时库中一些基于CJS编写的库就可以被构建到最终输出的代码中

但是

1. 这会导致构建后的输出文件越来越大

2. 如果使用库的用户已经安装了这些基于CJS编写的库，就存在重复安装对应库的问题

   如库依赖lodash，如果使用者的项目已经安装了loadsh

   难么lodash就在库和项目中被重复安装

所以在实际构建库的时候，这些第三方依赖并不会被打包到最终输出的代码中

而是会被作为`对等依赖`，要求使用者在使用这个库之前必须安装对应的依赖

```js
// rollup.config.js
const commonjs = require('@rollup/plugin-commonjs')
const nodeResolve = require('@rollup/plugin-node-resolve')

module.exports = {
  input: './src/index.js',
  output: {
    format: 'umd',
    name: 'util',
    file: './dist/bundle.js',
    // globals 属性是用来告诉 Rollup 打包过程中哪些模块是外部依赖，以及它们在全局命名空间中的名称
    globals: {
      'lodash': '_'
    }
  },
  // 在构建的时候，不构建lodash
  external: ['lodash'],
  plugins: [
    commonjs(),
    nodeResolve()
  ]
}
```

```js
// package.json
"peerDependencies": {
  "lodash": "^4.17.21"
}
```



#### @rollup/plugin-babel

在我们编写库的时候，依旧可以需要使用ES6+语法或TypeScript语法

所以我们依旧需要将这些代码通过babel等工具进行转换

转换为浏览器可以直接识别和运行的代码

```js
// rollup.config.js
const commonjs = require('@rollup/plugin-commonjs')
const nodeResolve = require('@rollup/plugin-node-resolve')
const babel = require('@rollup/plugin-babel')

module.exports = {
  input: './src/index.js',
  output: {
    format: 'umd',
    name: 'util',
    file: './dist/bundle.js',
    globals: {
      'lodash': '_'
    }
  },
  external: ['lodash'],
  plugins: [
    commonjs(),
    nodeResolve(),
    babel({
      // bundled是默认值，但是推荐明确显示定义
      babelHelpers: 'bundled',
      exclude: /node_modules/
    })
  ]
}
```



```js
// babel.config.js
module.exports = {
  presets: [
    '@babel/preset-env'
  ]
}
```



#### @rollup/plugin-terser

构建后的代码需要进行丑化和压缩

```js
const commonjs = require('@rollup/plugin-commonjs')
const nodeResolve = require('@rollup/plugin-node-resolve')
const babel = require('@rollup/plugin-babel')
const terser = require('@rollup/plugin-terser')

module.exports = {
  input: './src/index.js',
  output: {
    format: 'umd',
    name: 'util',
    file: './dist/bundle.js',
    globals: {
      'lodash': '_'
    }
  },
  external: ['lodash'],
  plugins: [
    commonjs(),
    nodeResolve(),
    // @rollup/plugin-terser 拥有默认的terser配置，可以不手动指定
    terser(),
    babel({
      babelHelpers: 'bundled',
      exclude: /node_modules/
    })
  ]
}
```



### 业务相关

#### [@rollup/plugin-html](https://github.com/rollup/plugins/tree/master/packages/html)

```js
// 功能类似于html-webpack-plugin
const html = require('@rollup/plugin-html');

module.exports = {
  // ....
  plugins: [
    // ....
  ]
}
```



#### [rollup-plugin-delete](https://github.com/vladshcherbin/rollup-plugin-delete)

```js
// 功能类似于clean-webpack-plugin
const clean = require('rollup-plugin-delete')

module.exports = {
  // ....
  plugins: [
    // ....
    clean({
      // 指定需要删除的文件夹
      targets: 'dist/*'
    })
  ]
}
```



#### [rollup-plugin-postcss](https://github.com/egoist/rollup-plugin-postcss)

```js
module.exports = {
  // ....
  plugins: [
    // ....
  ]
}
```



#### rollup-plugin-vue


```js
module.exports = {
  // ....
  plugins: [
    // ....
  ]
}
```



#### @rollup/plugin-replace

```js
module.exports = {
  // ....
  plugins: [
    // ....
  ]
}
```




#### rollup-plugin-serve

```js
module.exports = {
  // ....
  plugins: [
    // ....
  ]
}
```




#### rollup-plugin-livereload

```js
module.exports = {
  // ....
  plugins: [
    // ....
  ]
}
```




## 区分开发环境



















babelHelpers其余各选项的含义


export.xxx 错误
