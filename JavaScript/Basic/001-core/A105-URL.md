



```js
const searchStr = '?name=klaus&age=23'

const searchParams = new URLSearchParams(searchStr)

console.log(searchParams) // => URLSearchParams { 'name' => 'klaus', 'age' => '23' }

// 调用entries方法可以将 URLSearchParams实例对象 转换为 一个entries结构的可迭代对象
console.log(searchParams.entries()) // => URLSearchParams Iterator { [ 'name', 'klaus' ], [ 'age', '23' ] }

// entries是可迭代的， 所以可以使用for-of方法进行遍历
for (const [key, value] of searchParams.entries()) {
  console.log(key, value)
  /*
    =>
      name klaus
      age 23
  */
}

// 事实上，直接对URLSearchParams实例进行for-of遍历的时候，其会自动调用实例的entries方法
for (const [key, value] of searchParams) {
  console.log(key, value)
  /*
    =>
      name klaus
      age 23
  */
}

// 同样，既然URLSearchParams.prototype.entries方法返回的是一个entries
// 我们就可以通过Object.fromEntries方法将entries转换为普通对象
console.log(Object.fromEntries(searchParams.entries()))
// => { name: 'klaus', age: '23' }
```

