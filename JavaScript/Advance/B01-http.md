## 渲染模式

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7ad772d271bc43168c2bcdf14b7be96c~tplv-k3u1fbpfcp-zoom-1.image) 

1. 最早是服务器端渲染(SSR，server side render 或 后端渲染)
2. 使用模板引擎或asp/jsp等技术 将数据插入到html内 渲染形成完整的html

缺点:

1. 功能都由后端实现，项目越大，业务逻辑和界面渲染以及数据操作之间的耦合度会越来越高
2. 无法局部刷新，只能全局刷新



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/514a2a060af4405ebff705f6f64b41fd~tplv-k3u1fbpfcp-zoom-1.image)

1. ajax/fetch等拿到数据操作DOM更新界面 「 局部刷新 」
2. 服务器请求的资源都是静态资源 「 界面渲染 和 数据分开 」

这种渲染模式也被称之为前端渲染 (或被称之为前后端分离模式 client side render CSR 或 客户端渲染)

1. `不利于SEO`, 浏览器爬虫爬取的一般都是HTML文件(google爬虫除外)，并不会爬取js代码
2. 所有的渲染过程都是在前端完成，所需要的渲染时间较长，所以`不利于首屏渲染`



**同构渲染 (Isomorphic Rendering) **

- 结合了 SSR 和 CSR 的优点，初始页面由服务器渲染，后续交互由客户端渲染。
- 优点：更快的首屏渲染和更好的 SEO，同时也能实现局部刷新。



**静态站点生成 (Static Site Generation, SSG)：**

- 在构建时生成静态 HTML 文件，部署时直接提供这些静态文件。
- 优点：更快的页面加载速度和更好的 SEO，适用于内容不频繁变化的网站。



## HTTP

超文本传输协议(英语:HyperText Transfer Protocol，缩写:HTTP) 是一个客户端(用户)和服务端(网站)之间请求和响应的标准,用于客户端和服务器之间进行数据通信和数据交换

+ 用户代理程序(user agent) 「 如浏览器或爬虫工具 」
+ HTTP最初的目的是为了提供一种发布和接收HTML页面的方法

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d9ead3b906954ff18bc4feb8125fa7cc~tplv-k3u1fbpfcp-zoom-1.image) 



`请求`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/23d6f25f4c8e4a25a121626d951851d0~tplv-k3u1fbpfcp-zoom-1.image)



`响应`

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f13453c4e5cd43cebd61dde4462df04c~tplv-k3u1fbpfcp-zoom-1.image)



## RESTful API

| 方式名  | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| GET     | 获取资源                                                     |
| HEAD    | 只要响应头<br /> 在未获取实际资源的情况下，对资源的首部进行检查。<br /> 比如在准备下载一个文件前，先获取文件的大小，再决定是否进行下载。 |
| OPTIONS | 用于查询服务器支持的 HTTP 方法或检查服务器的功能             |
| POST    | 提交新数据                                                   |
| PUT     | 对数据进行整体更新                                           |
| DELETE  | 删除数据                                                     |
| PATCH   | 对数据进行部分更新                                           |
| CONNECT | 将 HTTP 的连接修改为管道方式进行连接，<br />建立一个到目标资源标识的服务器的隧道，<br />通常用在代理服务器，网页开发很少用到 |
| TRACE   | 沿着到目标资源的路径执行一个消息环回测试<br />用于请求和响应测试，帮助调试 Web 服务器或代理服务器的路径 |



## 请求头

请求头的key一般以首字母大写的，以中划线进行拼接的多个单词组成

| key                    | value                                                        |
| ---------------------- | ------------------------------------------------------------ |
| Content-Type           | 这次请求携带的数据的类型                                     |
| Content-Length         | 请求体的大小长度。客户端会自动进行计算并设置                 |
| Connection: keep-alive | 客户端和服务器之间请求是持久连接<br />尽可能复用同一个连接，而不是断开重连<br />不同的Web服务器会有不同的保持 keep-alive的时间<br />Node中默认是5s |
| Accept-Encoding        | 告知服务器，客户端支持的文件压缩格式                         |
| Accept                 | 告知服务器，客户端可接受文件的格式类型                       |
| User-Agent             | 客户端相关的信息                                             |



### Accept-Encoding

告知服务器，客户端支持的文件压缩格式。

比如，JavaScript 文件可以使用 gzip 编码，对应 .gz 文件（如 index.js -> index.js.gz）。

在请求时，客户端会自动配置自己可以接收的压缩文件格式并传输给服务器。

如果服务器有对应资源的压缩格式文件时，会自动将对应的压缩格式文件传输给客户端。

客户端在获取到对应的压缩文件后，会自动进行解压并解析。

常见的网络传输的压缩格式为 gzip。



### content-type

| 类型                              | 说明                                             |
| --------------------------------- | ------------------------------------------------ |
| application/x-www-form-urlencoded | 以URL编码的形式传递数据<br />如name=Klaus&age=23 |
| application/json                  | 表示是一个json类型                               |
| text/plain                        | 表示是文本类型                                   |
| application/xml                   | 表示是xml类型                                    |
| multipart/form-data               | 表示是提交表单                                   |



## 响应头

| 头                          | 说明                   |
| --------------------------- | ---------------------- |
| Access-Control-Allow-Origin | 是否允许跨域           |
| Content-Length              | 返回的响应体的长度     |
| Content-Type                | 返回的响应体的格式类型 |
| Date                        | 返回时间               |



### 响应码

[Http状态码(Http Status Code)](https://developer.mozilla.org/zh-CN/docs/web/http/status)是用来表示Http响应状态的数字代码

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bbf9270cb061469abfbfb87b57906bbc~tplv-k3u1fbpfcp-zoom-1.image) 

一般情况下:

1. 2xxx - 成功状态码
2. 3xxx - 重定向状态码
3. 4xxx - 客户端错误
4. 5xxx - 服务器端错误