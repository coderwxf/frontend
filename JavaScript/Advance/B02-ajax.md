AJAX 是异步的 JavaScript 和 XML (Asynchronous JavaScript And XML)的简称

AJAX可以使用 JSON，XML，HTML 和 text 文本等格式发送和接收数据

AJAX仅存在于浏览器，不存在于node

```js
const xhr = new XMLHttpRequest()

// 配置网络请求(通过open方法)
// 参数1 - 交互方式 - 大小写不区分
// 参数2 - URL请求地址 「 非字符串会先转字符串后再使用 」
// 参数3 - 同步还是异步处理 「 默认值是true，即异步处理 」
xhr.open('get', 'https://httpbin.org/get?name=Klaus&age=23')

xhr.addEventListener('readystatechange', () => {
  if (xhr.readyState === XMLHttpRequest.DONE) {
    // 默认情况下，response属性是字符串类型值
    console.log(JSON.parse(xhr.response))
  }
})

xhr.send()
```



## XHR状态

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/b37882065b994cdeaf40fe6f9dfa18d5~tplv-k3u1fbpfcp-zoom-1.image#?w=2060\&h=564\&s=126310\&e=png\&b=fae7d8) 

```js
const xhr = new XMLHttpRequest()

// xhr被创建的时候，xhr.readyState的默认值为0
console.log(xhr.readyState) // => 0

// xhr的配置和事件监听 没有绝对的先后顺序
xhr.addEventListener('readystatechange', () => {
  console.log(xhr.readyState)
  /*
    =>
      1
      2
      3
      4
  */

  // xhr.readyState状态值存在一一对应的常量值
  // 如果只想监听数据全部加载完成，可以将readystatechange替换为onload
  if (xhr.readyState === XMLHttpRequest.DONE) {
    console.log(xhr.response)
  }
})

xhr.open('get', 'https://httpbin.org/get?name=Klaus&age=23')

xhr.send()
```

```js
const xhr = new XMLHttpRequest()

xhr.open('get', 'https://httpbin.org/get?name=Klaus&age=23', false)

xhr.send()

console.log(xhr.response)
```



## 常见事件

| 事件      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| loadStart | `xhr.readyState` 的值为 1，刚开始请求。这个事件在请求开始时触发 |
| progress  | 请求加载中，`xhr.readyState` 的值为 2 或 3。<br />这个事件在数据接收过程中定期触发，<br />通常每 50ms 触发一次（具体间隔可能因浏览器而异）。 |
| abort     | 请求被取消，通常是调用了 `xhr.abort()` 方法时触发            |
| load      | 加载完成，`xhr.readyState` 的值为 4。<br />只要服务器返回了结果，不论 HTTP 状态码是什么都会触发该事件。 |
| timeout   | 请求超时触发。默认超时时间为 0，即没有超时时间设置           |
| error     | 请求失败时触发，例如网络中断或跨域错误等情况<br />注意，如果 HTTP 状态码不是 2xx，但请求完成并成功返回响应（即 `xhr.readyState` 为 4），则不会触发 `error` 事件，而是 `load` 事件 |
| loadend   | 请求结束时触发，不论是通过 `load`、`error`、`timeout` 还是 `abort` 事件结束的。只要请求结束，就会触发 `loadend` 事件 |

```js
const xhr = new XMLHttpRequest()

xhr.responseType = 'json'

// typeof e -> ProgressEvent
xhr.addEventListener('progress', e => {
  console.log(e.loaded) // => 已经接收的字节数
  console.log(e.total) // => 根据Content-Length响应头部确定的预期字节数
  console.log(e.lengthComputable) // => 一个布尔值 表示是否加载完全
  // 有了上述信息后，调用者可以自主决定如何展示对应的加载进度信息
})

xhr.open('get', 'http://www.httpbin.org/get?name=Klaus&age=23')

xhr.send()
```



## 响应类型

可以通过xhr的response获取返回结果，默认情况下，responseType的值为空字符串，response就会被认为是字符串类型值

| responseType 值                         | response对应类型                                             |
| --------------------------------------- | ------------------------------------------------------------ |
| 空字符串 「 默认值 ，效果和`text`一致」 | 字符串                                                       |
| text                                    | 字符串                                                       |
| xml                                     | `response` 是一个 `Document` 对象，<br />通常是解析后的 XML 或 HTML |
| json                                    | 解析后的 JSON 对象                                           |

```js
// 设置超时时间 单位毫秒 推荐10s 「 后续即使服务器给数据了，客户端也不要了 」
xhr.timeout = 3000

// response 返回的数据格式 (可以解析为普通文件 或 json)
console.log(xhr.response)

// 如果返回的类型为text，可以通过responseText获取返回结果
console.log(xhr.responseText)

// 如果返回的类型为xml，可以通过responseXML 来获取对应的解析后的xml内容
console.log(xhr.responseXML)

// 状态码 状态描述
console.log(xhr.status, xhr.statusText) // 404 'NOT FOUND'

// 取消请求 「 服务器请求还在，还会给结果，客户端不要了 」
xhr.abort()
```



## 参数传递

常见的参数传递方式有如下几种格式:

1.  GET请求的query参数
2.  POST请求 x-www-form-urlencoded 格式
3.  POST请求 FormData 格式
4.  POST请求 JSON 格式

```js
const xhr = new XMLHttpRequest()

const searchParams = new URLSearchParams()
searchParams.append('name', 'Klaus')
searchParams.append('age', 23)

// get请求在传递数据的时候，需要将参数挂载为URL的query参数
// http://www.httpbin.org/get?name=Klaus&age=23
xhr.open('get', `http://www.httpbin.org/get?${searchParams.toString()}`)

xhr.addEventListener('load', () => console.log(xhr.response))

// 让xhr将返回结果转json解析后的对象
xhr.responseType = 'json'

xhr.send()
```

 

```js
const xhr = new XMLHttpRequest()

xhr.open('POST', 'http://www.httpbin.org/post')

xhr.addEventListener('load', () => console.log(xhr.response))

xhr.responseType = 'json'

// 默认情况下，post请求方式中 请求体中的数据 是按照text/plain的方式进行传输和解析的
// 所以如果传递的数据是按照x-www-form-urlencoded进行传输的时候
// 需要手动设置请求头Content-type 为 application/x-www-form-urlencoded

// 设置请求头 请求头不区分大小写 不过请求数据编码类型一遍写为 Content-type
// setRequestHeader方法必须在open方法后才可以进行调用
xhr.setRequestHeader('Content-type', 'application/x-www-form-urlencoded')

// post请求传递参数的时候
// 如果参数使用 x-www-form-urlencoded方式传递数据的时候
// 需要将对应的参数以urlencoded方式进行编码后 作为send方法的参数被传入
// 也就是以key=value&key=value的字符串形式 「 非字符串形式会自动转字符串 」
xhr.send(searchParams.toString())
})
```



```js
const xhr = new XMLHttpRequest()

xhr.open('POST', 'http://www.httpbin.org/post')

xhr.responseType = 'json'
xhr.addEventListener('load', () => console.log(xhr.response))

// 同样默认post参数传递时按照字符串格式进行传递，如果需要传json，需要指定传输格式
// 设置Content-typed的时候可以在编码后边指定文件的类型 「 如 application/json; charset=utf-8 」
// 如果不指定文件类型的时候，分号必须省略，且默认的文件编码格式为utf-8
xhr.setRequestHeader('Content-type', 'application/json')

// post的参数以send参数形式进行传达
// 如果是JSON对象，必须转JSON格式字符串 
// 因为不同语言，JSON字符串解析出的格式都是不一样的，如JAVA是javabean
xhr.send(JSON.stringify({name: 'Klaus', age: 23}))
```



```html
<form id="form">
  <input type="text" name="username">
  <input type="password" name="password">
</form>
<button id="btn">send</button>

<script>
  const el = document.getElementById('btn')

  el.addEventListener('click', () => {
    const xhr = new XMLHttpRequest()

    const formEl = document.forms[0]

    xhr.open('POST', 'http://www.example.com/postform')

    xhr.addEventListener('load', () => console.log(xhr.response))

    xhr.responseType = 'json'

    // 如果要提交表单数据，可以使用FormData 结合表单元素 生成对应的formData实例
    const formData = new FormData(formEl)

    // post传递数据依旧通过send参数
    // 区别是传递数据的时候 如果浏览器发送传递的数据类型为formData的时候
    // 自动将Content-type设置为multipart/form-data，不需要手动设置
    xhr.send(formData)
  })
</script>
```



## 简单封装

```js
// 参数比较多，所以封装为一个配置对象
function request({
  url,
  method = 'get',
  header = {},
  timeout = 10000,
  isRequestJson = true,
  data = {}
} = {}) {
  if (!url) {
    throw new Error('url is required')
  }

  const xhr = new XMLHttpRequest()

  const promise = new Promise((resolve, reject) => {
    try {
      xhr.addEventListener('load', () => {
        if (xhr.status >= 200 && xhr.status < 300) {
          resolve(xhr.response)
        } else {
          reject({
            code: xhr.status,
            codeText: xhr.statusText
          })
        }
      })

      xhr.addEventListener('error', reject)

      // 设置返回值格式
      xhr.responseType = 'json'

      // 设置超时时间
      xhr.timeout = timeout

      // 设置响应头
      function setRequestHeader() {
        if(Object.keys(header).length) {
          for (const [key, value] of Object.entries(header)) {
            xhr.setRequestHeader(key, value)
          }
        }
      }

      if (method.toUpperCase() === 'GET') {
        // 如果请求是get请求 同时设置了query参数，就和url中原本query参数进行合并
        const searchObj = Object.fromEntries(new URLSearchParams(new URL(url).search).entries())

        // URL实例的search参数只接收字符串类型值，非字符串类型值会转字符串
        requestUrl.search = new URLSearchParams(Object.assign(searchObj, data))

        xhr.open(method, requestUrl)
        setRequestHeader()
        xhr.send()
      } else {
        xhr.open(method, url)
        setRequestHeader()

        if (data instanceof FormData) {
          xhr.send(data)
        } else {
           xhr.setRequestHeader('Content-Type', isRequestJson ? 'application/json' : 'application/x-www-form-urlencoded')

           if (isRequestJson) {
            xhr.send(JSON.stringify(data))
           } else {
            xhr.send(new URLSearchParams(data).toString())
           }
        }

      }
    } catch (e) {
      reject(e)
    }
  })

  // 将创建出来的xhr对象挂载到返回的promise对象上
  // 以便于调用者可以拿到原生的XHR对象，并进行对应操作 「 如执行abort方法 」
  promise.xhr = xhr

  return promise
}
```



## 示例

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>XHR实现文件上传</title>
</head>
<body>
  <input type="file" id="file">
  <button id="btn">submit</button>

  <script>
    const fileEl = document.getElementById('file')
    const btnEl = document.getElementById('btn')

    btnEl.addEventListener('click', () => {
      const xhr = new XMLHttpRequest()

      xhr.open('post', 'http://www.example.com/upload')

      xhr.onload = () => console.log(xhr.response)

      // 监听上传进度
      xhr.onprogress = e => console.log(e.loaded / e.total)

      xhr.responseType = 'json'

      // 当一个input的属性为type的时候，其对应的dom元素就会多一个files属性
      // 其是一个伪数组，记录这所有上传的文件信息，每一个文件信息都是File对象的实例对象
      const files = fileEl.files

      // 目前前端文件上传都是依靠表单来进行完成和实现的
      // 也就是说将文件对象取出，作为表单对象的一个key-value后
      // 上传整个表单对象
      const formData = new FormData()
      formData.append('avatar', files[0])

      xhr.send(formData)
    })
  </script>
</body>
</html>
```

