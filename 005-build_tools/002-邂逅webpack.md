事实上随着前端的快速发展，目前前端的开发已经变的越来越复杂了：

+ 比如开发过程中我们需要通过模块化的方式来开发
+ 比如也会使用一些高级的特性来加快我们的开发效率或者安全性，比如通过ES6+、TypeScript开发脚本逻辑，通过sass、less等方式来编写css样式代码
+ 比如开发过程中，我们还希望实时的监听文件的变化来并且反映到浏览器上，提高开发的效率
+ 比如开发完成后我们还需要将代码进行压缩、合并以及其他相关的优化
+ 等等 。。。

而在实际开发中，我们所需要兼容的目标浏览器并不能直接识别上述的新语法和新功能

或者说部分目标浏览器支持上述部分新语法和新功能，而部分目标浏览器并不支持

所以为了所有使用目标浏览器的用户可以使用我们编写的源代码应用程序

我们就需要使用一系列的工具将我们的源代码转换为浏览器可以原生识别的Html,Css,JavaScript

而我们每次都在编写完对应的源代码后，依次手动去执行我们所需要使用的一系列功能必然是十分麻烦而且重复的

于是我们需要一个自动化的集成工具，可以集成我们所需要使用的所有的小工具，并可以根据配置自动化完成我们所需要的构建和打包功能

这个自动化集成工具，就是前端打包工具，常见的有rollup，webpack，vite等



## 简介

webpack是一个为现代的JavaScript应用程序而生的静态模块化打包工具

| 概念         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| 打包bundler  | webpack可以将帮助我们进行打包，所以它是一个打包工具          |
| 静态的static | 这样表述的原因是我们最终可以将代码打包成最终的静态资源（部署到静态服务器) |
| 模块化module | webpack默认支持各种模块化开发，ES Module、CommonJS、AMD等    |
| 现代的modern | 正是因为现代前端开发面临各种各样的问题，才催生了webpack的出现和发展<br />对于早期的非模块化项目，并不需要使用webpack这类的打包工具 |

![image.png](https://s2.loli.net/2022/07/17/jwxWOP4gUQnMmha.png) 



## 基本使用

```shell
# 安装
# 自webpack4.x版本开始 安装webpack的时候 需要安装两个包 分别为webpack 和 webpack-cli
# webpack是实际使用到的包
# webpack-cli 是如果我们需要在cli(命令行工具)中使用webpack的时候需要使用到的包

# 也就是说如果我们直接在js中使用webpack
# 例如 const webpack = require('webpack') 
# 那么我们并不需要安装webpack-cli

# 但是如果我们需要在CLI中通过命令的方式来调用webpack
# 那么我们就需要安装webpack-cli 来帮助我们将对应的CLI指令进行解析后帮助去调用webpack

# 在实际开发中，我们可能有多个项目都需要使用webpack
# 所以为了不同项目中可以使用不同版本的webpack进行构建
# 我们一般使用将webpack进行局部安装并使用
npm install webpack webpack-cli
```



### 默认打包

```shell
# 执行webpack命令后 会先去命令行执行文件夹下有没有webpack的配置文件 默认配置文件名为webpack.config.js
# 如果有则读取加载对应的配置文件，并进行构建
# 如果不存在对应的配置文件，会使用默认配置
# 默认入口文件为 /src/index.js
# 默认输出文件 /dist/main.js

# 默认的打包方式下 webpack就会对我们的输出文件进行打包压缩和丑化
# 但是并不会对ES6代码进行转换，因为webpack默认功能仅仅只是进行模块化打包
# 默认情况下， webpack并不知道如何将ES6代码转换为ES5代码或者说是否需要进行转换
# 此时需要将browserslist和babel集成到webpack中，以扩展webpack的功能
webpack
```



### 配置文件

`webpack.config.js`

```js
const path = require('path')

module.exports = {
  // 入口文件 从哪里开始打包
  entry: '/src/index.js',

  // 输出文件和文件夹配置
  output: {
    // 输出文件名
    filename: 'bundle.js',

    // 输出文件路径 --- 必须是绝对路径
    // 这里的__dirname 不可省略
    // 虽然省略__dirname 依旧可以拼接成对应的绝对路径
    // 但是此时对应的绝对路径是基于命令行执行时候所在的路径的，所以可能存在问题
    // 所以需要使用__dirname进行拼接，使其成为绝对路径
    // __dirname对应的是当前脚步所在路径，在这里就是webpack.config.js文件所在的路径
    path: path.resolve(__dirname, 'build')
  }
}
```



此时执行`webpack`命令的时候，默认就会去加载根目录下的`webpack.config.js`

并使用其中的配置进行构建

当然我们也可以修改我们对应的配置文件路径和文件名

```shell
# 在构建的时候，指定对应的配置文件
webpack --config ./config/foo.config.js
```

当然这个命令相对比较繁琐，可以将其配置到`package.json`对应的`scripts`中

```json
"scripts": {
  "build": "webpack --config ./config/foo.config.js"
}
```



## 依赖关系图

事实上webpack在处理应用程序时，它会根据命令或者配置文件找到入口文件

从入口开始，会生成一个 依赖关系图，这个依赖关系图会包含应用程序中所需的所有模块（比如.js文件、css文件、图片、字体等）

然后遍历图结构，打包一个个模块（根据文件的不同使用不同的loader来解析）



## loader

默认情况下，webpack只能对JS文件进行处理，如果我们希望webpack可以处理别的类型文件，我们需要配置

对应的loader，而loader 可以用于对模块的源代码进行转换

> 安装好loader，直接配置并进行使用即可，不需要像plugin一样单独进行引用



### css-loader

`安装`

```shell
npm i css-loader -d
```



`源码`

`样式文件`

```css
.content {
  width: 300px;
  height: 300px;
  background-color: skyblue;
}
```

`入口文件`

```js
// 仅仅只是引入对应的css文件
// 目的不是为了在入口文件中使用该css文件
// 而是为了将对应的css文件加入到对应的依赖关系图中
import './css/style.css'

const dvEl = document.createElement('div')
dvEl.classList.add('content')

document.body.appendChild(dvEl)
```



此时为了解析对应的css文件，我们需要`css-loader`来帮助我们进行使用

loader的配置方式主要有两种

1. 内联方式

+ 内联方式使用较少，因为不方便管理
+ 使用方式是在引入的样式前加上使用的loader，并且使用!分割

```js
import 'css-loader!./css/style.css'
```



2. 配置文件的方式

```js
const path = require('path')

module.exports = {
  entry: '/src/index.js',

  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  },

  // module.rules中允许我们配置多个loader
  module: {
    // rules属性对应的值是一个数组：[Rule]
    // 数组中存放的是一个个的Rule，Rule是一个对象，对象中可以设置多个属性
    rules: [
      {
        // 用于对 resource（资源）进行匹配的，通常会设置成正则表达式
        test: /\.css$/,
        // 对应的值时一个数组：[UseEntry]
        // 如果只有一个值的时候 可以使用 [UseEntry] 或 UseEntry
        use: [
          // UseEntry是一个对象，可以通过对象的属性来设置一些其他属性
          // 例如通过options属性来传入对应的参数
          { loader: 'css-loader' }
        ]
      }
    ]
  }
}
```

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        // 如果useEntry不需要传递对应参数的时候
        // 可以简写为字符串形式
        'css-loader'
      ]
    }
  ]
}
```

```js
module: {
  rules: [
    {
      test: /\.css$/,
      // 如果[Rule]中对应loader只有一个
      // 可以简写成如下形式
      // use: 'css-loader'
      // 或者
      loader: 'css-loader'
    }
  ]
}
```



### style-loader

css-loader此时已经配置成功，但是css在代码中并没有生效（页面没有效果）

因为css-loader只是负责将.css文件进行解析，并不会将解析之后的css插入到页面中

如果我们希望再完成插入style的操作，那么我们还需要另外一个loader，就是style-loader

`安装`

```shell
npm i style-loader -D
```

`配置`

```js
module: {
  rules: [
    {
      test: /\.css$/,
      // style-loader需要写在css-loader的上边
      // 因为loader的解析顺序是从后往前，从下往上进行解析
      use: [
        'style-loader',
        'css-loader'
      ]
    }
  ]
}
```





### sass-loader

如果我们需要解析sass文件的时候，我们需要借助`sass-loader`，将`sass转换为对应的css`

> less等其它css预处理器配置同理

`安装`

```shell
# sass-loader 在解析的时候 依赖于sass
# 所以在安装sass-loader的时候，需要将sass一并进行安装
npm i sass sass-loader -D
```

`配置`

```js
module: {
  rules: [
    {
      test: /\.s[ac]ss$/,
      use: [
        'style-loader',
        'css-loader',
        'sass-loader'
      ]
    }
  ]
}
```



### postcss-loader

PostCSS是一个通过JavaScript来转换样式的工具

postcss是一个可以独立运行的工具，也可以被嵌入到构建工具中一起使用

默认情况下，postcss本身并没有任何的功能

但是通过集成postcss的插件，可以帮助我们进行一些CSS的转换和适配

比如自动添加浏览器前缀、css样式的重置,自动将px转换为rem或vw等

`安装`

```shell
# postcss-loader 只是在webpack中使用postcss的一个工具
# 所以安装postcss-loader的时候，也需要将postcss一起进行安装
npm install postcss-loader postcss -D
```

`配置 -- 功能: 自动添加浏览器前缀`

`安装postcss插件`

```shell
npm i autoprefixer -D
```

`配置`

```js
module: {
  rules: [
    {
      test: /\.css$/,
      use: [
        'style-loader',
        'css-loader',
        // 在将css交给css-loader进行处理之前
        // 先让postcss-loader对css进行处理
        {
          loader: 'postcss-loader',
          // 设置postcss-loader的配置
          // 具体配置方式为loader自定义，需查看文档
          options: {
            postcssOptions: {
              plugins: [
                // 需要使用require方法引入对应的插件
                // 如果在引入插件的时候不需要传参，那么可以将其简写为字符串形式
                // require('autoprefixer')(参数列表)
                'autoprefixer'
              ]
            }
          }
        }
      ]
    }
  ]
}
```

但是将postcss的配置写在`webpack.config.js`中并不合适，因为如果在开发中使用了less，scss等预处理器的时候

我们的less和scss文件在被对应的loader处理完毕后，依旧是需要交给postcss-loader来进行转换

这就意味着对应的配置需要重复配置，为了避免重复配置，我们可以将postcss配置单独进行抽离

抽离到项目根目录下的一个叫做`postcss.config.js`的文件中

`postcss.config.js`

```js
module.exports = {
  plugins: [
    'autoprefixer'
  ]
}
```



事实上，在配置postcss-loader时，我们配置插件并不需要使用autoprefixer

在实际开发中，我们可能会使用到很多的postcss插件，为此postcss为我们提供了`插件集合`，即对应的`预设文件`

所谓预设就是一系列的插件和对应插件配置的集合包

在打包的时候，预设包会自动进行按需引入，即自动识别并使用我们所需要的预设包中的插件进行打包

我们配置预设包后，就相当于集成了一系列的插件

`postcss`也有对应的预设包`postcss-preset-env`

`postcss-preset-env`可以帮助我们将一些现代的CSS特性，转成大多数浏览器认识的CSS，并且会根据目标浏览器或者运行时环境添加所需的polyfill，也包括会自动帮助我们添加autoprefixer（即`postcss-preset-env`内部集成了`autoprefixer`）

`安装`

```shell
npm install postcss-preset-env -D
```

`配置`

```js
module.exports = {
  plugins: [
    'postcss-preset-env'
  ]
}
```



## 打包资源文件

在webpack5之前，加载这些资源我们需要使用一些loader，比如raw-loader 、url-loader、file-loade

在webpack5开始，我们可以直接使用资源模块类型（asset module type），来替代上面的这些loader

资源模块类型(asset module type)，通过添加 4 种新的模块类型，来替换所有这些 loader：

| 模块类型       | 说明                                                 |
| -------------- | ---------------------------------------------------- |
| asset/resource | 形成一个单独的文件URL                                |
| asset/inline   | 将文件转换为对应的data URL                           |
| asset/source   | 导出资源的源代码                                     |
| asset          | 在导出一个 data URI 和发送一个单独的文件之间自动选择 |

`配置`

```js
module: {
  rules: [
    {
      test: /\.(gif|png|jpe?g|svg)$/,
      type: 'asset'
    }
  ]
}
```



`asset/resource`会将文件拷贝一份放置到打包文件夹中，并使用hash算法生成唯一的文件名，虽然在打包后的文件中进行引入，例如`http://localhost/demo.png`

`asset/inline`会将文件以base64的形式进行编码后，再插入到打包后的文件中



`asset/resource`可以使打包后的文件体积变小，但是需要多发网络请求来获取资源

`asset/inline`会将对应资源文件打包入文件中，可以减少对应的网络请求，

但是因为资源全部被插入到打包后的文件中，便会导致打包后的文件过大

所以为了在两者之间寻求平衡，我们一般将对于的asset module type设置为`asset`

即小的图片需要转换为base64后进行页面嵌入，但是大的图片直接使用图片即可

```js
module: {
  rules: [
    {
      test: /\.(gif|png|jpe?g|svg)$/,
      type: 'asset',
      // 设置资源解析条件
      parser: {
        // 当资源体积 大于等于 5kb的时候，以单独链接的形式进行引入
        // 当资源体积 小于 5kb的时候 对图片资源进行base64编码，并嵌入到对应的打包后文件中
        dataUrlCondition: {
          maxSize: 5 * 1024 // 单位为byte
        }
      },
      // 设置打包后对应资源的输出位置和命名方式
      generator: {
        // [name], [ext] [hash] 等是帮助我们获取原始文件信息的占位符
        // [name] - 文件名
        // [hash] - 文件的内容，使用MD4的散列函数处理，生成的一个128位的hash值(32个十六进制)
        //        - [hash:6] 表示取hash值的前6个字符
        // [ext] - 文件扩展名 - 注意这里的ext获取的文件后缀名是带点的
        filename: 'img/[name]-[hash:6][ext]'
      }
    }
  ]
}
```

```js
output: {
  filename: 'bundle.js',
  path: path.resolve(__dirname, 'build'),
  // 如果我们需要指定输出的资源路径和名称，我们也可以在output中进行配置
  // 但是output中的配置是全局配置，而实际开发中我们可能会使用到多种不同的资源
  // 如文件资源，图片资源，字体资源等，我们需要将他们放置到不同的文件夹下
  // 所以不推荐在output中对输出文件的名称和位置进行统一配置
  // 而是根据文件类型，单独对不同的文件类型进行单独配置
  assetModuleFilename: '[name]-[hash:6][ext]'
}
```



## babel

Babel是一个工具链，主要用于旧浏览器或者环境中将ECMAScript 2015+代码转换为向后兼容版本的JavaScript，包括:语法转换、源代码转换等

目前babel主要有三个功能:

1. ES6+ -> es5
2. Typescript -> Javascript
3. JSX -> React.createElemnet



babel本身可以作为一个独立的工具(和postcss一样)，不和webpack等构建工具配置来单独使用

如果我们希望在命令行尝试使用babel，需要安装如下库

| 库          | 说明                        |
| ----------- | --------------------------- |
| @babel/core | babel的核心代码，必须安装   |
| @babel/cli  | 可以让我们在命令行使用babel |



在实际开发中，我们通常会在构建工具中通过配置babel来对其进行使用的，比如在webpack中，

那么我们就需要去安装构建工具中对应的babel插件，在webpack中，对应的插件即为`babel-loader`

```shell
npm install babel-loader @babel/core =D
```



`基本配置`

```js
module: {
  rules: [
    {
      test: /\.js$/,
      use: [
        'babel-loader'
      ]
    }
  ]
}
```



此时执行构建指令后，会发现ES6+的代码并没有被转换为ES5

这是因为babel和postcss一样，如果要实现对应的功能需要再安装babel的插件和预设包

| 插件                                    | 功能                   |
| --------------------------------------- | ---------------------- |
| @babel/plugin-transform-arrow-functions | 箭头函数转换为普通函数 |
| @babel/plugin-transform-block-scoping   | let/const -> var       |



如果我们一个个去安装使用插件，那么需要手动来管理大量的babel插件，我们可以直接给webpack提供一个preset，webpack会 根据我们的预设来按需加载对应的插件列表，并且将其传递给babel

| 预设                     | 说明        |
| ------------------------ | ----------- |
| @babel/preset-env        | ES6+ -> ES5 |
| @babel/preset-react      | Jsx -> JS   |
| @babel/preset-typescript | TS -> Js    |

```js
module: {
  rules: [
    {
      test: /\.js$/,
      use: [
        {
          loader:  'babel-loader',
          options: {
            presets: [
              '@babel/preset-env'
            ]
          }
        }
      ]
    }
  ]
}
```



但是将配置写在loader配置中，在后期维护成本是很高的

babel可以和postcss一样，将对应的配置抽离到根目录下的一个名为`babel.config.js`的文件中

`babel.config.js`

```js
module.exports = {
  // plugins: [
  //   '@babel/plugin-transform-arrow-functions',
  //   '@babel/plugin-transform-block-scoping'
  // ],

  presets: [
    '@babel/preset-env'
  ]
}
```

`webpack.config.js`

```js
module: {
  rules: [
    {
      test: /\.js$/,
      loader: 'babel-loader'
    }
  ]
}
```



## sfc

在开发中我们会编写Vue相关的代码，webpack可以对Vue代码进行解析

```shell
# vue-loader 用于对vue进行构建
npm install vue-loader -D

# vue-loader的构建和vue的运行依赖于vue
# 所以需要安装vue，并且需要为生产依赖
# 事实上 对于vue的SFC文件的解析 使用的就是@vue/compiler-sfc
npm install vue
```

`webpack.config.js`

```js
const path = require('path')

// 打包vue的SFC文件比较特殊，即需要使用loader，也需要使用plugin
const { VueLoaderPlugin  } = require('vue-loader/dist/index')

module.exports = {
  entry: '/src/index.js',

  output: {
    filename: 'bundle.js',
    path: path.resolve(__dirname, 'build')
  },

  module: {
    rules: [
      {
        // 识别vue文件，并对其进行解析
        test: /\.vue$/,
        loader: 'vue-loader'
      }
    ]
  },

  plugins: [
    // VueLoaderPlugin主要用于对vue SFC文件中的样式进行抽离和处理
    new VueLoaderPlugin()
  ]
}
```



## resolve

resolve用于设置模块如何被解析:

+ 在开发中我们会有各种各样的模块依赖，这些模块可能来自于自己编写的代码，也可能来自第三方库
+ resolve可以帮助webpack从每个 require/import 语句中，找到需要引入到合适的模块代码
+ webpack 使用 enhanced-resolve 来解析文件路径
+ webpack能解析三种文件路径
  1. 绝对路径
     + 由于已经获得文件的绝对路径，因此不需要再做进一步解析
  2. 相对路径
     + 在这种情况下，使用 import 或 require 的资源文件所处的目录，被认为是上下文目录
     + 在 import/require 中给定的相对路径，会拼接此上下文路径，来生成模块的绝对路径
  3. 模块路径
     + 在 resolve.modules中指定的所有目录检索模块
     + resolve.modules的默认值是['node_modules']，所以默认会从node_modules中查找文件
+ webpack对于文件和文件夹的解析规则如下:
  + 如果是一个文件
    + 如果文件具有扩展名，则直接打包文件
    + 否则，将使用 resolve.extensions选项作为文件扩展名解析
  + 如果是一个文件夹
    + 会在文件夹中根据 resolve.mainFiles配置选项中指定的文件顺序查找
    + resolve.mainFiles的默认值是 ['index']
    + 再根据 resolve.extensions来解析扩展名



### extensions 和 alias

#### extensions

extensions是解析到文件时自动添加扩展名

默认值是 ['.wasm', '.js', '.json']

所以如果我们代码中想要添加加载 .vue 或者 jsx 或者 ts 等文件时，我们必须自己写上扩展名



#### alias

当我们项目的目录结构比较深的时候，或者一个文件的路径可能需要 `../../../`这种路径片段时候，我们可以给某些常⻅的路径起一个别名

```js
resolve: {
  // 设置默认后缀，如果自己手动设置，会覆盖默认设置，所以对应的默认值也需要被添加
  // .wasm 后缀文件是 web assembly 文件的后缀 较少使用 一般会被移除
  extensions: ['.js', '.json', '.ts', '.json', '.vue', '.jsx', '.tsx'],
    alias: {
      // 别名的值可以是绝对路径，也可以是相对路径
      // 一般推荐是使用绝对路径
      '@': path.resolve(__dirname, '/src')
    }
}
```



## 浏览器兼容性

在开发中，我们需要去处理兼容性问题，针对不同的浏览器支持的特性: 比如css特性、js语法，进行不同的适配

那么我们就需要一个工具，单独来查询浏览器适配问题，并将适配信息在多个工具之间进行共享这些信息

这个工具就是`browserslist`, 他会去使用其依赖的包`caniuse-lite`去[caniuse](https://link.juejin.cn/?target=https%3A%2F%2Fcaniuse.com%2Fusage-table)网站上查询浏览器的兼容性情况

随后在所有需要进行浏览器兼容性适配的工具之间共享浏览器兼容性数据

`browserslist的默认配置如下`，详细规则参考[browserslist配置规则](https://github.com/browserslist/browserslist)

当postcss或babel在进行代码适配的时候，就会通过browserslist去查找对应所需要适配的浏览器列表

1. postcss和babel在安装的时候，默认会将browserslist作为其对应依赖进行安装
2. 如果没有配置browserslist规则，那么就会使用browserslist的默认规则去进行适配

```js
> 0.5%
last 2 versions
Firefox ESR
not dead
```

| 常见配置项         | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| > 0.5%             | 市场份额大于0.5%的浏览器<br />还可以`>=，>, <和<=`一起使用   |
| last 2 versions    | 最后两个版本                                                 |
| Firefox ESR        | Firefox专门为企业或组织创建的浏览器                          |
| not dead           | 某一个浏览器如果官方24个月没有发布任何新的版本，那么就认为这个浏览器已经不维护了，也就是说是dead<br />not dead 就意味着24个月内该浏览器有进行升级和维护 |
| node 10和node 10.4 | 需要兼容node10.x，node10.4.x 版本的浏览器                    |
| ie > 8             | 需要兼容ie8以上的浏览器                                      |
| not ie <= 8        | 需要兼容ie8以上的浏览器                                      |

| 关系符             | 说明                 |
| ------------------ | -------------------- |
| or 或 逗号 或 换行 | 并集，表示或的关系   |
| and                | 交集， 表示并的关系  |
| not                | 取反，表示非集的关系 |



#### 在命令行使用

```shell
# 没有查询条件，查看有没有配置文件，有就使用配置文件，没有就使用内置的默认配置
$ npx browserslist

# 查询的时候，使用查询条件来进行查询
# 多个条件之间使用逗号进行隔开
$ npx browserslist ">1%, last 2 version, not dead"
```



#### 结合webpack进行使用

`配置方式1 --- package.json`

```json
"browserslist": [
  "> 1%",
  "last 2 versions",
  "not dead"
]
```

```json
"browserslist": {
  "development": [
    "> 1%",
    "last 2 versions",
    "not dead"
  ],

  "production": [
    "> 0.5%",
    "not dead"
  ]
}
```



`置方式2 --- 在项目根目录下新建.browserslistrc`

```shell
> 2%
last 2 versions
not dead
```

