Vue除了支持开发SPA应用之外，其实也是支持开发SSR应用

在Vue中创建SSR应用，需要调用createSSRApp函数，而不是createApp

+ createApp:  创建应用，直接挂载到页面上
+ createSSRApp:  创建应用，是在激活的模式下挂载应用

服务端用 @vue/server-renderer 包中的 renderToString 来进行渲染



## 基本结构搭建

```js
// server.config.js --- 部分配置

const path = require('path')
const nodeExternal = require('webpack-node-externals')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

// node服务是运行在服务器端给客户端提供对应的服务的应用程序
// node服务在上线前也是需要和运行在浏览器中的项目一样进行构建和打包
// 因为node中仅有一个v8，依旧无法解析vue, jsx 或ts
module.exports = {
  // 告诉webpack, 目标运行环境是node，不是浏览器， node会自动构建出可以在node运行的代码
  // target的默认值是web，可选的值有node, webworker等
  // 设置为node时
  // 1. 使用CJS模块解析规则解析模块
  // 2. 禁用node中不存在的api
  // 3, 将node对应模块构建到对应的bundle中
  target: 'node',
  mode: 'development',
  // 这里的entry相对的是命令执行所在的路径，而非配置文件所在的路径
  entry: './src/server/index.js',
  output: {
    filename: 'bundle_server.js',
    // 这里的path必须是绝对路径
    path: path.resolve(__dirname, '../build/server')
  },
  
  resolve: {
    // resolve.extensions的默认值为 ['.js', '.json', '.wasm']
    extensions: ['.js', '.json', '.wasm', '.vue'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },

  // 在构建的时候，不需要将类似于fs，path等内置模块或第三方模块一起进行构建到输出文件中
  // 内置模块和第三方模块的时候 可以在部署的时候使用包管理工具进行安装
  // 而webpack-node-externals是Webpack的一个插件，用于忽略在打包时不需要处理的模块
  // 包括内置模块和所有在node_modules中的第三方模块
  externals: [nodeExternal()],
  plugins: [
    // 和rollup不同，webpack中的插件大多都是一个个类
    new CleanWebpackPlugin()
  ]
}
```

```lua
-- package.json

"scripts": {
  -- 如果运行的脚本文件是以 .js，.json 或 .node 结尾的，文件后缀名可以省略
  "preview": "node ./build/server/bundle_server.js",
  -- 指令名称多个单词之间 使用-或_或: 进行分割 
  -- 推荐使用: 进行分割
  -- webapck --watch 表示自动监视当前项目，如果当前项目中有任意一个文件发送了改变，就自动进行重新编译
  "build:server": "webpack --config ./config/server.config.js --watch",
  "build:client": "webpack --config ./config/client.config.js --watch"
}
```

```js
// server/index.js
const express = require('express')

// 这里命名为server，而不是app
// 避免和vue根组件app 同名
const server = express()

server.get('/', (req, res) => {
  res.end('Hello Wolrd')
})

server.listen(3000, () => console.log('server is running~'))
```



## 将APP渲染成html string

```js
// app.js

// 在服务端渲染需要使用createSSRApp 而不是createApp
import { createSSRApp } from "vue";
import App from "./App.vue";

// 导出的需要一个函数，以避免跨请求状态污染
export default () => createSSRApp(App)
```

```js
const express = require('express')
const { renderToString } = require('@vue/server-renderer')
// 在ESM中的默认导出，在CJS中会被转换为对象，并挂载到default中
const { default: createApp } = require('/app')

const server = express()

server.get('/', async (req, res) => {
  const app = createApp()
  
  // 调用renderToString方法将应用转换为字符串格式的html
  // renderToString返回的是一个promise
  // 需要等待字符串转换完成
  const htmlStr = await renderToString(app)

  res.end(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        ${htmlStr}
      </div>
    </body>
    </html>
  `)
})

server.listen(3000, () => console.log('server is running~'))
```



## hydration

```js
// client/index.js
// CSR渲染的代码和SPA的代码是完全一致的
import { createApp } from 'vue'
import App from '../../App.vue'

createApp(App).mount('#app')
```

```js
// client.config.js
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const { VueLoaderPlugin } = require('vue-loader')
const { DefinePlugin } = require('webpack')

module.exports = {
  mode: 'development',
  entry: './src/client/index.js',

  output: {
    filename: 'client_server.js',
    path: path.resolve(__dirname, '../build/client')
  },

  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader'
      },

      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },

      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ]
      },

      {
        test: /\.vue/,
        use: 'vue-loader'
      }
    ]
  },

  resolve: {
    extensions: ['.js', '.json', '.wasm', '.vue'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },

  plugins: [
    new CleanWebpackPlugin(),
    new VueLoaderPlugin(),
    // 以下两个配置只需要在CSR的时候进行配置即可
    // 因为在SSR的时候，服务端会根据用户请求动态生成HTML字符串
    // 生成的HTML字符串只包含所需的代码，未使用的代码不会被包含在HTML字符串中
    // 这就意味着在SSR的时候并不需要tree shaking
    // 而下面的选项是显示告诉打包工具是否需要将devtool和options api的支持打包到最终代码中
    // 以方便tree shaking操作，所以下述配置只需要在CSR的时候进行配置即可
    new DefinePlugin({
      // 在生产环境 关闭devtools
      __VUE_PROD_DEVTOOLS__: false,
      // 禁用options api
      __VUE_OPTIONS_API__: false
    })
  ]
}
```

```js
// server/index.js
// 对SSR代码进行如下修改，以便hydration
const express = require('express')
// 这个包不需要手动安装 安装vue，这个包就会存在于@vue目录中
const { renderToString } = require('@vue/server-renderer')
const { default: app } = require('/app')

const server = express()

// 将构建后的文件作为express的静态资源
// 以便于可以直接通过文件名来访问和加载对应的文件
server.use(express.static('build'))

server.get('/', async (req, res) => {
  const htmlStr = await renderToString(app())

  res.end(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        ${htmlStr}
      </div>

			<!--
        将构建后的脚本添加到html中
        以便于浏览器可以去加载执行对应的脚本文件
        使SSR渲染的代码具有交互功能
        这个过程被称之为水合(hydration)
      -->
      <script src="/client/client_server.js"></script>
    </body>
    </html>
  `)
})

server.listen(3000, () => console.log('server is running~'))
```



## 配置抽离

```js
// common.config.js
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const { VueLoaderPlugin } = require('vue-loader')

module.exports = {
  mode: 'development',

  module: {
    rules: [
      {
        test: /\.js$/,
        loader: 'babel-loader'
      },

      {
        test: /\.css$/,
        use: [
          'style-loader',
          'css-loader'
        ]
      },

      {
        test: /\.scss$/,
        use: [
          'style-loader',
          'css-loader',
          'sass-loader'
        ]
      },

      {
        test: /\.vue/,
        use: 'vue-loader'
      }
    ]
  },

  resolve: {
    extensions: ['.js', '.json', '.wasm', '.vue'],
    alias: {
      '@': path.resolve(__dirname, 'src')
    }
  },

  plugins: [
    new CleanWebpackPlugin(),
    new VueLoaderPlugin()
  ]
}
```

```js
// server.config.js
const path = require('path')
const nodeExternal = require('webpack-node-externals')
const { merge } = require('webpack-merge')
const commonConfig = require('./common.config')

module.exports = merge(commonConfig, {
  target: 'node',
  entry: './src/server/index.js',
  externals: [nodeExternal()],

  output: {
    filename: 'bundle_server.js',
    path: path.resolve(__dirname, '../build/server')
  }
})
```

```js
// client.config.js
const path = require('path')
const { DefinePlugin } = require('webpack')
const { merge } = require('webpack-merge')
const commonConfig = require('./common.config')

module.exports = merge(commonConfig, {
  entry: './src/client/index.js',

  output: {
    filename: 'client_server.js',
    path: path.resolve(__dirname, '../build/client')
  },

  plugins: [
    new DefinePlugin({
      __VUE_PROD_DEVTOOLS__: false,
      __VUE_OPTIONS_API__: false
    })
  ]
})
```



## 跨请求状态污染

在SPA中，整个生命周期中只有一个App对象实例 或 一个Router对象实例 或 一个Store对象实例

每个用户在 使用浏览器访问SPA应用时，应用模块都会重新初始化，这本质上是一种`单例模式`

在 SSR 环境下，App应用模块通常只在服务器启动时初始化一次，同一个应用模块会在多个服务器请求之间被复用

当某个用户对共享的单例状态进行修改，那么这个状态可能会意外地泄露给另一个在请求的用户

我们把这种情况称为: **跨请求状态污染**

为了避免跨请求状态污染，在每个请求中都使用函数的方式为整个应用创建一个全新的实例，包括router 和全局 store等实例

我们在创建App 或 路由 或 Store对象时都是函数去进行创建的，这就可以保证每次请求返回的都是一个全新的实例

但是这样也会存在对应的缺点，即每请求一次就需要创建一个全新的实例，这就会导致在服务器的内存中存在大量的实例对象，增加服务器成本开销



## ssr router

```js
// router/index.js
import { createRouter } from 'vue-router'

const routes = [
  {
    path: '/',
    component: () => import('../pages/Home.vue')
  },
  {
    path: '/about',
    component: () => import('../pages/About.vue')
  },
]

// 为了避免跨请求污染 --- 路由导出也需要使用函数
// 路由类型需要根据实际情况进行决定
// 对于SPA是createWebHashHistory或createWebhHistory
// 对于SSR是createMemoryHistory
export default function(history) {
  return createRouter({
    history,
    routes
  })
}
```

```js
// server/index.js
const express = require('express')
const { renderToString } = require('@vue/server-renderer')
const { default: createApp } = require('/app')
const { default: createRouter } = require('../router')
const { createMemoryHistory } = require('vue-router')

const server = express()

server.use(express.static('build'))

// 保证界面刷新的时候, 无论在哪个路由，都可以进入到SSR的逻辑中

// 如果路由 /
// 在http://examle.com/ 刷新 可以正常访问
// 在http://examle.com/about 刷新 无法正常访问

// 如果路由为 /*
// 在http://examle.com/ 刷新 可以正常访问
// 在http://examle.com/about 刷新 可以正常访问
server.get('/*', async (req, res) => {
  const app = createApp()

  // 在服务端没有history api 也没有hash路由方法
  // 所以在SSR时候 需要使用内存路由
  const router = createRouter(createMemoryHistory())

  // 挂载路由
  app.use(router)

  // 解析用户路由并跳转到对应的路由
  // 因为使用的是路由懒加载，所以这是一个异步的过程
  // req.url --- 用户请求过来的路径
  await router.push(req.url ?? '/')

  // 路由组件一般是异步组件，所以到等待异步组件加载和解析完毕，同时渲染完成
  await router.isReady()
  
  // 需要在路由挂载完毕后，才可以去生成对应的html字符串
  const htmlStr = await renderToString(app)

  res.end(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        ${htmlStr}
      </div>

      <script src="/client/client_server.js"></script>
    </body>
    </html>
  `)
})

server.listen(3000, () => console.log('server is running~'))
```

```js
// client/index.js
import { createApp } from 'vue'
import { createWebHistory } from 'vue-router'
import App from '../../App.vue'
import createRouter from '../router'

const router = createRouter(createWebHistory())

const app = createApp(App).use(router)

// 和SPA应用不同的是服务器渲染在大多数情况下都是异步的
// 客户端JavaScript必须等待服务器完成渲染和生成HTML响应，然后才能进行水合（hydration）操作
// 当浏览器加载应用程序的JavaScript代码时，它会创建应用程序的实例，并将其挂载到与服务器端渲染生成的HTML字符串中相同的DOM节点上
// 这个过程称为水合（hydration）它将静态的HTML转换为交互式的应用程序

// 如果客户端路由不使用isReady，客户端执行的就会是SPA逻辑
// 1. 可能会导致水合失败
// 2. 虽然SSR后 返回的依旧是完整的html，但是此时客户端已经通过SPA渲染完成了，所以没有起到加快首屏渲染的目的
router.isReady().then(() => {
  app.mount("#app");
})
```



## ssr pinia

在进行SSR的时候，服务端渲染会生成一个pinia，客户端也会生成一个pinia

在服务端渲染完成后，服务端会将状态管理 对象转换为 JSON 格式字符串，并将其嵌入到生成的 HTML 中, 挂载到window对象上

客户端从window中取出对应的json格式字符串，并进行转换以同步到本地pinia中

```js
// store/index.js
import { defineStore } from 'pinia'

export const useCountStore = defineStore('count', {
  state() {
    return {
      count: 100
    }
  },
  actions: {
    increment() {
      this.count++
    }
  }
})
```

```js
// server/index.js
const express = require('express')
const { renderToString } = require('@vue/server-renderer')
const { default: createApp } = require('/app')
const { default: createRouter } = require('../router')
const { createMemoryHistory } = require('vue-router')
const { createPinia } = require('pinia')

const server = express()

server.use(express.static('build'))

server.get('/*', async (req, res) => {
  const app = createApp()
  const router = createRouter(createMemoryHistory())
  const store = createPinia()

  app.use(router).use(store)

  await router.push(req.url ?? '/')
  await router.isReady()

  const htmlStr = await renderToString(app)

  res.end(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta http-equiv="X-UA-Compatible" content="IE=edge">
      <meta name="viewport" content="width=device-width, initial-scale=1.0">
      <title>Document</title>
    </head>
    <body>
      <div id="app">
        ${htmlStr}
      </div>

      <script src="/client/client_server.js"></script>
    </body>
    </html>
  `)
})

server.listen(3000, () => console.log('server is running~'))
```

```js
// client/index.js
import { createApp } from 'vue'
import { createWebHistory } from 'vue-router'
import App from '../../App.vue'
import createRouter from '../router'
import { createPinia } from 'pinia'

const router = createRouter(createWebHistory())
const store = createPinia()

const app = createApp(App).use(router).use(store)

router.isReady().then(() => {
  app.mount("#app");
})
```

