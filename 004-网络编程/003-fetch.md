Fetch可以看做是早期的XMLHttpRequest的替代方案，它提供了一种更加现代的处理方案

+ 比如返回值是一个Promise，提供了一种更加优雅的处理结果方式
+ 比如不像XMLHttpRequest一样，所有的操作都在一个对象上
+ fetch可以同时在浏览器中使用，在node中可以借助node-fetch实现相同的功能，但是XMLHTTPRequest只能在浏览器中使用，node中不支持

## 基本使用

```shell
# Promise<Response> fetch(input[, init])
```

+ input: 定义要获取的资源地址，是一个URL字符串
+ init: 其他初始化参数
  + method: 请求使用的方法，如 GET、POST
  + headers: 请求的头信息
  + body: 请求的 body 信息

Fetch的数据响应主要分为两个阶段:

1. 当服务器返回了响应(response)

   + fetch 返回的 promise 就使用内建的 Response class 对象来对响应头进行解析
   + 在这个阶段，我们可以通过检查响应头，来检查 HTTP 状态以确定请求是否成功
   + 如果 fetch 无法建立一个 HTTP 请求，例如网络问题，亦或是请求的网址不存在，那么 promise 就会 reject
   + 异常的 HTTP 状态，例如 404 或 500，不会导致出现 error
   + 所以我们在第一阶段就可以获取到对应的状态码和状态描述信息

2. 为了获取 response body，我们需要使用一个其他的方法调用

   + response.text() —— 读取 response，并以文本形式返回 response
   + response.json() —— 将 response 解析为 JSON

```js
async function fetchData() {
 try {
   // fetch方法返回的是一个响应流
   const response = await fetch('http://www.httpbin.org/get?name=Klaus&age=23')

   // 我们可以调用response.json方法 或 response.text方法获取到对应的数据
   // response.json和response.text方法返回的是promise
   // 他们会等到响应流数据加载完毕的时候，才会调用对应的resolve方法
   const res = await response.json()

   console.log(res)
 } catch (e) {
   console.error(e.message)
 }
}

fetchData()
```

```js
async function fetchData() {
  try {
    // fetch方法参数一 为请求的URL地址
    // fetch方法参数二 请求配置对象
    const response = await fetch('http://www.httpbin.org/post', {
      method: 'post', // 请求方法
      // 请求体 - 如果是JSON格式数据 必须将其转换为json格式字符串
      // 因为json字符串在不同语言中解析出的对象都是不一致的
      // 所以在网络中传输json采用的一定且只能是json格式对应的字符串
      body: JSON.stringify({
        name: 'Klaus',
        age: 23
      }),
      // 请求头
      headers: {
        'Content-type': 'application/json'
      }
    })

    const res = await response.json()

    console.log(res)
  } catch (e) {
    console.error(e.message)
  }
}

fetchData()
```

## 响应状态和响应码

```js
async function fetchData() {
 try {
   const response = await fetch('http://www.example.com/postjson', {
      method: 'post',
      body: JSON.stringify({
        name: 'Klaus',
        age: 23
      }),
      headers: {
        'Content-type': 'application/json'
      }
   })

   /*
    1. response.status - 状态码
    2. response.statusText - 状态码描述信息
    3. response.ok - 一个boolean类型值
                   - 如果 HTTP 状态码为 200-299，则为 true
                   - 其余情况下 值为 false
   */
   console.log(response.status, response.statusText, response.ok)

   const res = await response.json()

   console.log(res)
 } catch (e) {
   console.error(e.message)
 }
}

fetchData()
```

## 示例 文件上传

fetch和XHR实现文件上传有一个比较大的区别是：

虽然fetch在实际使用的时候，比XHR要简约和方便很多

但是fetch并没有提供类似于XHR的progress方法，所以使用fetch进行文件上传的时候，无法对上传进度进行监听

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>使用fetch实现文件上传</title>
</head>
<body>
  <input type="file" id="file">
  <button id="btn">submit</button>

  <script>
    const fileEl = document.getElementById('file')
    const btnEl = document.getElementById('btn')

    btnEl.addEventListener('click', async() => {
      const files = fileEl.files

      const formData = new FormData()
      formData.append('avatar', files[0])

      const response = await fetch('http://www.example.com/upload', {
        method: 'post',
        body: formData // 当上传的请求体为表单数据时，Content-type会被自动设置为multipart/form-data
      })

      const res = await response.json()
      console.log(res)
    })
  </script>
</body>
</html>
```