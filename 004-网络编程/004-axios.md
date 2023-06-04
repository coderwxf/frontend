在实际开发中，我们经常需要进行网络请求，如果使用原生提供的方法，会存在如下问题：

1. 需要对原生的XHR或fetch进行封装，以方便使用
2. 在浏览器中可以使用XHR或fetch，但是在node环境中只能使用http模块来发送网络请求
3. 某些情况下，我们需要拦截对应的请求和响应操作，以完成对应的功能

所以在实际开发中，我们更推荐的是使用第三方库来进行网络请求，这个库就是[axios](https://github.com/axios/axios)

axios在浏览器会使用XHR来发送网络请求，在node端会使用http模块发送对应的网络请求

无论是在浏览器端还是node端，其api都是一致的，也就是说axios帮我们屏蔽了宿主环境的差异

axios不仅支持Promise风格API，而且对原生的XHR或fetch进行了扩展，如axios可以设置拦截器监听对应的请求和响应

以方便我们执行对应的自定义操作



## 基本使用

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



### get请求

```js
// 方式一 --- 不配置method 默认情况下 使用的是get请求
const res = await axios.request({
  url: 'http://www.example.com/lyric?id=500665346'
})

// axios返回的响应对象是axios封装的对象，其中包括对应的响应体，配置项，状态码 ...
// 如果我们需要获取到实际服务器返回的数据，需要通过其中的data属性
console.log(res.data)
```

```js
// axios.get(url[, config])
// 方式二 - query参数直接跟在url后边
await axios.get('http://www.example.com/lyric?id=500665346')
```

```js
// 方式三 - query参数写在config配置项中
await axios.get('http://www.example.com/lyric', {
  // 使用params接收query参数
  params: {
    id: 500665346
  }
})
```



### post请求

```js
// axios.post(url[, data][, config])
// 当使用axios.post方法的时候，axios会自动将method配置项的值修改为post
// 参数一 - url地址
// 参数二- 配置对象
await axios.post('http://www.example.com:1888/02_param/postjson', {
  // 使用data定义post方法需要传递的参数
  data: {
    id: 500665346
  }
})
```

```js
// axios.post也支持以如下方式进行参数的传递
// 1. 参数一 - url地址
// 2. 参数二 - 需要传递的data参数
// 3. 参数三 - 配置对象
await axios.post('http://www.example.com:1888/02_param/postjson', {
  id: 500665346
})
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



## 基准路径

```js
import axios from 'axios'

// 可以通过axios.defaults来配置公共配置
axios.defaults.baseURL = 'http://www.example.com' // 基准路径
// 公共头配置信息是一个对象
// 一般情况下执行的是类似于axios.defaults.headers[token] = 'xxxx’
// 下述的操作是直接清空对应的请求头信息
axios.defaults.headers = {} // 公共头信息
axios.defaults.timeout = 10000 // 超时时间

async function foo() {
	// axios.all 本质就是 Promise.all
	const res = await axios.all([
		axios.get('/home/multidata'),
		// 如果传入的是全路径地址，那么就是要全路径地址
		// 如果传入的不是全路径地址，axios内部会通过new URL(传入的路径, baseURL)
		axios.get('http://api.example.com/lyric?id=500665346')
	])

	console.log(res)
}

foo()
```



## 创建实例

当我们从axios模块中导入对象时, 使用的实例是默认的实例

当给该实例设置一些默认配置时, 这些配置就被固定下来了

但是后续开发中, 某些配置可能会不太一样, 比如某些请求需要使用特定的baseURL或者timeout等

这个时候, 我们就可以创建新的实例, 并且传入属于该实例的配置信息

```js
import axios from 'axios'

// 全局的公共配置 --- 也就是针对默认实例的配置
axios.defaults.baseURL = 'http://www.example.com:8000'

// 针对于某一个特定实例的配置 - 会执行类似于Object.assign(默认配置, 特定配置)的操作
const api = axios.create({
  baseURL: 'http://www.example.com:9001',
  timeout: 10000
})

async function foo() {
  const res = await api.get('/lyric?id=500665346')
  console.log(res.data)
}

foo()
```



## 拦截器

axios的也可以设置拦截器:拦截每次请求和响应

+ axios.interceptors.request.use(请求成功拦截, 请求失败拦截)
+ axios.interceptors.response.use(响应成功拦截, 响应失败拦截)

> 拦截器必须在发送请求或接收响应前被设置

```js
import axios from 'axios'

axios.defaults.baseURL = 'http://www.example.com:8000'

const api = axios.create({
	baseURL: 'http://www.example.com:9001',
	timeout: 10000
})

// 对于请求拦截和响应拦截 都有两个参数，分别为请求成功时候的回调和请求失败时候的回调
// 如果不传，默认函数就是直接将传入的参数原封不动的返回

// 定义请求拦截
// 一般会在请求拦截中，开启loading动画或者修改配置项(如在header中添加上token等)
api.interceptors.request.use(config => {
	console.log('请求拦截成功')
	// 请求拦截成功的回调
	// 会将配置信息传递过来，可以对config进行任何想要的修改后
	// 在返回新的配置信息
	return config
}, err => {
	console.log('请求拦截失败的回调')
	// 错误回调 会接收错误信息信息作为参数
	// 在错误回调中可以执行任何所需要的错误处理
	// 需要返回对应的错误对象
	return err
})

// 定义响应拦截
axios.interceptors.response.use(res => {
	// 响应拦截成功时候触发的回调函数
	// 会将结果作为参数传入，需要将处理后结果返回
	// 在响应拦截成功回调中，经常执行如下操作:
	// 1. 关闭loading动画
	// 2，对返回的数据进行处理，如取出其中的data属性后再进行返回
	return res.data
}, err => {
	// 响应拦截错误时候触发的回调函数
	// 会将错误传入，需要返回经过处理后的错误对象
	return err
})

async function foo() {
	const res = await api.get('/lyric?id=500665346')
	console.log(res.data)
}

foo()
```



## 封装

[使用TS对axios进行封装](./005-%E4%BD%BF%E7%94%A8ts%E5%AF%B9axios%E8%BF%9B%E8%A1%8C%E5%B0%81%E8%A3%85.md)