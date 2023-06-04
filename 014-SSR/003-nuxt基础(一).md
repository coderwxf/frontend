[Nuxt3](https://nuxt.com/) 是基于 Vue3 + Vue Router + Vite 等技术栈，全程 Vue3+Vite 开发体验(Fast)

## 特点

**自动导包**

+ Nuxt 会自动导入辅助函数(helper function)、组合 API和 Vue API ，无需手动导入
+ 基于规范的目录结构，Nuxt 还可以对自己的组件、 插件使用自动导入



**约定式路由**

约定式路由 即 目录结构即路由

Nuxt 路由基于vue-router，在 pages/ 目录中创建的每个页面，都会根据目录结构和文件名来自动生成路由



**渲染模式**

Nuxt 支持多种渲染模式(SSR、CSR、SSG等)

也支持混合渲染，即同时使用SSR,  CSR 和 SSG 中的一种或多种开发模式进行开发



**服务器引擎**

+ 在开发环境和生产环境中，nuxt3使用的都是nitro+vite进行开发和构建
+ 在nuxt3中使用的是vite + [nitro](https://nitro.unjs.io/), 而在nuxt2中使用的是express提供服务端的渲染服务
  + nitro也是一个web服务框架，用于替换express
  + nitro构建后的代码具有很好的移植性，可以部署在包括 Node、Deno、Serverless、Workers等平台上
    + 可以安装对应的插件并使用其中的默认配置进行部署
    + 也可以使用自定义配置覆盖默认配置
    + nuxt.js 需要手动进行对应配置的编写，相比nitro需要手动编写的配置要多得多
    + 这类似于webpack和vite
+ 在生产环境中，使用 Nitro 将应用程序和服务器构建到一个通用`.output`目录中
  + 部署的时候 直接上传`.output`目录就可以直接进行部署
  + 而nuxt.js需要上传整个项目代码，再进行对应的配置和部署
+ Nitro底层使用的框架是[h3](https://www.jsdocs.io/package/h3) 
  + h3是一个轻量级的h[ttp]框架，是基于node原生http模块进行二次开发
  + 相比node原生的http模块，具有更好的性能和效率
  + nitro可以方便的进行跨平台，是因为底层所使用的h3具有跨平台能力



**全栈开发**

nuxt中存在node服务，所以可以使用nuxt开发api服务器，单纯作为后端服务器或ssr服务器进行使用（类似于express）

使用esbuild，兼容旧版浏览器，支持最新的 JavaScript 语法转译



### Serverless

Serverless是一种云计算模型，它使开发人员可以在无需管理服务器或基础架构的情况下构建和运行应用程序。在传统的服务器模型中，开发人员需要自己管理服务器，配置和维护基础设施，包括服务器硬件、网络、操作系统、安全性等。而在Serverless模型中，云服务提供商（如AWS、Azure、Google Cloud）会负责这些基础设施的管理，开发人员只需要编写应用程序代码并上传到云平台上。

Serverless通常基于事件触发，应用程序代码在需要时自动被云服务提供商调用。这些事件可以是各种来源，例如API请求、数据库更改、文件上传等等。在Serverless模型中，开发人员只需要为这些事件编写代码，而无需考虑如何管理服务器。

Serverless的好处包括：

- 降低基础设施成本：无需购买服务器、配置和维护基础设施
- 按需计费：只需为实际使用的计算资源付费，无需支付闲置服务器的费用
- 更快的开发速度：可以更专注于应用程序代码的开发，而不需要关注基础设施的管理

常见的serverless运营商有阿里云，腾讯云，vercel，netify，cloudFlare



### 容器

容器是一个单独的运行环境，可以理解为是一个轻量级的虚拟化技术，它包含了所有代码运行所需要的环境，如操作系统、运行库、应用程序等

一个云上会存在多个容器，容器和容器之间相互独立运行，这种隔离可以避免不同项目之间出现环境冲突、版本冲突等问题，提高了应用程序的可靠性和安全性

容器在运行的时候会被进行打包，而构建出来的就是镜像

镜像是容器的静态表现形式，它包含了一个应用程序的运行环境和代码，可以被用于创建和运行容器



## 安装

```shell
$ npx nuxi init <project_name>

# dlx --- pnpm don't link (execute)
# 它的作用是将指定的 npm 包的二进制文件下载到本地并执行，而无需将其链接到全局环境或将其安装到项目的node_modules 目录下
# 简而言之 dlx的功能和npx是一致的

# 如果使用pnpm安装对应的依赖，需要加上参数--shamefully-hoist
# 或者在npmrc中配置shamefully-hoist=true
$ pnpm dlx nuxi init <project_name>

# 全局安装后 在进行项目初始化 --- 这种方式无法确保每一次创建项目所使用的脚手架都是最新的版本
$ npm install –g nuxi && nuxi init <project_name>

# 在使用nuxi初始化项目的时候，使用到了一个名为raw.githubusercontent.com的url
# 如果需要加载这个url，就需要在terminal中科学上网(注意是terminal，而不是浏览器)
# 或者在/etc/hosts文件中配置对应的域名和ip之间的映射 --- 185.199.108.133 raw.githubusercontent.com
```



## 运行脚本

```js
{
  "private": true, // 私有项目
  "scripts": {
    "build": "nuxt build", // 项目构建 --- 用于生成SPA应用
    "dev": "nuxt dev", // 开发项目
    // 项目构建，并预渲染好每一条路由 等价于 nuxt build --prerender
    // 用于生产SSG应用
    "generate": "nuxt generate",
    // 项目预览
    // 执行该命令，会在内存中使用生产环境脚本进行构建
    // 构建完成后，再进行预览
    // 所以执行preview命令之前，不需要执行nuxt build指令
    "preview": "nuxt preview", 
    // 一个npm的生命周期函数，会在npm install完成后自动执行
    // 用于对项目进行预编译(即安装完依赖就进行一次打包)和优化，以提升首次渲染速度
    // 在应用程序运行时，Nuxt.js 会从 .nuxt 目录中读取所需的文件和资源，并在内存中构建出应用程序的运行环境
    "postinstall": "nuxt prepare" 
  },
  "devDependencies": {
    "nuxt": "^3.2.2"
  }
}
```



## 目录结构

![image.png](https://s2.loli.net/2023/03/02/jDUMCybfNJocO7T.png)  

在Vue、React等前端框架中，index.html文件是作为应用的入口页面参与构建的。但是，为了更好地控制和管理这个入口文件，index.html文件通常被放置在public目录下而不是assets目录下

这样可以避免对index.html中引入public目录下对应资源的路径的处理，只需要处理assets资源对应路径即可，而assets资源对应路径本来就会交给构建工具进行处理



## 入口文件(App.vue)

```js
<template>
  <div>
    <NuxtWelcome />
  </div>
</template>
<script setup>
/*
  1. 定义页面布局Layout 或 自定义布局，如:NuxtLayout
  2. 定义路由的占位，如:NuxtPage
  3. 编写全局样式
  4. 全局监听路由 等等 
     因为nuxt是约定式路由，所以对于路由的监听只能编写在这里
*/
</script>
```



## 配置文件

nuxt3的默认配置文件，是位于项目根目录的`nuxt.config.ts`



### runtimeConfig

`runtimeConfig` 是在运行时被使用的变量，这些变量在不同的环境中可能会有不同的值， 这些变量也被称之为环境变量，动态环境变量

runtimeConfig会被序列化，所以不能使用undefined等不可被序列化的数据类型

```ts
// @nuxt.config.ts
// 配置优先级 如下:
// 命令行参数 > .env > nuxt.config.ts

// nuxt run dev --port 8080
// runtimeConfig 就是将--port 8080 写入到一个配置文件中

// defineNuxtConfig 是 nuxt中的辅助函数
// 对于vue的内置组件，和nuxt的内置组件以及辅助函数等
// 可以不导包 直接使用 虽然导包也可以

// defineNuxtConfig的功能是对配置对象添加类型注解
export default defineNuxtConfig({
  // 运行时环境变量, 也就是项目在运行的时候会被注入的全局变量
  runtimeConfig: {
    apiKey: 'key', // 定义在顶层的变量可以在server(服务端)访问
    public: { // 定义在public中的变量可以在server和client(客户端)访问
      baseUrl: 'https://www.example.com'
    }
  }
})
```

```vue
<script setup>
 // 获取运行时配置
 const runtimeConfig = useRuntimeConfig()

 // 判断在服务端还是在客户端也可以通过是否存在window来进行判断  
 
 // 判断是否在服务端渲染
 if (process.server) {
  console.log(runtimeConfig.apiKey) // => key
  console.log(runtimeConfig.public.baseUrl) // => https://www.example.com
 }

 // 判断是否在客户端渲染
 if (process.client) {
  console.log(runtimeConfig.apiKey) // => undefined
  console.log(runtimeConfig.public.baseUrl) // => https://www.example.com
 }
</script>
```



运行时配置除了可以在`nuxt.config.ts`的`runtimeConfig`中进行配置外，

还可以配置在项目根目录下的`.env`中

1. `.env`中的优先级高于`nuxt.config.ts`

   如果两者存在相同的配置，`.env`中的配置会高于`nuxt.config.ts`

2. `.env`是在程序运行的时候被注入到项目中，而`runtimeConfig`是会被构建到代码中

    所以对于一些敏感信息，如API密钥等，应该使用`.env`

3. `.env`编写的变量会注入到`process.env`和`import.meta.env`中, `runtimeConfig`不会

4.  `.env`一般用于某些终端启动应用时动态指定配置，同时支持dev和pro

    可以根据开发环境加载不同的变量和不同的值，`runtimConfig`不可以

所以 `.env` 文件通常用于应用程序的动态配置和保护敏感信息，而 `runtimeConfig` 通常用于应用程序的静态配置

```shell
# .env中 #表示注释
# .env文件不区分大小写，多个单词之间可以使用 - 或 _ 连接
# 推荐全部大写，并使用 _ 进行连接 ---> 约定俗成
# .env文件是在应用程序启动时加载的，并在运行时被缓存到内存中
# 如果要更改环境变量，必须重新启动应用程序才能使其生效
# 格式为 key = value

# 对于自定义的NUXT环境变量 必须以NUXT_开头
# 自定义环境变量 可以通过useRuntimeConfig方法获取

# server envrionment config
NUXT_API_KEY = 'apikey'

# server and client envrionment config
NUXT_PUBLIC_BASE_URL = 'http://foo.com'

# 也可以在env文件中配置命令行参数
# 命令行参数的NUXT_前缀是可选的 --- 一般不添加NUXT_前缀
# 命令行参数无法通过useRuntimeConfig方法获取
PORT = 3000
```



### appConfig

`appConfig` 是在构建时需要使用的变量的配置，这些变量在不同的环境中的值是相同的

所以这些变量又被称之为`静态环境变量`， 类似于全局的一个`consts.ts`

`nuxt.config.ts` 配置会和 `app.config.ts` 的配置合并

优先级 `app.config.ts > appConfig`

appConfig对应的值 可以是 任何的数据类型

```tsx
// define
// defineNuxtConfig也是一个辅助函数
export default defineNuxtConfig({
  // 在这里编写的属性，在项目构建的时候，就会被注入到生产代码中
  appConfig: {
    title: 'Hello Nuxt'
  }
})

// use
<script setup>
// 使用appConfig
const appConfig = useAppConfig()

// appConfg中的内容会在server和client中都被注入
// 在语言层面 appConfig可以被修改，但是在逻辑层面不应该被修改
// onMounted 生命周期仅会在client执行
onMounted(() => {
  document.title = appConfig.title
})
</script>
```

```ts
// app.config.ts --- 采用ESM语法
export default {
  title: 'Hello Nuxt'
}
```

![Snipaste_2023-03-10_00-02-41的副本.png](https://s2.loli.net/2023/03/10/SBE2gdLUhynM6za.png) 

### app

app属性对当前项目进行相应的配置

```js
export default defineNuxtConfig({
  app: {
    // 为每一个页面进行head设置 --- SEO 和 加载外部资源
    // 在nuxt项目中 没有index.html文件
    // 所以对应的配置写在这里
    head: {
     // 设置title
     title: 'Nuxt Page',
     // charset 和 viewport比较特殊 可以提到app顶层进行配置
     // charset: 'utf-8',
     // viewport: 'width=device-width, initial-scale=1.0, maximum-scalable=1.0, minmum-scalable=1.0, user-scalable=no',

     // 设置meta
     meta: [
      // 对象中的key是meta的属性名
      // 对象中的value是meta的属性值
      {
        charset: 'utf-8'
      },
      {
        name: 'viewport',
        content: 'width=device-width, initial-scale=1.0, maximum-scalable=1, minmum-scalable=1, user-scalable=no'
      },
      {
        name: 'keywords',
        content: 'nuxt3 config'
      },
      {
        name: 'description',
        content: 'nuxt3 config description'
      }
     ],
     // 设置script
     script: [
      {
				// 默认情况下，是插入到head中
        src: 'https://www.example.js'
      },
			
      {
        src: 'https://www.foo.js',
        // script插入到body的最后边
        body: true
      }
     ],
     // 设置link
     link: [
      {
        rel: 'shortcut icon',
        href: 'favicon.ico',
        type: 'image/x-icon'
      }
     ],
     style: [
      {
        // 有一个特殊的属性 children
        // children对应的属性值会被作为对应属性的子元素
        // 所以下述会被渲染为 <style>body { color: red }</style>
        children: `
          body {
            color: red;
          }
        `,
      },
     ],

     // 给body元素添加对应的属性
     // ps: 这里的值是对象不是数组
     bodyAttrs: {
      class: 'body-style'
     }
    }
  }
})
```

```js
const title = ref('app head')

// 使用 useHead宏函数 在页面进行动态配置
// 1. 在App.vue中使用 是对每一个页面的通用配置
// 2. 如果是在<page>.vue中使用 是对某一个具体页面的配置
// 优先级
// 内置组件 > 页面内部配置 > nuxt.config.ts
// <page>.vue > App.vue
useHead({
  // 支持使用ref作为value
  // nuxt.config.ts中是对head的静态配置
  // useHead是对head的动态配置
  title 
})
```

```html
<!--
也可以使用内置组件 设置head
1. 优先级 内置组件是最高的
2. 可以使用Head包裹，也可以不使用Head进行包裹
-->
<Head>
  <Title>NUXT PAGE TITLE</Title>
  <Meta name="keywords" content="nuxt3 vite" />
  <Style>
    body {
      color: red
    }
  </Style>
  <Link src="http://www.example.com" />
</Head>
```



### 其它属性

```ts
export default defineNuxtConfig({
  // ssr 默认值为true
  // 为true的时候 使用SSR
  // 为false的时候 使用CSR
  ssr: false,
  // 路径的别名，默认已配好
  // alias
  // 用于配置nuxt的插件
  // modules
  // 指定
  // 使用那种打包器进行构建
  // builder --- 默认是vite
})
```

