Babel 是一个 JavaScript 编译器，主要可以完全以下几个功能

1. 将 ECMAScript 2015+ 代码转换为旧浏览器或环境中的向后兼容版本的 JavaScript

2. 将JSX语法编写的代码转换为浏览器可以识别的原生JS代码
3. 将TS代码转换为浏览器可以直接识别的JS代码

Babel 可以帮助开发人员使用最新的 JavaScript 语言特性进行代码编写，同时又可以保证代码在旧环境中仍能正常工作

包括:语法转换、源代码转换、Polyfill实现目标环境缺少的功能等

Babel 通过将新特性转换为能在旧环境中正常工作的代码来实现这一目标

目前除了babel可以实现上述功能外，`SWC`和`esbuild`也可以同样实现和babel一样的功能



Babel使用的是`微内核`架构，即babel只提供核心的架构`@babel/core`

`@babel/core`和postcss一样，能做的功能是有限的，或者说基本没有什么功能

如果我们需要使用babel的转换功能，那么我们就需要为babel添加对应的插件或预设



## 在命令行使用

babel是一个独立的工具，可以脱离构建工具，单独使用

```shell
# @babel/core:babel的核心代码，必须安装
# @babel/cli:可以让我们在命令行使用babel
npm install @babel/cli @babel/core

 # 使用babel来处理我们的源代码
 # src:是源文件的目录
 # --out-dir:指定要输出的文件夹dist
 # npx babel <源文件或文件夹路径> --out-dir <输出文件夹>
 npx babel src --out-dir dist
```

此时，babel并没有给我们转换任何的代码，即转换前后代码是基本一致的

所以如果我们需要转换，我们可以为babel配置对应的plugins

```shell
# 一个新语法转旧语言 就需要使用一个对应的插件
# @babel/plugin-transform-arrow-functions ==> 转换箭头函数
# @babel/plugin-transform-block-scoping ===> 将const 和 let 转换为 var
npm install @babel/plugin-transform-arrow-functions  @babel/plugin-transform-block-scoping -D

# 转换 使用多个插件用逗号分隔（前后不要有空格）
npx babel src --out-dir dist --plugins=@babel/plugin-transform-block-scoping,@babel/plugin-transform-arrow-functions
```

但是如果要转换的内容过多，一个个设置和下载插件是比较麻烦的，我们可以使用预设(preset)

所谓预设，其实就是插件的集合包

```shell
# 安装基本预设
# 安装了preset后，babel会根据我们所需要适配的浏览器
# 自动选择所需要抓换的代码并对其进行转换
npm install @babel/preset-env -D

# 编译
npx babel src --out-dir dist --presets=@babel/preset-env
```



## 核心原理

从一种源代码(原生语言)转换成另一种源代码(目标语言)是编译器所需要完成的工作

所以可以将babel看成就是一个`编译器`

Babel编译器的作用就是将我们的源代码，转换成浏览器可以直接识别的另外一段源代码

对应的流程可以划分为:

1. 解析阶段(Parsing) ===> 将源代码经过解析，生成AST(抽象语法树)
2. 转换阶段(Transformation) ===> 遍历阶段，将对应的节点应用对应的插件，进行代码的转换，形成新的AST
3. 生成阶段(Code Generation) ===> 将新的AST转回我们平时常见的代码，此时新生成的代码就是已经被转换后的代码

![image.png](https://s2.loli.net/2022/12/24/AzHc439lIxeqwK2.png) 

![image.png](https://s2.loli.net/2022/12/24/HNrMCzfywvAl3UK.png) 



## 结合webpack

直接在CLI中使用对应的babel，在每次构建之前都需要手动执行对应的构建逻辑，十分不方便

为此需要将babel编译转换的命令结合到webpack中，以便于每次使用webpack进行构建的时候，可以自动对对应的JS代码进行转换操作

如果需要再webpack中对某种文件进行某种形式的转换，就需要使用loader

在这里就是`babel-loader`

```shell
npm install @babel/core babel-loader -D
npm install @babel/preset-env -D
```

```js
const path = require('path')
module.exports = {
  mode: 'development',
  devtool: false,
  entry: '/src/main.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'bundle.js',
    // 自动清除上次编译生成的文件 --- 功能和cleanWebpackPlugin一致
    clean: true
  },
  module: {
    rules: [
     {
      test: /\.m?js$/,
      use: {
        // 遇到JS文件，使用babel-loader调用babel进行处理，处理完毕后，再交给webpack进行解析
        loader: 'babel-loader',
        // 不对node_modules下的第三方库进行处理
        exclude: /node_modules/,
        // 配置项
        options: {
          // 需要使用的插件列表
          // plugins: []
          
          // 需要使用的预设列表
          // babel会自动根据browserslist查询到的所需要适配的浏览器所支持的语法
          // 自动使用预设包中对应的plugin
          // 常见的预设有@bebel/preset-env, @babel/preset-typescript, @babel/preset-react
          presets: [
            // ['@babel/preset-env', { 配置对象 }]
            /*
            	例如:
            		presets: [
                  ['@babel/preset-env', {
                    // targets属性的功能和browserslist的功能是一致的
                    // targets中的配置会覆盖browserslist中的配置
                    // 所以一般不推荐设置targets
                    // 因为browserslist可以在多个工具之间共享所需要兼容的代理列表
                    // 而targets只能用于babel一个工具
                    targets: ">5%, not dead"
                  }]
                ]
            */
            '@babel/preset-env'
          ]
        }
      }
     }
    ]
  }
}
```



## stage-x

一个新特性被正式加入ECMA规范，需要经历5个阶段

| 阶段    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| stage 0 | strawman(稻草人)，任何尚未提交作为正式提案的讨论、想法变更或者补充都被认为是第 0 阶段的"稻草人" |
| stage 1 | proposal(提议)，提案已经被正式化，并期望解决此问题，还需要观察与其他提案的相互影响 |
| stage 2 | draft(草稿)，Stage 2 的提案应提供规范初稿、草稿<br />并且开始观察设置是否合理，是否会和其它提案相互冲突 |
| stage 3 | candidate(候补)，Stage 3 提案是建议的候选提案<br />在这个阶段，规范的编辑人员和评审人员必须在最终规范上签字<br />Stage 3 的提案不会有太大的改变，在对外发布之前只是修正一些问题 |
| stage 4 | finished(完成)，进入 Stage 4 的提案将包含在 ECMAScript 的下一个修订版中 |

```js
presets: [
  // stage-0 表示 使用 babel-preset-stage-0 这个对应的预设
  // 但这是babel6的配置选项，在babel7开始已经不在被推荐，babel7建议使用preset-env
  'stage-0'
]
```



## 配置文件

如果babel配置和webpack配置写在一起，会导致webpack的配置文件越来越冗余

在实际开发中，我们可以将babel的配置单独抽离出来，形成一个独立的配置文件

| 文件                                         | 说明                                                         |
| -------------------------------------------- | ------------------------------------------------------------ |
| babel.config.json(或者.js，.cjs，.mjs)       | 推荐，babel7的配置文件<br />可以直接作用于Monorepos项目的子包 |
| .babelrc.json(或者.babelrc，.js，.cjs，.mjs) | 早期babel的配置文件<br />不再推荐使用                        |

```js
module.exports = {
  presets: [
    '@babel/preset-env'
  ]
}
```



babel的配置文件不仅仅可以导出一个对象，同样可以导出一个函数, 该函数可以接收一个参数， 这个参数就是babel在编译时候使用的[api对象](https://link.juejin.cn/?target=https%3A%2F%2Fbabeljs.io%2Fdocs%2Fen%2Fconfig-files%23apicache)

```js
module.exports = api => {
  // 开启babel对于配置文件的缓存操作
  // babel会自动在每次打包的时候判断配置文件是否发生改变，并尽可能的使用缓存来提升编译时babel的打包性能
  api.cache(true)

  const presets = [
    '@babel/preset-env'
  ]

  const plugins = [
    ['@babel/plugin-transform-runtime', {
      corejs: {
        version:  3,
        proposals: true
      }
    }]
  ]

  return {
    presets,
    plugins
  }
}
```



## polyfill

Babel 是用于转换 JavaScript 语法的工具，它不能转换 JavaScript API

如果你想在低版本浏览器或运行环境中使用某个不支持的 API，则需要使用 Polyfill

Polyfill 通常是一段 JavaScript 代码，用于模拟在旧版浏览器或运行环境中不存在的功能

例如，假设你想在旧版浏览器中使用 Fetch API 进行 HTTP 请求，但该浏览器不支持 Fetch API

这时你就可以使用 Polyfill 来模拟这个 API，从而让你的代码在旧版浏览器中正常运行



babel7.4.0之前，可以使用 @babel/polyfill的包，但是该包现在已经不推荐使用了

babel7.4.0之后，@babel/polyfill被拆分成了core-js和regenerator-runtime

我们可以通过单独引入core-js和regenerator-runtime来完成polyfill的使用

```shell
# core-js 包含了所有的新特性对应的polyfill
# regenerator-runtime包含了async/await generator所对应的polyfill
npm install core-js regenerator-runtime -D
```



### useBuiltIns

如果我们需要再项目中使用polyfill，那么我们需要在babel.config.js文件中进行配置，给preset-env配置一些属性

```js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      // corejs的默认版本是v2，但是一般使用的是v3，所以需要手动指定
      // 可以设置的版本是major和minor 例如 corejs: 3.16 (corejs: 3.16.0 => error)
      'corejs': 3, 
      // 设置以什么样的方式来使用polyfill
      // 可选值:
      // 1. false - 打包后的文件不使用polyfill来进行适配
      //          - 此时设置corejs是无效的
      //          - 因为此时相当于没有设置corejs和useBuiltIns
      'useBuiltIns': false
    }]
  ]
}
```

```js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      'corejs': 3,
      // 当使用usage，会根据源代码中出现的语言特性，自动检测所需要的polyfill
      // 这样可以确保最终包里的polyfill数量的最小化，打包的包相对会小一些
      // 1. 这里引入的polyfill仅仅值包括代码中所使用到的新特性，第三方库中的新特性并不会引入对应的polyfill
      // 2. 假设使用了String.prototype.includes这个新特性，而目标浏览器不支持该特性
      //    那么会自动引入string相关的所有polyfill
      'useBuiltIns': 'usage' // === 推荐设置
    }]
  ]
}
```

```js
module.exports = {
  presets: [
    [
      '@babel/preset-env',
      {
        useBuiltIns: 'usage',
        corejs: { 
          version: 3, // 设置使用corejs的版本号
          propsals: true // 设置corejs在进行转换的时候对提议阶段的特性进行支持
        }
      }
    ]
  ]
}
```

```js
module.exports = {
  presets: [
    ['@babel/preset-env', {
      'corejs': 3,
      // entry会根据browserslist查询到的结果，引入所有需要的 polyfill 
      // 无论引入的polyfill是否有被真正使用
      
      // 如果我们依赖的某一个库本身使用了某些polyfill的特性
      // 如果我们使用的是usage，转换之后用户浏览器可能会报错
      // 此时可以将useBuiltIns的值设置为entry
      // 因为所有所需的polyfill无论是否真正使用都已经被引入
      'useBuiltIns': 'entry'
    }]
  ]
}
```

`mian.js`

```js
// 如果useBuiltIns的值是entry
// 则必须在项目的入口文件中显示的引入如下两个库

// 此时webpack在构建的时候, 会从入口文件出发
// 根据 browserslist 目标导入所有的polyfill
// 但是此时会导致构建后的文件体积变得很大 --- 不推荐
import 'core-js/stable'
import 'regenerator-runtime/runtime'
```



| 可选值 | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| usage  | 只引入实际使用的特性所需的 polyfill<br />推荐                |
| entry  | 显式引入所有可能需要的 polyfill<br />会导致文件体积变大，且需要再入口文件中单独引入core-js和regenerator-runtime<br />不推荐 |



### @babel/plugin-transform-runtime

`core-js`和`regenerator-runtime`添加的polyfill是添加到全局的

如果开发的是一个库，且不希望污染库使用者的代码，可以使用`@babel/plugin-transform-runtime`

```shell
# 对代码进行转换，添加对应的helper函数
npm install @babel/plugin-transform-runtime -D

# 被引入的包含helper函数的库，所以是生产依赖
npm install @babel/runtime-corejs3 
```

```js
module.exports = {
  presets: [
    '@babel/preset-env',
    '@babel/preset-react',
    '@babel/preset-typescript'
  ],
  plugins: [
    ['@babel/plugin-transform-runtime', {
      corejs: 3
    }]
  ]
}
```



### [polyfill.io](https://polyfill.io/v3/)

无论是使用了`core.js和regenerator-runtime`添加对应的polyfill

还是使用`@babel/plugin-transform-runtime`添加对应的polyfill

都存在一个问题，即对应的polyfill会被添加入构建后的代码中

也就是说即使浏览器支持对应的新特性，浏览器也需要加载对应的polyfill代码

为此，社区维护了一个名为`polyfill.io`的服务，是一个CDN链接

该CDN将所需要的特性已脚本的方式引入项目中，可以根据用户浏览器的UA和连接的参数来动态判断是否需要polyfill

1. 当浏览器支持对应的polyfill代码的时候，该链接回返回空连接，因为我们并不需要任何的polyfill

2. 当浏览器不支持对应新特性，需要使用polyfill的时候，该链接就会包含所有所需要的对应polyfill

通过polyfill.io可以将polyfill不打包到自己的项目中，并且会更加精确的使用polyfill而不会浪费用户的性能

可以使用[polyfill-service](https://github.com/Financial-Times/polyfill-service)构建自己的polyfill CDN

```js
// https://polyfill.io/v3/url-builder/ 
https://polyfill.io/v3/polyfill.min.js?features=default,es2015,es2017,es2018,es2019,es2020,es2021,es2022,es2016
```



## 编译JSX

在我们编写react代码时，react使用的语法是jsx，jsx是可以直接使用babel来转换的

```shell
npm install @babel/preset-react 
```

```js
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
  mode: 'development',
  devtool: false,
  // react项目 如文件为jsx或tsx
  entry: '/src/main.jsx',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'bundle.js',
    clean: true
  },
  resolve: {
    // 默认 extensions 为 ['.js', '.json', '.wasm']
    // ps: 需要加上 点前缀
    extensions: ['.js', '.json', '.jsx']
  },
  module: {
    rules: [
     {
      // 如果是jsx 使用babel-loader进行解析
      test: /\.jsx?$/,
      exclude: /node_modules/,
      loader: 'babel-loader'
     }
    ]
  },
  plugins: [
    new HtmlWebpackPlugin({
      template: 'index.html'
    })
  ]
}
```

```js
module.exports = {
  presets: [
    '@babel/preset-env',
    '@babel/preset-react'
  ]
}
```



## 编译TS

```shell
# ts-loader在编译的时候实际使用的是tsc
# 所以在安装的时候需要将typescript一起进行安装
npm install ts-loader typescript -D

# ts文件如何进行编译取决于tsconfig.json文件
# 所以编译ts文件之前 必须先生成tsconfig.json
tsc --init
```

```js
{
  test: /\.tsx?/,
  exclude: /node_modules/,
  loader: 'ts-loader'
}
```

我们也可以使用`@babel/preset-typescript`来进行对应的转换

```js
module.exports = {
  presets: [
    '@babel/preset-env',
    '@babel/preset-react',
    // babel-loader在转换TS代码的时候，使用的是babel
    // 不是tsc，而tsconfig.json是给tsc使用的
    // 所以使用babel-loader对ts进行转换的时候 是不需要tsconfig.json的
    '@babel/preset-typescript'
  ]
}
```

| 库           | 优点                                                         | 缺点                                                         |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| ts-loader    | 会进行类型校对，出现类型错误的时候，编译会不通过             | 1. 需要额外安装一个库<br />2. 对转换后的JS代码不会进行ES6转ES5的操作，也不会添加任何的polyfill |
| babel-loader | 1.  不需要额外安装库<br />2.  对转换后的代码可以直接使用`@babel/preset-env`进行ES6转ES5的操作<br />也可以单独配置对应的polyfill | 不会进行类型校对，即使在编译的时候出现类型错误，编译依旧可以正常通过 |



所以在实际开发中，一般是`babel-loader`和`ts-loader`一起结合使用

```js
{
  test: /\.tsx?/,
  exclude: /node_modules/,
  use: [
    'babel-loader',
    'ts-loader'
  ]
}
```

我们也可以通过如下方式将`babel-loader`和`ts-loader`结合在一起进行使用

```json
"scripts": {
  // A && B - 先执行A命令，A命令执行成功后，再执行B命令
  // A & B - 同时执行A命令和B命令
  "build": "pnpm ts-check && webpack",
  "serve": "pnpm ts-check-watch && webpack serve",
  // tsc --noEmit - 编译ts文件，但是只进行类型校对，但是不进行任何的输出
  "ts-check": "tsc --noEmit",
  // --watch 检测文件内容的改变，当文件内容发送改变的时候，再次执行tsc --noEmit
  "ts-check-watch": "ts-check --noEmit --watch"
  // 上述写法等价于
  // -- --watch 是指将--watch传递给命名ts-check
  // "ts-check-watch": "npm run ts-check -- --watch"
},
```

https://juejin.cn/post/6982360866385723399#heading-4

polyfill-service

