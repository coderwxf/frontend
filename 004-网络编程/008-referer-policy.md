## referer

HTTP 请求的头信息里面，`Referer` 是一个常见字段，提供访问来源的URL

`Referer`的正确拼写是`Referrer`，但是写入标准的时候，不知为何，没人发现少了一个字母`r`

标准定案以后，只能将错就错，所有头信息的该字段都一律错误拼写成`Referer`



在页面引入图片、JS 等静态资源，或者从一个页面跳到另一个页面，都会产生新的 HTTP 请求

浏览器一般都会给这些请求头加上表示来源的 Referrer 字段



用户在地址栏输入网址，或者选中浏览器书签，会被认为是直接向对应的服务器请求对应的资源， 所以在请求头中就不包含对应的`Referer`字段

同样，如果在请求头中不存在对应的`Referrer`字段，会被认为是直接向服务器请求对应资源 ，因此可以使用这个方式获取一些通过refer进行防盗链检测的资源



Referrer 在分析用户来源时很有用，有着广泛的使用，但 URL 可能包含用户敏感信息，如果被第三方网站拿到很不安全

例如内网 URL，不希望外部用户知道内网有这样的地址 或者 一些页面的URL可能会携带一些敏感信息，如UID，订单编号等 ，这种情况，网站肯定不希望泄露用户的身份凭证信息

某些情况下，某些网站在客户端请求请求自己资源的时候，会对refer进行判断，以确保是白名单用户来请求和访问对应的资源，常见的有防盗链

为此，W3C 的 Web 应用安全工作组（Web Application Security Working Group）发布了 [Referrer Policy](http://w3c.github.io/webappsec/specs/referrer-policy/) 草案，对浏览器该如何发送 Referrer 做了详细的规定。



## Referrer Policy

通过设置[referrer-policy](https://caniuse.com/?search=referrer-policy)的不同属性值，我们可以精准控制我们提交什么形式的源地址信息给外链或其它网页

| 值                              | 说明                                                         |
| ------------------------------- | ------------------------------------------------------------ |
| no-referrer                     | 不提供referrer的任何信息在请求头中                           |
| no-referrer-when-downgrade      | 当网站安全降级时(即HTTPS→HTTP），不提供referrer的信息<br />其余情况显示完整的referrer信息<br />默认值，referrer属性值为空字符串的时候，其就代表`no-referrer-when-downgrade` |
| same-origin                     | 只给同源网站提供完整referrer信息                             |
| origin                          | 只给同源网站提供网站的源地址信息（即协议、域名、端口）       |
| strict-origin                   | 只给同源网站提供网站的源地址信息（即协议、域名、端口）<br />但当网站安全降级的时候，不提供referrer信息 |
| origin-when-cross-origin        | 同源网站，提供完整的referrer信息<br />跨域网站，提供源地址信息 |
| strict-origin-when-cross-origin | 同源网站，提供完整的referrer信息<br />跨域网站，提供源地址信息<br />当网站安全降级的时候，不提供referrer信息 |
| unsafe-url                      | 无论任何时候，都提供完整的referrer信息<br />不推荐           |



### 设置方式

#### 1. 设置请求头Referrer-Policy

```shell
Referrer-Policy: no-referrer
```



#### 2. 设置meta标签

```html
<meta name="referrer" content="no-referrer">
```



#### 3. 为静态资源链接元素添加referrerpolicy属性

```html
<img
   src="https://tenfei01.cfp.cn/creative/vcg/800/new/VCG41N1227618770.jpg"
   alt="avatar"
   referrerpolicy="no-referrer"
 >
```

