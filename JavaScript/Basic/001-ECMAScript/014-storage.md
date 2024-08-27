`storage`是浏览器提供的存储机制，相比`cookie`而言，更直观，因为`storage`采用的是`key和value`的存储方式

| 存储方式                      | 说明                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| localStorage 「 本地存储 」   | 1. 持久化存储<br />2. 除非手动删除，否则一直存在             |
| sessionStorage 「 会话存储 」 | 1. 会话存储<br />2. 会话结束时，存储内容就会消失，类似于一种临时存储 |



**会话**

浏览器会话（Browser Session）通常指的是用户在浏览器中打开一个网页到关闭该页面或浏览器期间，与服务器进行的交互活动

一般情况下，只有关闭了页签才会关闭当前会话  

| 属性     | 说明                                                        |
| -------- | ----------------------------------------------------------- |
| `length` | 1. 一个只读number类型值<br />2. 返回存储在storage中的数据项 |

| 方法                  | 说明                                                         |
| --------------------- | ------------------------------------------------------------ |
| `key(n)`              | 1. typeof n -> number<br />2. 通过索引去取数据项<br />3. 有则返回，无则null |
| `getItem(key)`        | 1. typeof key -> string<br />2. 根据key返回数据项，有则返回，无则null |
| `setItem(key, value)` | 1. `key`和`value`需要时字符串，不是会自动转换<br />2. 有则替换，无则新增 |
| `removeItem(key)`     | 删除数据项，没有对应数据项则静默失效                         |
| `clear()`             | 清空storage                                                  |

```js
// storage的key需要在多个文件中被重复使用，所以抽取为常量
// 1. key和使用key的地方解耦，如果需要修改key，只要修改常量的值就可
// 2. 统一使用导出的常量，在使用时会存在提示，不易出错

// 这类常量 约定俗成下 我们一般将这类常量的常量名 全部大写
const ACCESS_TOKEN = 'access_token'

localStorage.setItem(ACCESS_TOKEN, <token值>)

console.log(localStorage.getItem(ACCESS_TOKEN));
```



## 简单封装

### 中间件

中间件是在网络请求传递给服务器之前或获取响应数据之后，对请求或响应数据进行拦截并执行自定义操作的函数或方法。

它可以用于修改请求或响应、添加信息、处理错误等，**通常以链式调用的方式工作**。

```js
// 这个类本质是对storage操作的拦截处理，可以看成是storage的"中间件"
class Cache {
  // 私有属性需要在类中显示声明后才可以使用
  #cache

  constructor(type = 'local') {
    this.#cache = type === 'local' ? localStorage : sessionStorage
  }

  setItem(key, value) {
    // JSON不存在bigInt，symbol和undefined，直接解析会报错
    if (!key || typeof value === undefined) {
      throw new TypeError('key 和 value 不能为空')
    }

    if (['object', 'function', 'symbol', 'bigint'].includes(typeof key)) {
      throw new TypeError('key只能是基本数据类型值')
    }

    this.#cache.setItem(key, JSON.stringify(value))
  }

  getItem(key) {
    if (['object', 'function'].includes(typeof key)) {
      throw new TypeError('key只能是基本数据类型值')
    }

    return JSON.parse(this.#cache.getItem(key))
  }

  removeItem(key) {
    if (['object', 'function'].includes(typeof key)) {
      throw new TypeError('key只能是基本数据类型值')
    }

    this.#cache.removeItem(key)
  }

  clear() {
    this.#cache.clear()
  }
}

// Test Code
const SESSION_TEST = 'session_test'
const LOCAL_TEST = 'local_test'

const sessionCache = new Cache('session')
sessionCache.setItem(SESSION_TEST, { name: 'Klaus', age: 23 })
console.log(sessionCache.getItem(SESSION_TEST))

const localCache = new Cache()
localCache.setItem(LOCAL_TEST, { name: 'Alex', age: 18 })
console.log(localCache.getItem(LOCAL_TEST))
```



## storage

当`Web Storage ` 「 如`localStorage`和`sessionStorage` 」的状态发生变化时会触发`storage`事件

```js
const el = document.getElementById('btn')

window.addEventListener('storage', e => {
  console.log('Key:', event.key);
  console.log('Old Value:', event.oldValue);
  console.log('New Value:', event.newValue);
  console.log('URL:', event.url);
})

el.addEventListener('click', () => {
  sessionStorage.setItem('name', 'Klaus')
})
```

