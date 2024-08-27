axios在浏览器会使用XHR来发送网络请求，在node端会使用http模块发送对应的网络请求

无论是在浏览器端还是node端，其api都是一致的，也就是说axios帮我们屏蔽了宿主环境的差异

axios不仅支持Promise风格API，而且对原生的XHR或fetch进行了扩展，如axios可以设置拦截器监听对应的请求和响应

axios支持多种请求方式

+ axios(config) --- 本质就是axios.request(config)
+ axios.request(config)
+ axios.get(url[, config])
+ axios.delete(url[, config])
+ axios.head(url[, config])
+ axios.post(url[, data[, config]])
+ axios.put(url[, data[, config]])
+ axios.patch(url[, data[, config]])
+ axios.all([]) --- 本质就是Promise.all() --- axios.all 是axios的静态方法，不存在于axios的实例对象上

> `axios.request(config)`的最核心方法，其余方法本质是对`axios.request(config)`的二次封装



## get

```js
import axios from 'axios'

// 默认发送的就是get请求
const res = await axios.request({
 url: 'https://httpbin.org/get?name=Klaus'
})

// axios返回的是axios响应封装对象 「 包含了响应体，配置项，状态码 ... 」
// 服务器实际返回的数据位于data属性上
console.log(res.data)
```

```js
// axios.get(url[, config])
// 传参方式一
await axios.get('https://httpbin.org/get?name=Klaus')

// 传参方式二
await axios.get('https://httpbin.org/get', {
  params: {
    name: 'Klaus'
  }
})
```



## post

```js
import axios from 'axios'

const res = await axios.request({
  url: 'https://httpbin.org/post',
  // 更新请求方式 默认get，不区分大小写
  method: 'post',
  // 通过data传递post参数
  data: {
    id: 500665346
  }
})

console.log(res.data)
```

```js
import axios from 'axios'

// axios.post(url[, config])
const res = await axios.post('https://httpbin.org/post', {
  data: {
    id: 500665346
  }
})

console.log(res.data)
```

```js
import axios from 'axios'

// axios.post(url[, data])
const res = await axios.post('https://httpbin.org/post', {
  id: 500665346
})

console.log(res.data)
```



## 常见配置项

| 配置项                 | 示例                                          |
| ---------------------- | --------------------------------------------- |
| 请求地址               | url: '/user'                                  |
| 请求类型 - 默认值是get | method: 'get'                                 |
| 基准路径               | baseURL: 'http://www.example.com/api'         |
| 请求前的数据处理       | transformRequest:[function(data){}]           |
| 请求后的数据处理       | transformResponse: [function(data){}]         |
| 自定义请求头           | headers:{'x-Requested-With':'XMLHttpRequest'} |
| URL查询对象            | params:{ id: 12 }                             |
| request body           | data: { key: 'aa'}                            |
| 超时设置               | timeout: 1000                                 |

```js
import axios from 'axios'

// 通过axios.defaults.xxx 来覆盖axios的默认公共配置
axios.defaults.baseURL = 'http://www.example.com' // 基准路径
// axios.defaults.headers = {} // 清空所有请求头
axios.defaults.headers.token = 'xxxx' // 设置请求头
axios.defaults.timeout = 10000 // 超时时间

const res = await axios.all([
  // 传入的路径如果不是完整url会自动和baseURL进行路径拼接
  axios.post('https://httpbin.org/post', {
    id: 500665346
  }),
  axios.get('https://httpbin.org/get?name=Klaus')
])

console.log(res)
```



## 创建实例

```js
// axios 导入的是 axios的一个实例对象
import axios from 'axios'

// axios.create方法返回新的axios实例
// 参数是配置对象 => 实际生效的配置对象为 Object.assign(axios.defaluts, 传入的配置对象)
const api = axios.create({
  baseURL: 'https://httpbin.org/'
})

const res = await api.get('/get', {
  params: {
    name: 'Klaus'
  }
})

console.log(res.data)
```



## 拦截器

可以在发送请求或接收响应前设置拦截器，拦截器本质上是一种中间件机制，允许你在请求或响应的生命周期中插入自定义逻辑

每个拦截器都可以传入请求成功拦截函数和请求失败拦截函数

拦截器可以添加多个，多个拦截器对应的函数会组合成数组并依次进行调用

如果不提供拦截函数，Axios 会执行默认的行为，直接返回请求配置对象或响应数据，或者在失败时返回错误对象。

+ axios.interceptors.request.use(请求成功拦截, 请求失败拦截)
+ axios.interceptors.response.use(响应成功拦截, 响应失败拦截)

```js
import axios from 'axios'

const api = axios.create({
	baseURL: 'https://httpbin.org/',
	timeout: 10000
})

// 请求拦截 --- 设置请求头 或 开启loading动画
api.interceptors.request.use(config => {
	console.log('请求拦截成功')
  // 请求拦截参数是配置对象，返回值也需要是配置对象
	return config
}, err => {
	console.log('请求拦截失败的回调')
  // 失败拦截参数是错误信息，返回值也需要是错误对象
	return err
})

// 定义响应拦截 --- 关闭loading动画 或 对返回值进行处理 「 如直接返回res.data 」
api.interceptors.response.use(res => {
  console.log('响应拦截成功')
	return res.data
}, err => {
  // 此时，错误拦截函数和默认错误拦截函数一致 可以省略
	return err
})

const res = await api.get('/get?name=klaus')
console.log(res)
```



## 简单封装

实际使用时，会对axios进行封装，即在axios和项目之间多增加一层，从而减少项目和axios的耦合度

如果哪天axios发生了重大更新或者不维护的时候，需要修改的也只有封装的部分仅此而已



对于网络请求的封装，一般会放在`services`或`api`目录下

```ts
// @ /service/config.ts -- 有时候也会写入环境变量

// 常量 全部字符大写表示 单词和单词之间使用下划线进行连接
export const BASE_URL = 'http://www.example.com:8000'
export const TIMEOUT = 10000
```

```ts
// @ /service/api.ts
import axios from 'axios'
import type { AxiosRequestConfig, AxiosInstance } from 'axios'

// 关联度比较高的属性和方法一般会单独封装成一个类
class API {
  instance: AxiosInstance

  constructor(config: AxiosRequestConfig) {
    this.instance = axios.create(config)
  }

  request(config: AxiosRequestConfig) {
    return this.instance.request(config)
  }
}

export default API
```



## 拦截器

```js
import axios from 'axios'
import type { AxiosRequestConfig, AxiosInstance } from 'axios'


class API {
  instance: AxiosInstance

  constructor(config: AxiosRequestConfig) {
    this.instance = axios.create(config)
    
    // 这里是全局拦截器，所有实例共有
    this.instance.interceptors.request.use(config => config, err => err)
    this.instance.interceptors.response.use(res => res.data, err => err)
    
    // 如果实例配置中有拦截器，则添加实例特有的拦截器
    this.instance.interceptors.request.use(
     	config.interceptors?.requestResolved, 
     	config.interceptors?.requestRejected
   	)

    this.instance.interceptors.response.use(
      config.interceptors?.responseResolved,
      config.interceptors?.responseRejected
    )
  }

  request(config: AxiosRequestConfig) {
    return this.instance.request(config)
  }
}

export default API
```

```js
import type { AxiosRequestConfig, AxiosResponse } from 'axios'

// 可以通过 接口继承的方式 扩展axios原有配置类型，以增加自定义配置
export interface RequestConfig<T = AxiosResponse> extends AxiosRequestConfig {
  interceptors?: {
    requestResolved?: (config: AxiosRequestConfig) => AxiosRequestConfig;
    requestRejected?: (err: Error) => Error;
    responseResolved?: (res: T) => T;
    responseRejected?: (err: Error) => Error;
  }
}
```



```js
import axios from 'axios'
import type { AxiosInstance } from 'axios'
import type { RequestConfig, IData } from './type'

class API {
  instance: AxiosInstance

  constructor(config: RequestConfig) {
    this.instance = axios.create(config)

    this.instance.interceptors.request.use(config => config, err => err)
    this.instance.interceptors.response.use(res => res.data, err => err)

    this.instance.interceptors.request.use(config.interceptors?.requestResolved, config.interceptors?.requestRejected)
    this.instance.interceptors.response.use(config.interceptors?.responseResolved, config.interceptors?.responseRejected)
  }

  // 用于对某个接口单独设置拦截器
  // 不能挂载到axios实例上，否则会被加入拦截器数组 「 被多个接口调用所复用 」
  // 所以只能直接对传入的拦截器进行调用
  request<T>(config: RequestConfig<T>) {
    try {
      if (config.interceptors?.requestResolved) {
        config = config.interceptors.requestResolved(config)
      }

      return new Promise<T>(async (resolve, reject) => {
          try {
            const res = await this.instance.request<any, T>(config)
            if (config.interceptors?.responseResolved) {
              res = config.interceptors.responseResolved(res)
            }
            resolve(res)
          } catch(err) {
            if (config.interceptors?.responseRejected) {
              err = config.interceptors.responseRejected(err)
            }
            reject(err)
          }
      })
    } catch(err) {
      if (config.interceptors?.requestRejected) {
        config = config.interceptors.requestRejected(err)
      }
    }
  }

  // 其余方法的调用本质是axios.request方法的语法糖写法
  get<T>(url: string, config?: RequestConfig<T>) {
    return this.request<T>({url, ...config, method: 'GET'})
  }

  post<T>(url: string, data: IData, config?: RequestConfig<T>) {
    return this.request<T>({url, data, ...config, method: 'POST'})
  }

  delete<T>(url: string, config?: RequestConfig<T>) {
    return this.request<T>({url, ...config, method: 'DELETE'})
  }

  put<T>(url: string, data: IData, config?: RequestConfig<T>) {
    return this.request<T>({url, data, ...config, method: 'PUT'})
  }

  patch<T>(url: string, data: IData, config?: RequestConfig<T>) {
    return this.request<T>({url, data, ...config, method: 'PATCH'})
  }
}

export default API
```

