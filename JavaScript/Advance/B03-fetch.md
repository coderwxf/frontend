1. 用于替代ajax
2. 只能用于浏览器，但在node中有node-fetch等第三方库可以实现类似功能
3. 原生支持promise



## 基本使用

```shell
# Promise<Response> fetch(input[, init])
```

+ input: 定义要获取的资源地址，是一个URL字符串
+ init: 其他初始化参数
  + method: 请求使用的方法，如 GET、POST
  + headers: 请求的头信息
  + body: 请求的 body 信息



```js
try {
    // fetch 方法返回一个 promise
    // 执行 await 表示等待和服务器连接成功，开始下载资源
    const response = await fetch('http://www.httpbin.org/get?name=Klaus&age=23')

    // 连接已建立，可以获取响应状态和响应状态码
    // 当状态码为 200-299, response.ok 为 true，否则即为 false
    console.log(response.status, response.statusText, response.ok) // 200 'OK' true

    // 加载完成，开始解析数据
    // response.json() --- 数据解析为 json
    // response.text() --- 数据解析为普通文本
    const data = await response.json()
    console.log(data)
} catch (error) {
    console.error('Fetch error:', error)
}
```

```js
try {
  // fetch(api地址, 配置对象)
  const response = await fetch('http://www.httpbin.org/post', {
    method: 'post', // 请求方法
    body: JSON.stringify({
      name: 'Klaus',
      age: 23
    }),
    // 请求头
    headers: {
      // 当上传的请求体为表单数据时，Content-type会被自动设置为multipart/form-data
      // 其余情况下都会识别为text/plain，如果要修改需要显示手动设置
      'Content-type': 'application/json'
    }
  })

  const res = await response.json()

  console.log(res)
} catch (e) {
  console.error(e.message)
}
```



