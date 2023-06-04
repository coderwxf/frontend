在实际开发中，在多个组件中都需要进行网络请求，那么就需要在多个组件中都需要使用axios来进行网络请求

这就会导致项目和axios的耦合度，也就是关联度过高，如果哪天axios发生了重大更新或者不维护的时候，需要修改和变动的地方就非常多

所以在实际开发中，会单独对网络请求进行二次封装，也就是在axios和业务逻辑之间添加上单独的一层

这样对于项目业务逻辑而言使用的是自己封装的内容，实际在项目中对于axios的引用只有一处

这样可以减低项目和axios之间的耦合度，便于后续的升级和维护



## 简单封装

`/service/index.ts`

```ts
import { BASE_URL, TIMEOUT } from './config';
import API from './api'

export const api = new API({
  baseURL: BASE_URL,
  timeout: TIMEOUT
})
```



`/service/config.ts`

将axios的配置选项进行抽离，方便在多处使用相同的配置选项，也方便后期对这些配置选项进行修改和维护

```ts
// 常量 全部字符大写表示 单词和单词之间使用下划线进行连接
export const BASE_URL = 'http://www.example.com:8000'
export const TIMEOUT = 10000
```



`/service/api.ts`

```ts
import axios from 'axios'
import type { AxiosRequestConfig, AxiosInstance } from 'axios'

// 对于关联度比较高的属性和方法一般会单独封装成一个类
// 一般对应的网络请求方法会被封装到src/api文件夹中或src/services文件夹下
class API {
  instance: AxiosInstance

  constructor(config: AxiosRequestConfig) {
    // 每次创建实例的时候，都调用axios.create方法
    // axios.create可以返回一个axios实例
    // 这样保证我们可以使用不同的配置创建多个axios实例
    this.instance = axios.create(config)
  }

  // request是最为核心的方法
	// get, post等方法 本质上调用的还是request方法
  request(config: AxiosRequestConfig) {
    return this.instance.request(config)
  }
}

export default API
```



`测试`

```ts
import { api } from './service'

async function fetchMultidata() {
  const res = await api.request({
    url: '/home/multidata'
  })

  console.log(res.data)
}

fetchMultidata()
```



## 拦截器

### 全局拦截器

全局拦截器是那些所有实例对象都需要使用的拦截器

```ts
import axios from 'axios'
import type { AxiosRequestConfig, AxiosInstance } from 'axios'
import type { RequestConfig } from './type'

class API {
  instance: AxiosInstance

  constructor(config: RequestConfig) {
    this.instance = axios.create(config)

    // 这里编写所有组件实例都需要进行的请求和响应拦截
    // 在请求拦截中 可以修改配置/加载loading动画/添加token等
    // 在响应拦截中，可以对返回的数据进行二次处理后再返回 或者关闭loading动画
  	this.instance.interceptors.request.use(config => config, err => err)
    this.instance.interceptors.response.use(res => res.data, err => err)
  }

  request(config: AxiosRequestConfig) {
    return this.instance.request(config)
  }
}

export default API
```



### 实例拦截器

实例拦截器是针对于某一个具体的axios实例所需要使用的拦截器

`type.ts`

```ts
import type { AxiosRequestConfig, AxiosResponse } from 'axios'

// 继承自axios的AxiosRequestConfig 是对AxiosRequestConfig的扩展
export interface RequestConfig extends AxiosRequestConfig {
  interceptors?: {
    requestResolved?: (config: AxiosRequestConfig) => AxiosRequestConfig;
    requestRejected?: (err: any) => any;
    responseResolved?: (res: AxiosResponse) => AxiosResponse;
    responseRejected?: (err: any) => any;
  }
}
```



`api.ts`

```ts
import axios from 'axios'
import type { AxiosRequestConfig, AxiosInstance } from 'axios'
import type { RequestConfig } from './type'

class API {
  instance: AxiosInstance

  constructor(config: RequestConfig) {
    this.instance = axios.create(config)

    this.instance.interceptors.request.use(config => config, err => err)
    this.instance.interceptors.response.use(res => res.data, err => err)

    // 拦截器 对应use方法的 成功回调函数和失败回调函数 都是可选的
    // 拦截器可以添加多个，多个拦截器对应的函数会组合成数组并依次进行调用
    this.instance.interceptors.request.use(config.interceptors?.requestResolved, config.interceptors?.requestRejected)
    this.instance.interceptors.response.use(config.interceptors?.responseResolved, config.interceptors?.responseRejected)
  }

  request(config: AxiosRequestConfig) {
    return this.instance.request(config)
  }
}

export default API
```



`测试`

```ts
import { BASEURL, TIMEOUT } from './config';
import API from './api'

export const api = new API({
  baseURL: BASE_URL,
  timeout: TIMEOUT,
  interceptors: {
    requestResolved(config) {
      // 在这里可以对某一个实例的请求进行单独的拦截处理
      console.log('全局成功的请求拦截')
      return config
    },

    responseResolved(res) {
      // 在这里可以对某一个实例的响应进行单独的拦截处理
      console.log('全局成功的响应拦截')
      return res.data
    }
  }
})
```





### 接口拦截器

接口拦截器是针对于某一个具体的请求所需要使用到的拦截器

```ts
import axios from 'axios'
import type { AxiosInstance } from 'axios'
import type { RequestConfig } from './type'

class API {
  instance: AxiosInstance

  constructor(config: RequestConfig) {
    this.instance = axios.create(config)

    this.instance.interceptors.request.use(config => config, err => err)
    this.instance.interceptors.response.use(res => res.data, err => err)

    this.instance.interceptors.request.use(config.interceptors?.requestResolved, config.interceptors?.requestRejected)
    this.instance.interceptors.response.use(config.interceptors?.responseResolved, config.interceptors?.responseRejected)
  }

  request(config: RequestConfig) {
    // 接口拦截器不能直接使用instance.interceptors
    // 因为instance.interceptors是挂载到axios实例对象上的
    // 这意味着后续所有的操作都会存在接口级别的网络请求
    // 所以添加接口拦截器最好的方法就是手动执行传入的回调

    try {
      if (config.interceptors?.requestResolved) {
        config = config.interceptors.requestResolved(config)
      }

      return new Promise((resolve, reject) => {
          this.instance.request(config).then(res => {
            if (config.interceptors?.responseResolved) {
              res = config.interceptors.responseResolved(res)
            }
            resolve(res)
          }).catch(err => {
            if (config.interceptors?.responseRejected) {
              err = config.interceptors.responseRejected(err)
            }
            reject(err)
          })
      })
    } catch(err) {
      if (config.interceptors?.requestRejected) {
        config = config.interceptors.requestRejected(err)
      }
    }
  }
}

export default API
```



`测试`

```ts
import { api } from './service'

async function fetchMultidata() {
  const res = await api.request({
    url: '/home/multidata',
    interceptors: {
      requestResolved(config) {
        console.log('接口层面的请求成功拦截')
        return config
      },
      responseResolved(res) {
        console.log('接口层面的响应成功拦截')
        return res
      }
    }
  })

  console.log(res)
}

fetchMultidata()
```



## 类型问题

但是此时会发现返回的结果都是unlnown类型，因为默认情况下axios无法知道我们请求的实际返回值类型

所以我们希望通过添加泛型的方式，手动告诉axios对应的返回结果的类型



`type.ts`

```ts
import type { AxiosRequestConfig, AxiosResponse } from 'axios'

export interface RequestConfig<T = AxiosResponse> extends AxiosRequestConfig {
  interceptors?: {
    requestResolved?: (config: AxiosRequestConfig) => AxiosRequestConfig;
    requestRejected?: (err: any) => any;
    responseResolved?: (res: T) => T;
    responseRejected?: (err: any) => any;
  }
}
```



`api.ts`

```ts
import axios from 'axios'
import type { AxiosInstance } from 'axios'
import type { RequestConfig } from './type'

class API {
  instance: AxiosInstance

  constructor(config: RequestConfig) {
    this.instance = axios.create(config)

    this.instance.interceptors.request.use(config => config, err => err)
    this.instance.interceptors.response.use(res => res.data, err => err)

    this.instance.interceptors.request.use(config.interceptors?.requestResolved, config.interceptors?.requestRejected)
    this.instance.interceptors.response.use(config.interceptors?.responseResolved, config.interceptors?.responseRejected)
  }

  request<T>(config: RequestConfig<T>) {
    try {
      if (config.interceptors?.requestResolved) {
        config = config.interceptors.requestResolved(config)
      }

      // Promise如果不传泛型 那么默认的返回值的类型就是unknown
      return new Promise<T>((resolve, reject) => {
          this.instance.request<any, T>(config).then(res => {
            if (config.interceptors?.responseResolved) {
              res = config.interceptors.responseResolved(res)
            }
            resolve(res)
          }).catch(err => {
            if (config.interceptors?.responseRejected) {
              err = config.interceptors.responseRejected(err)
            }
            reject(err)
          })
      })
    } catch(err) {
      if (config.interceptors?.requestRejected) {
        config = config.interceptors.requestRejected(err)
      }
    }
  }
}

export default API
```



## 其余方法的简单封装

`type.ts`

```ts
import type { AxiosRequestConfig, AxiosResponse } from 'axios'

export interface IData {
  [key: string]: unknown
}

export interface RequestConfig<T = AxiosResponse> extends AxiosRequestConfig {
  interceptors?: {
    requestResolved?: (config: AxiosRequestConfig) => AxiosRequestConfig;
    requestRejected?: (err: any) => any;
    responseResolved?: (res: T) => T;
    responseRejected?: (err: any) => any;
  }
}
```



```ts
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

  request<T>(config: RequestConfig<T>) {
    try {
      if (config.interceptors?.requestResolved) {
        config = config.interceptors.requestResolved(config)
      }

      return new Promise<T>((resolve, reject) => {
          this.instance.request<any, T>(config).then(res => {
            if (config.interceptors?.responseResolved) {
              res = config.interceptors.responseResolved(res)
            }
            resolve(res)
          }).catch(err => {
            if (config.interceptors?.responseRejected) {
              err = config.interceptors.responseRejected(err)
            }
            reject(err)
          })
      })
    } catch(err) {
      if (config.interceptors?.requestRejected) {
        config = config.interceptors.requestRejected(err)
      }
    }
  }

  // 对于get，post等请求，config参数可以不传递
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

