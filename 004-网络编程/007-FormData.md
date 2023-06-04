FormData可以用来在传输数据的时候模拟表单数据的提交

在使用Fetch/XHR发生FormData的时候，`Content-type`会自动被调整为`multipart/form-data`



## 基本使用

`使用方式一`

```js
const btnEl = document.getElementById('btn')

btnEl.addEventListener('click', () => {
  const formEl = document.getElementById('form')

  // 如果在FormData构造器中提供了form元素对应的DOM元素，它会自动捕获对应form中的元素字段
  const formData = new FormData(formEl)
  console.log(formData.get('username'), formData.get('password'))
})
```



`使用方式二`

```js
// 创建一个空的FormData对象，然后调用它的append()方法来添加字段并使用
const formData = new FormData()

formData.append('name', 'Klaus')
formData.append('age', 23)

console.log(formData)
```



`formData` 里面存储的数据形式，一对 `key/value` 组成一条数据， `key是唯一的，一个key可能对应多个value`

结构类似于

| key  | value            |
| ---- | ---------------- |
| k1   | [v1, v2, v3, v4] |
| k2   | v5               |



## 实例方法

| 方法               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| append(key, value) | 添加数据<br />无则添加，有则追加                             |
| set(key, value)    | 修改数据<br />无则添加，有则覆盖                             |
| get(key)           | 根据key去查找key对应的value<br />get方法只能获取第一个满足条件的值<br />key对应的value不存在的时候，默认返回null |
| getAll(key)        | 根据key去查找key对应的value<br />getAll方法会获取key对应的所有的value值组成的数组<br />key对应的value不存在的时候，默认返回空数组 |
| has(key)           | 判断key对应的value是否存在，返回值是boolean类型值            |
| delete(key)        | 移除key对应的所有value值                                     |
| keys()             | 返回所有key值组成的迭代器对象，可以使用for-of遍历            |
| values()           | 返回所有value值组成的迭代器对象，可以使用for-of遍历          |
| entries()          | 返回所有键值对组成的迭代器对象，可以使用for-of遍历           |
| forEach()          | formData可以使用forEach进行遍历                              |
| for-of             | 实现了 FormData 接口的对象可以直接在for...of结构中使用，而不需要调用entries()<br />for (var p of myFormData) 的作用和 for (var p of myFormData.entries()) 是相同的 |

```js
const formData = new FormData()

formData.append('name', 'Klaus')
formData.append('name', 'Alex')

console.log(formData.get('name')) // => Klaus
console.log(formData.getAll('name')) // => [ 'Klaus', 'Alex' ]
```

```js
const formData = new FormData()
formData.append('name', 'Klaus')
formData.append('age', 23)

for (const [key, value] of formData.entries()) {
  console.log(key, value)
}
```



在FormData中添加的值的数据类型应为 `string | Blob` 中的一种

如果添加的数据类型不是`string | Blob`的时候，FormData会先尽可能将对应的值转换为字符串类型后在进行添加

```js
const btnEl = document.getElementById('btn')

btnEl.addEventListener('click', () => {
  const inputEl = document.querySelector('input')

  const formData = new FormData()

  // 如果input的type值为file的时候
  // 该input对应的dom元素就会存在属性files
  // 其值是对应的上传文件对应的File对象构成的伪数组对象
  formData.append('file', inputEl.files[0])

  console.log(formData.get('file'))
})
```

```js
const formData = new FormData()
formData.append('user', { name: 'Klaus', age: 23 })
console.log(formData.getAll('user')) // => [ '[object Object]' ]
```



当formData中添加的是二进制文件的时候，会尽可能取上传的文件对应的文件名作为传递给服务器时对应文件的文件名，如果无法获取，会存在对应的默认文件名

也可以通过`set`或`append`方法的第三个参数 来修改传递给服务器的时候，对应的文件名

```js
const btnEl = document.getElementById('btn')

btnEl.addEventListener('click', () => {
  const inputEl = document.querySelector('input')

  const formData = new FormData()

  // 将自定义上传的文件 命名为 foo.txt
  formData.append('file', inputEl.files[0], 'foo.txt')

  console.log(formData.get('file'))
})
```

