![37234ad844fe67ad9ea2eb8e13bbdcc4.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/c51a7fefc356440883de526577f51e55~tplv-k3u1fbpfcp-zoom-1.image) 

JS通过寄生组合式实现继承

所以 如果一个构造函数的显示原型存在于当前实例的原型链上，那么就可以认为该构造函数是当前实例的父类



`实例 instanceOf 构造函数` --- 实例的原型链上 有没有右边的 构造函数

```js
function Person(name) {
  this.name = name
}

function Student(name, age) {
  Person.call(this, name)
  this.age = age
}

Student.prototype = Object.create(Person.prototype, {
  constructor: {
    value: Student,
    configurable: true,
    writable: true
  }
})

const stu = new Student('Tom', 22)

console.log(str instanceof Student) // => true
console.log(str instanceof Person) // => true
```

