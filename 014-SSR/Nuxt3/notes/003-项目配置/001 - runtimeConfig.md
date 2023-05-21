运行时配置，也就是项目运行的时候才会被注入的配置， 所以经常被用来定义环境变量

常见的配置位置有`runtimeConfig`和`env文件	`

如果env和runtimeConfig配置相同，将优先选择使用env文件中对应的配置

```ts
export default defineNuxtConfig({
  runtimeConfig: {
    // 在这里定义的变量只能在服务端被获取
    apiKey: 'key',
    // 在public中定义的变量即可以在服务端被获取 也可以在客户端被获取
    public: {
      baseUrl: 'https://www.example.com'
    }
  }
})
```

```ts
const config = useRuntimeConfig()

// process.server 是nuxt在运行时注入的 用于标识当前是服务端渲染流程
if (process.server) {
  console.log(config.apiKey)
  console.log(config.public.baseUrl)
}

// process.client 是nuxt在运行时注入的 用于标识当前是客户端渲染流程
if (process.client) {
  console.log(config.public.baseUrl)
}

// 除了判断process.server的值以确定是否在服务端渲染
// 还可以通过判断window的值是否为object或者null来判断是客户端渲染还是服务端渲染
```



#### .env

在nuxt项目中，同样可以使用env文件定义环境变量

env文件一般位于项目的根目录，且以`.env`作为文件的后缀名

env文件最终会经过`dotenv`这个第三方库解析，并将对应的环境变量注入到`process.env`中

和`runtimeConfig`不同的是，`runtimeConfig`中的配置文件最终会被构建到生产代码中

而env文件不会，他仅仅只是在项目运行的时候进行注入，所以使用env文件可以根据不同的环境注入不同的值

```shell
# .env.dev --- 开发环境使用的环境变量

# 在.env文件中 使用#进行注释
# 在.env中变量名 全大写 多个单词之间使用下划线进行分割

# 使用NUXT_ 开头的会覆盖服务端 同名环境变量
NUXT_API_KEY = 'apiKey'

# 使用NUXT_PUBLIC_开头的 会覆盖服务器和客户端的 同名通用配置
NUXT_PUBLIC_BASE_URL = 'https://www.localhost'

# 如果这个值是作为nuxt参数传递的 可以省略NUXT_前缀 ---> nuxt dev --port 9090
# 也就是此时 NUXT_PORT 等价于 PORT
PORT = 9090
```

```ts
import dotenv from 'dotenv'

// dotenv默认会加载项目根目录下的.env文件
// 如果自定义了env文件的名称，需要使用config函数进行加载
dotenv.config({
  // webpack 和 Vite 都提供了 process.env.NODE_ENV 环境变量，用于判断当前是生产环境还是开发环境
  // 开发环境 -> development  生产环境 -> production
  path: process.env.NODE_ENV === 'development' ? './.env.dev' : './.env.prod'
})

// 在env文件中定义的变量会被注入到process.env中
// key为在env文件中定义的属性名
console.log(process.env.NUXT_API_KEY)
console.log(process.env.NUXT_PUBLIC_BASE_URL)
```

