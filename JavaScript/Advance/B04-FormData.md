FormData可以用来在传输数据的时候模拟表单数据的提交

```js
const formEl = document.forms[0]

// 如果在FormData构造器中提供了form元素对应的DOM元素，它会自动捕获对应form中的元素字段
const formData = new FormData(formEl)
console.log(formData.get('username'), formData.get('password'))
```

```js
// 创建一个空的FormData对象，然后调用它的append()方法来添加字段并使用
const formData = new FormData()

formData.append('name', 'Klaus')
formData.append('age', 23)

console.log(formData)
```



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
| for-of             | 迭代FormData实例, 会自动调用FormData实例的entries方法        |



在FormData中添加的值的数据类型应为 `string | Blob` 中的一种，否则会尽可能先转字符串



当你添加二进制文件时，`FormData` 会尝试使用文件的原始文件名，如果读取失败则使用默认文件名

如果需要，你也可以手动指定文件名。

```js
// 将自定义上传的文件 命名为 foo.txt
formData.append('file', inputEl.files[0], 'foo.txt')
```

