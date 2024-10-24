`Headers`类用于操作请求头或响应头

```js
const headers = new Headers()

console.log(headers)
console.log(headers.toString()) // => [object Header] => Object.prototype.toStirng是检测数据类型最准确的方法 
```



打印`Headers`示例,  可以在其原型上查看其所支持的方法

| 方法        | 说明                                      |
| ----------- | ----------------------------------------- |
| **append**  | 设置头 「 多则追加 」                     |
| **set**     | 设置头 「 多则覆盖 」                     |
| **delete**  | 删除头                                    |
| **get**     | 获取头                                    |
| **has**     | 判断头是否存在                            |
| **forEach** | 迭代头                                    |
| **entries** | 获取用于迭代响应头entries格式数据的迭代器 |
| **keys**    | 获取用于迭代响应头key的迭代器             |
| **values**  | 获取用于迭代响应头value的迭代器           |

