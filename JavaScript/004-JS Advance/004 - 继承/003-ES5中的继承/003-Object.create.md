## 寄生

JavaScript中的寄生 表示的是 

1. `创建一个对象，该对象的隐式原型是一个自定义的对象`
2. 后续可以在改对象上执行额外自定义操作

简单来说 就是在当前原型链上加入一个自定义节点



`早期解决方法` 

```js
function createObj(obj) {
  // 使用一个新的空的构造函数来创建一个空的新对象
  function F() {}
  // 修改原型
  F.prototype = obj
  // 返回对象
  return new F()
}
```



`后续解决方法`

```js
function createObj(obj) {
  const newObj = {}

  // 将参数二 作为参数一的原型对象
  Object.setPrototypeOf(newObj, obj)

  return newObj
}
```



`最新解决方法`

```js
const newObj = Object.create(obj)
```



## Object.create

```js
const parent = {age: 23}

// Object.create方法会创建一个以第一个参数为原型的新对象 

// 参数一: 返回新对象的原型 可以是一个对象 也可以是null 
// 		为null时 表示该对象为顶层对象

// 参数二: 会添加到返回对象中的属性 
// 以对象形式进行描述 
// 		key 为 属性名 
// 		value 为 属性对应的属性描述符 

const child = Object.create(parent, { name: { value: 'Klaus', enumerable: true } })

console.log(child.name, child.age) // => Klaus 23
```

