Set 是特殊的数组，WeakSet是特殊的set

Map是特殊的对象，WeakMap是特殊的Map



## 引用类型

1. 默认情况下，所有的引用都是强引用 「 会被GC统计在内 」
2. ES6新增加了一个引用方式，弱引用 「 不会被GC统计在内 」



弱引用不被GC统计，所以弱引用指向的对象可能并不一定存在 「 无法确保对应对象没有被GC回收 」

需要通过`deref`方法进行判断，`deref`方法会返回弱引用指向的对象，如果存在就返回，不存在，返回`undefined`

```js
const user = { name: 'Klaus' }

// 通过WeakRef创建弱引用
const weak = new WeakRef(user) 

if (weak.deref()) {
  // 直接通过weak.deref()调用属性，才能避免创建强引用
  console.log(weak.deref().name) // => Klaus
}

```

通过`weak.deref()`使用，我们期望的是弱引用，所以不要使用如下方式使用

1. `let ref = weak.deref()` --- 形成了强引用

2. 在控制台中直接打印 `weakRef.deref()` 时，它返回的对象是一个强引用

   从而确保打印对象不会被GC回收，从而可以供调试者进行调试和数据查看



## Set

1. key不能重复的数组 「 可以实现数组去重 」
2. 只能通过Set构造函数创建，没有字面量方式
3. set没有任何可迭代非symbol属性 「 可以使用for-in迭代，但没有任何意义 」
4. set是可迭代对象

```js
// Set的参数是任意数量的可迭代对象 『 0个或多个 』
const set = new Set([1, 2, 3, 4, 1])

// set -> array
console.log(Array.from(set)) // => [ 1, 2, 3, 4 ]
console.log([...set]) // => [ 1, 2, 3, 4 ]
```



### 属性和方法

| 属性 | 说明                |
| ---- | ------------------- |
| size | 返回set数据项的个数 |



| 方法                           | 说明                           |
| ------------------------------ | ------------------------------ |
| add(value)                     | 添加元素，返回set本身          |
| delete(value)                  | 删除元素，并返回布尔值         |
| has(value)                     | 判断元素是否存在，并返回布尔值 |
| clear()                        | 清空，没有返回值               |
| forEach(callback, [, thisArg]) | set支持forEach迭代             |
| for-of                         | set是可抵达对象                |

> 在JavaScript中，并不是所有可迭代对象都支持 `forEach`，因为 `forEach` 方法仅存在于特定类型的可迭代对象上，例如数组、Map和Set等

```js
const set = new Set([4, 3, 2, 1])

// set中，key和value的值是一致的
set.forEach((item, index) => console.log(item, index))
/*
  =>
   4 4
	 3 3
	 2 2
	 1 1
*/
```



### weakset

1. 特殊的set
2. 数据项必须是对象，且会形成该对应的弱引用
3. 弱引用无法确保每次迭代结果都是一致的，因此无法打印内部weakset或对weakset进行迭代

```js
const weakset = new WeakSet()

weakset.add({name: 'Klaus'})
console.log(weakset) // => WeakSet { <items unknown> }

// 参数不是对象，直接报错，不进行类型转换
weakset.add(123) // error
```



| 方法          | 说明                           |
| ------------- | ------------------------------ |
| add(value)    | 添加元素，返回WeakSet本身      |
| delete(value) | 删除元素，并返回布尔值         |
| has(value)    | 判断元素是否存在，并返回布尔值 |



`应用场景`

`避免重复操作`

```js
const visitedNodes = new WeakSet();

function visitNode(node) {
  if (!visitedNodes.has(node)) {
    visitedNodes.add(node);
    // 执行一些操作
  }
}

let node1 = { id: 1 };
let node2 = { id: 2 };

visitNode(node1);
visitNode(node2);
visitNode(node1); // node1 已经被访问过，不会再次执行操作

// 如果 node1 和 node2 被垃圾回收了，visitedNodes 中的引用也会被自动清除
```



`限制实例调用方式`

```js
class Person {
  static #weakset = new WeakSet()

  constructor() {
    Person.#weakset.add(this)
  }

  // 确保running方法，只能通过Person类或其子类进行调用
  running() {
    if (Person.#weakset.has(this)) {
      console.log('running')
    } else {
      throw new TypeError('type error')
    }
  }
}

class Student extends Person {}

const per = new Person()
const stu = new Student()

per.running() // => running
stu.running() // => running

const running = per.running
running() // error
```



## Map

1. 特殊对象
1. 对象的key可以是任意类型，不会进行任何转换
1. 通过Map方法创建，没有字面量创建形式
1. map没有任何可迭代非symbol属性 「 可以使用for-in迭代，但没有任何意义 」
1. map是可迭代对象

```js
// Map构造函数的参数可以是空的参数列表 也可以是一个entries结构的二维数组
const map = new Map([['name', 'Klaus'], [{name: 'Alex'}, 'Alex']])

console.log(map)
// => Map(2) { 'name' => 'Klaus', { name: 'Alex' } => 'Alex' }
```

```js
const map = new Map()

map.set(123, 123)
map.set({name: 'Klaus'}, 'Klaus')
map.set([], [1, 2, 3])

console.log(map)
// => Map(3) { 123 => 123, { name: 'Klaus' } => 'Klaus', [] => [ 1, 2, 3 ] }
```



| 属性 | 说明                |
| ---- | ------------------- |
| size | 返回Map中数据项个数 |



| 方法                           | 说明                                                     |
| ------------------------------ | -------------------------------------------------------- |
| set(key, value)                | 添加元素，并返回map本身                                  |
| get(key)                       | 根据key获取Map中的value<br />如则返回，无则返回undefined |
| has(key)                       | 判断key是否存在，返回布尔值                              |
| delete(key)                    | 删除key对应的键值对，返回布尔值                          |
| clear()                        | 清空map                                                  |
| forEach(callback, [, thisArg]) | map支持forEach迭代                                       |
| for-of                         | map是可迭代的                                            |



```js
const map = new Map()

map.set('name', 'Klaus')
map.set({ name: 'Klaus' }, 'object')
map.set([], 'array')

map.forEach((item, key) => console.log(key, item))
/*
  =>
    name Klaus
    { name: 'Klaus' } object
    [] array
*/


for (const mapEntry of map) {
  console.log(mapEntry)
  /*
    =>
      [ 'name', 'Klaus' ]
      [ { name: 'Klaus' }, 'object' ]
      [ [], 'array' ]
  */
}
```



### weakMap

1. key只能是对象
2. key是弱引用，值不是弱引用
3. 弱引用无法确保每次迭代结果都是一致的，因此无法打印内部weakmap或对weakmap进行迭代

| 方法            | 说明                                    |
| --------------- | --------------------------------------- |
| set(key, value) | 添加元素，并返回weakmap                 |
| get(key)        | 根据key获取value                        |
| has(key)        | 根据key判断数据项是否存在，并返回布尔值 |
| delete(key)     | 根据key删除数据项，并返回布尔值         |

```js
const weakmap = new WeakMap()

weakmap.set({ name: 'Klaus' }, 'object')
weakmap.set([], 'array')

console.log(weakmap) // => WeakMap { <items unknown> }

// 参数不是对象，直接报错，不进行类型转换
weakset.add(123) // error
```



`应用场景`

`WeakMap`常用于在对象上关联一些附加数据，并期望在对象被GC回收时，附加的相关信息也会被自动清除



`关联私有数据`

```js
class MyObject {
  static #privateData = new WeakMap()
  
  constructor() {
   	MyObject.#privateData.set(this, { secret: 'hiddenValue' })
  }

  getSecret() {
    return MyObject.#privateData.get(this).secret
  }
}

// 使用示例
const obj = new MyObject();
console.log(obj.getSecret()); // 输出: hiddenValue
```



`为DOM存储额外信息`

```js
const elementData = new WeakMap()

function setData(element, data) {
  elementData.set(element, data)
}

function getData(element) {
  return elementData.get(element)
}
```



`弱引用事件监听`

```js
const listeners = new WeakMap()

function addListener(obj, listener) {
  if (!listeners.has(obj)) {
    listeners.set(obj, [])
  }
  
  listeners.get(obj).push(listener)
}

function triggerListeners(obj, event) {
  const objListeners = listeners.get(obj)
  if (objListeners) {
    objListeners.forEach(listener => listener(event))
  }
}
```



## FinalizationRegistry

一般作为测试时使用

创建一个注册表，并将对象注册到注册表中，当该对象被GC回收时，会触发`FinalizationRegistry`中预设的回调，从而告诉我们什么时候对应对象被GC回收了

```js
// FinalizationRegistry是一个类，实例是注册表对象
const finalRegister = new FinalizationRegistry(value => {
  console.log(`${value}被GC清除了`)
})

let obj = { name: 'obj' }
let foo = { name: 'foo' }

// 通过`register(对象，标识符)`注册对象
// 当对象被GC回收时，会触发注册表中的回调，其中value参数的值就是这里设置的标识符
// 标识符可以是任意类型数据
finalRegister.register(foo, 'foo')
finalRegister.register(obj, 'obj')

// 对象没有任何强引用指向时，会被标记为无用对象, GC会在未来某个时间点进行清理
obj = null
foo = null
```

