## location

location用于操作URL

![image.png](https://s2.loli.net/2024/08/14/UuHESnXJ9MrxYjL.png) 

> 在URL中，路径部分（pathname）通常区分大小写，而其余部分通常不区分大小写

| 属性     | 说明                                     | 示例                                                   |
| -------- | ---------------------------------------- | ------------------------------------------------------ |
| href     | 当前window对应的超链接URL, 整个URL       | http://127.0.0.1:5501/index.html?name=klaus&age=23#foo |
| protocol | 当前的协议<br />带最后的 : 后缀          | http:                                                  |
| host     | 主机地址<br />host = hostname + port     | 127.0.0.1:5501                                         |
| hostname | 主机地址(不带端口)                       | 127.0.0.1                                              |
| port     | 端口<br />不带 ：前缀                    | 5501                                                   |
| origin   | 源地址<br />origin属性是唯一一个只读属性 | http://127.0.0.1:5501                                  |
| pathname | 路径<br />带 / 前缀                      | /demo/index.html                                       |
| search   | 查询字符串<br />带 ？前缀                | ?name=klaus&age=23                                     |
| hash     | 哈希值(又名片段值)<br />带 # 前缀        | #hash                                                  |

| 方法    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| assign  | 当前页打开新页面<br />添加历史记录<br />直接赋值location.href的效果是一样的 |
| replace | 当前页打开新页面<br />不添加历史记录                         |
| reload  | 重新加载页面<br />如果参数传入true，则表示强制刷新           |



## URL

URL的本质就是有特定规则的字符串，直接操作字符串容易出错且不方便

所以JS提供了`URL`和`URLSearchParams`来帮助我们操作URL

```js
const url = new URL(pathname, base)
```

1. URL实例的toString方法本质上调用URL实例的href属性。
2. 如果pathname和base不是字符串，会自动转换为字符串。
3. 如果pathname是相对路径（不包含协议和主机部分），会使用base进行拼接
4. 如果pathname是完整路径（以协议开头），则忽略base，直接使用pathname。
5. 如果pathname是完整路径，则base可以省略，否则base是必传的，且必须是完整路径

```js
const base = 'https://www.example.com'
console.log(new URL('foo', base).href) // => https://www.example.com/foo
console.log(new URL('foo', base).toString()) // => https://www.example.com/foo
```



`示例`

```js
// URL在生成URL实例时，会自动将url和base进行拼接
let base = 'https://www.example.com/study/a/b/c'

console.log(new URL('sp/icon', base).href) // => https://www.example.com/study/a/b/sp/icon

console.log(new URL('./sp/icon', base).href) // => https://www.example.com/study/a/b/sp/icon

console.log(new URL('../sp/icon', base).href) // => https://www.example.com/study/a/sp/icon

// url链接地址以/结尾，所以上一层目录为study/a/b/c
console.log(new URL('../sp/icon', base + '/').href) // => https://www.example.com/study/a/b/sp/icon

// 如果第一个参数以/开头，那么就表示是绝对路径，其会替换原本的pathname
console.log(new URL('/sp/icon', base).href) // => https://www.example.com/sp/icon

// 没有协议的url认为是相对地址，协议取自base地址
// 生成的url地址如果没有pathname的时候，会自动以/结尾
console.log(new URL('//image.example.com', 'https://www.example.com').href) // => https://image.example.com/

// url为绝对地址，base参数被忽略
console.log(new URL('https://image.example.com', base).href); // => https://image.example.com/
```



**属性**

URL实例所具有的属性和location完全一致, 但多个了`searchParams`

| 属性         | 说明                                     | 示例                                                   |
| ------------ | ---------------------------------------- | ------------------------------------------------------ |
| href         | 当前window对应的超链接URL, 整个URL       | http://127.0.0.1:5501/index.html?name=klaus&age=23#foo |
| protocol     | 当前的协议<br />带最后的 : 后缀          | http:                                                  |
| host         | 主机地址<br />host = hostname + port     | 127.0.0.1:5501                                         |
| hostname     | 主机地址(不带端口)                       | 127.0.0.1                                              |
| port         | 端口<br />不带 ：前缀                    | 5501                                                   |
| origin       | 源地址<br />origin属性是唯一一个只读属性 | http://127.0.0.1:5501                                  |
| pathname     | 路径<br />带 / 前缀                      | /demo/index.html                                       |
| search       | 查询字符串<br />带 ？前缀                | ?name=klaus&age=23                                     |
| hash         | 哈希值(又名片段值)<br />带 # 前缀        | #hash                                                  |
| searchParams | search属性对应的URLSearchParams实例对象  |                                                        |



**实例方法**

URL有两个实例方法 `toString()`和`toJSON()`

他们的功能是一致的，都是直接返回URL实例对象的href属性



**静态方法**

| 方法名          | 功能                                                         |
| --------------- | ------------------------------------------------------------ |
| createObjectURL | 可以把二进制对象「 如File，Blob对象 」变成一个一个唯一的blob URL |
| revokeObjectURL | 撤消之前使用`URL.createObjectURL()`创建的URL对象             |

Blob URL 是一种特殊的临时URL，以 `blob:` 开头。

Bob URL一般用于引用由JavaScript创建的`Blob`对象，存在于网页的内存中。

Blob URL 是临时的，它们的生命周期是与页面或浏览器进程相关联的。

页面关闭、刷新或显式调用 `URL.revokeObjectURL()` 方法时，Blob URL 会失效，所引用的内存资源会被释放。



## URLSearchParams

`URLSearchParams` 是一个用于操作 URL 中查询参数的强大工具。

可以通过 `URLSearchParams` 构造函数创建查询参数对象实例，

参数可以是查询字符串、二维数组，或者是一个对象。

```js
// 字符串头部的？是可以省略的
const queryStr = '?name=Klaus&age=25'
const queryArr = [['name', 'klaus'], ['age', 25]]
const queryObj = {
  name: 'Klaus',
  age: 25
}

const searchParams1 = new URLSearchParams(queryStr)
const searchParams2 = new URLSearchParams(queryArr)
const searchParams3 = new URLSearchParams(queryObj)
console.log(searchParams1, searchParams2, searchParams3)
```



| 方法     | 说明                                                         |
| -------- | ------------------------------------------------------------ |
| append   | 追加新的键值对作为查询参数<br />如果存在重复的键，会新增值而不会覆盖 |
| delete   | 删除已存在的查询参数                                         |
| get      | 返回指定关键字对象的值<br />多个返回第一个，不存在，返回null |
| getAll   | 以数组形式返回所有当前查询关键字对应的值<br />如果不存在返回空数组 |
| has      | 返回布尔值，`true`或者`false`，是否包含某个查询关键字        |
| entries  | 返回查询参数们的迭代器对象，我们可以使用for-of 迭代该迭代器对象获得所有的键值对 |
| keys     | 返回一个迭代器对象，允许迭代该对象中所有的关键字             |
| values   | 返回一个迭代器对象，允许迭代该对象中所有的关键字值           |
| forEach  | 此方法可以遍历查询对象                                       |
| set      | 新增属性和属性值，有则覆盖，无则添加                         |
| sort     | 按照key的Unicode码顺序进行升序排序                           |
| toString | 将URLSearchParams实例对象转换为对象的查询字符串              |



直接迭代`searchParams`，会自动调用`searchParams.entries`

```js
const searchStr = '?name=Klaus&age=23'
const searchParams = new URLSearchParams(searchStr)

// 类似于在迭代[[name, Klaus], [age, 23]]
for (const [key, value] of searchParams) {
  console.log(key, value)
}

// 转换为字符串格式 --- 转换后的结果不带？前缀
console.log(searchParams.toString()) // => name=Klaus&age=23
```



**URLSearchParams实例的entries形式和对象形式之间的相互转换**

```js
const searchStr = 'name=Klaus&age=23'
const searchParams = new URLSearchParams(searchStr)

// URLSearchParams实例 -> entries
const entries = searchParams.entries()
console.log(entries)
// => URLSearchParams Iterator { [ 'name', 'Klaus' ], [ 'age', '23' ] }

// entries -> object
const obj = Object.fromEntries(entries)
console.log(obj) // => { name: 'Klaus', age: '23' }

// object -> URLSearchParams实例
console.log(new URLSearchParams(obj))
// => URLSearchParams { 'name' => 'Klaus', 'age' => '23' }
```



## URL编码

在URL中，只有字母（A-Z，a-z）、数字（0-9）以及一些特定的特殊符号（如：`/`、`?`、`#`、`&`、`=`、`.`、`-`、`_`、`~` 等）可以直接使用。「 例如 `&name=klaus_wang&user=mary-jane` 」

对于其他字符，特别是非ASCII字符（如中文、空格和一些特殊符号），必须经过URL编码（也称为百分号编码）后才能在URL中使用。

这意味着这些字符会被转换为特定的编码格式，例如空格会被编码为 `%20`，以确保URL的正确性和兼容性。



URL编码有两对编解码方式

`encodeURIComponent / decodeURIComponent` 

+ `encodeURIComponent`编码，必须通过`decodeURIComponent`解码

+ 用于编解码URL中的查询字符串(?开头的部分) 和 hash部分(#开头的部分)

`encodeURI / decodeURI` 

+ `encodeURI`编码，必须通过`decodeURI`解码
+ 用于对整个URL进行编解码操作



## URL、URN、URI

URI (Uniform Resource Identifier， 统一资源标识符)，给互联网中的资源起一个名字

URL(Uniform Resource Locator，统一资源定位符)，资源的名称加上路径，告诉你如何在网络中定位和访问该资源

URN(Uniform Resource Name，统一资源名称)，一个独立的、持久的、唯一标识某个资源的名称



`URI = URL + URN`

例如:

`https://example.com` 是一个 URL，也是一个 URI。

`urn:isbn:0451450523` 是一个 URN，也是一个 URI。

![img](https://s2.loli.net/2024/08/15/kyT8mrAnVltOadb.png) 



## 域名划分

**一级域名（Top-Level Domain, TLD）** 是域名的最右侧部分，通常用于表示域名的类别（例如 `.com`、`.org`），或特定国家/地区的归属（例如 `.cn` 表示中国，`.uk` 表示英国）。

**二级域名（Second-Level Domain, SLD）** 紧邻一级域名的左侧，通常代表域名的主体部分，如公司名称、品牌或组织。例如在 `example.com` 中，`example` 就是二级域名。

**三级域名及更高层级的子域名** 位于二级域名的左侧，由域名持有者自行划分，用于进一步组织和管理其网络资源。三级域名有一些约定俗成的三级域名，例如 `www`（用于表示网站的主页）和 `mail`（用于邮件服务器），但也可以根据特定需求进行自定义。

#### 示例：

- `https://www.google.com`
  - **一级域名（TLD）**：`.com`
  - **二级域名（SLD）**：`google`
  - **三级域名（子域名）**：`www`
- `https://docs.microsoft.com`
  - **一级域名（TLD）**：`.com`
  - **二级域名（SLD）**：`microsoft`
  - **三级域名（子域名）**：`docs`