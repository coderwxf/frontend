## 概述

### 特点

1. 在不刷新页面的情况下，单独发送网络请求  => 用于替代ajax
2. 支持promise编程



### 缺点

1. 只能在浏览器使用，node中不能使用
2. 功能没有`ajax`丰富 => 没有超时时间、中断请求等功能



## 基本使用

```js
const response = fetch(请求地址, 配置项);
```

1. fetch的返回值response是一个Promise实例
2. 只要和服务器连接成功，`response`的状态就是`fulfilled`, 其余时候都是`rejected`
   + 只要和服务器连接成功即可，不看响应码「 即使返回的状态码是500或404 」 => 和AJAX最大的区别
   + 只有断网，请求超时或者请求中断等情况下，`response`的状态才会是`rejected`



### [配置项](https://developer.mozilla.org/zh-CN/docs/Web/API/Fetch_API/Using_Fetch)

#### method

1. 设置请求方式
2. 默认值是`get`
3. 可选值 => 支持如下请求方式的api 被称之为`RESTful API`
   + `get` => 获取数据
   + `post` => 插入数据 「 新增 」
   + `put` => 替换数据 「 整体替换 」
   + `patch` => 替换数据 「 部分替换 」
   + `delete` => 删除数据
   + `options` => 查看接口支持的请求方式「 预检请求 」
   + `head` =>  仅获取请求头 => 获取指定资源的元信息 「 响应体大小，是否可以走缓存而不用再发请求，... 」



##### 预检请求

在发生跨域请求时 => **CORS**（跨域资源共享，Cross-Origin Resource Sharing）

如果满足如下条件中的某一个就可以发送预检 ( Preflight ) 请求

1. 非简单请求
2. 存在自定义请求头



浏览器会在实际请求之前发生预检请求，来查看是否可以进行跨域请求

一般查看的`Access-Control`开头的响应头和`Allow` 响应头

+ 允许的请求方法（`Access-Control-Allow-Methods`）
+ 允许的请求头字段（`Access-Control-Allow-Headers`）
+ 允许的来源（`Access-Control-Allow-Origin`）
+  `Allow`响应头用于描述该接口所支持的请求方式



如果允许，则发生实际请求，否则请求中断，并在控制台输出跨域错误信息



##### 非简单请求

`非简单请求`是CORS中的概念

满足如下几个条件的是简单请求，其余都是非简单请求

1. 请求方法时`get | post | head`
2. 没有自定义请求头



### mode

用于配置是否可以发送跨域请求

+ `cors` => 可以发送跨域请求 => 默认值
+ `same-origin` => 只能发送同源请求 => 在发送请求之前就被浏览器阻止 「 跨域是返回结果不被浏览器处理 」
+ `no-cors` => 允许跨域，但获取数据会存在限制



### cache

用于配置是否可以使用缓存

+ `default` => 浏览器自行决定
+ `no-cache` => 不使用缓存



### credentials

在进行跨域请求时，浏览器是否允许发送 cookies 或 自定义请求头 

+ `same-origin` => 只允许同源接口携带凭据 「 默认值 」
+ `include` => 同源和跨域都可以携带凭据
+ `omit` => 同源和跨域都不可以携带凭据



#### 请求凭据

用于验证用户身份的内容， 可以是任意形式

1. 名为`token`的cookie => cookie会自动在发送请求时给服务器
2. 名为`Authorization`的请求头
3. 。。。



### headers

用于自定义请求头

+ 普通对象 => `Record<请求头名，请求头值>`
+ `Headers`实例



#### 命名规则

同时使用于请求头和响应头

1. 多个单词之间使用`-`进行拼接
2. 每个单词的首字母大写

例如`Content-Type`、`Access-Control-Allow-Origin`等



### referrerPolicy

用于单独配置请求头的`refer`请求头



### body

用于设置请求体

1. 只有需要请求体的请求方式「 例如`post`、`put`等 」才可以设置`body`配置项
2. 对于不支持请求体的请求方式「 例如`get`、`delete`等 」=> `fetch`方法返回的那个promise状态会直接变为`rejected`
3. `body`的值一般是字符串类型值 => 不能是普通对象，会被转换为`[object Object]`

```js
const response = fetch('https://httpbin.org/get', {
  body: {}
})

// response这个Promise实例的状态会被成rejected
```



可以设置的值 => 只要不是普通字符串文本，就需要通过`Content-Type`请求头设置`MIME`类型

| 类型                | `Content-Type`                      | 说明                                                         |
| ------------------- | ----------------------------------- | ------------------------------------------------------------ |
| 普通文本            | `text/plain`                        | 默认值<br />示例： `Hello World`                             |
| JSON格式字符串      | `application/json`                  | 示例： `{"name":"Klaus","age":23}`                           |
| urlencode格式字符串 | `application/x-www-form-urlencoded` | 示例：`name=Klaus&age=23`                                    |
| FormData实例        | `multipart/form-data`               | 用于上传表单或文件时使用<br />`Content-Type`会被自动设置「 不用自己设 」 |





## get请求

```js
```



























```md
1. redirect
signal
2. referrerPolicy
默认情况下它的reform理的情况，他就把当前客户端那个地址，然后通过ref传给谁，传给服务器，是不是他那个ref就是这么处理的，然后我们也不用调，对吧



第三类：其他方案，主要是跨域为主
+ jsonp
+ postMessage
+ 利用img的src发送请求，实现数据埋点和上报！！
+ ...


JSONP => script src 不受跨域限制 => 只能get
PV和UV
数据买点和上报呢
image src 数据上报 ？？
```



