在ES6之前，我们存储数据的结构主要有两种：数组、对象

在ES6中新增了另外两种数据结构：Set (集合)、Map (映射)，以及它们的另外形式WeakSet、WeakMap



## Set

Set是一个新增的数据结构，可以用来保存数据，类似于数组，但是和数组的区别是`元素不能重复`

创建Set我们需要通过Set构造函数（暂时没有字面量创建的方式）

Set中存放的元素是不会重复的，所以Set有一个非常常用的功能就是给数组去重

Set没有任何的可迭代的非symbol属性，所以对Set使用for-in遍历，不会报错，但并不会迭代出任何的内容，所以对Set使用for-in遍历是没有意义的

```js
// Set构造函数的参数可以不传，也可以传递任意的可迭代对象
// Set用于存储一组唯一的数据
// Set并没有为每一个元素提供对应的索引值也没有添加对应的get方法
// 所以如果需要获取Set中的某个特定的值，可以使用以下方法：
// 1. 将set转换为原生数组
// 2. 使用Set.prototype.keys() 或 Set.prototype.values()
//    		或 set[Symbol.iterator]()
//    获取到Set对应的迭代器 通过迭代来获取Set中的每一个元素值
const set = new Set([1, 2, 3, 4, 1])

// set转普通数组
// set也是一个可迭代对象
console.log(Array.from(set)) // => [ 1, 2, 3, 4 ]
console.log([...set]) // => [ 1, 2, 3, 4 ]
```



### 属性和方法

| 属性 | 说明                |
| ---- | ------------------- |
| size | 返回Set中元素的个数 |



| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| add(value)                     | 添加某个元素，返回Set对象本身                                |
| delete(value)                  | 从set中删除和这个值相等的元素，返回boolean类型               |
| has(value)                     | 判断set中是否存在某个元素，返回boolean类型                   |
| clear()                        | 清空set中所有的元素，没有返回值                              |
| keys()                         | 返回set的迭代器                                              |
| values()                       | 返回set的迭代器                                              |
| `[Symbol.iterator`]()          | 返回set的迭代器                                              |
| forEach(callback, [, thisArg]) | 通过forEach遍历set                                           |
| for-of                         | 任何的可迭代对象都可以通过for-of遍历，而set也是一种可迭代的数据结构<br />所以set也可以使用for-of迭代 |

```js
const set = new Set([1, 2, 3, 4])

// 在set中, set中的索引值 和 set对应的元素值 是一致的
set.forEach((item, index) => console.log(item, index))
/*
  =>
    1 1
    2 2
    3 3
    4 4
*/
```



## weakset

和Set类似的另外一个数据结构称之为WeakSet，也是内部元素不能重复的数据结构

但是weakSet内部使用的是弱引用(weak reference)



### 弱引用 和 强引用

在JS中，默认情况下，所有的引用都是强引用(strong reference), 也就是会被GC在标记清除的时候，计算在内

如果一个元素被一个强引用所指向，那么这个元素就不被GC因为是垃圾元素，而被GC清除

而ES6后，新增了弱引用，弱引用是不会被GC作为可达性计算在内的

也就是说如果一个元素没有强引用所指向，即使其有n个弱引用指向，GC依旧会认为这个元素是垃圾元素，而移除该元素



所以如果我们需要一个引用，但是我们不希望该引用导致后期该引用所指向的元素因为该引用的指向而无法正常的被GC所清除的时候

我们就可以将这个引用设置为弱引用

弱引用是一种在垃圾回收时不会阻止被引用对象被释放的引用

而`weakset`和`weakmap`对于其数据结构中所有的元素的引用皆为弱引用



### weakset 和 set

1. WeakSet中只能存放对象类型，不能存放基本数据类型
2. WeakSet对元素的引用是弱引用，如果没有其他引用对某个对象进行引用，那么GC可以对该对象进行回收

```js
const weakset = new WeakSet()

weakset.add({name: 'Klaus'})
// weakset因为对于其内部元素的引用是弱引用
// 所以无法保证其内部元素的个数和内容
// 所以对于weakset而言，其是无法打印内部元素 或 对weakset进行迭代的
console.log(weakset) // => WeakSet { <items unknown> }

// 如果weakset.add的参数不是一个对象类型的时候
// weakset会直接报错，并不会尝试将对应的参数转换为对应的包装类类型
weakset.add(123) // error
```



| 方法          | 说明                                               |
| ------------- | -------------------------------------------------- |
| add(value)    | 添加某个元素，返回WeakSet对象本身                  |
| delete(value) | 从WeakSet中删除和这个值相等的元素，返回boolean类型 |
| has(value)    | 判断WeakSet中是否存在某个元素，返回boolean类型     |



### weakset的应用

```js
class Person {
  static #weakset = new WeakSet()

  constructor() {
    Person.#weakset.add(this)
  }
  
  // 可以使用weakset来确保 当前类的实例方法
  // 只能通过类的实例对象来进行调用, 而不可以使用其它形式来进行调用
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

Map用于存储映射关系, 其和对象的最大区别是 

+ 对象的key只能是字符串或symbol类型的值，如果是其余类型的值，会先将其余类型的值转换为字符串后在作为对象的key来进行使用
+ 而map的key可以是任意的数据类型，也就意味着map的key可以是symbol类型值，字符串，也可以是对象或其它任意的数据结构

创建Map我们需要通过Map构造函数（暂时没有字面量创建的方式）

Map没有任何的可迭代的非symbol属性，所以对Map使用for-in遍历，并不会报错，但是不会迭代出任何的内容，所以对Map使用for-in遍历是没有意义的

```js
const map = new Map()

map.set(123, 123)
map.set({name: 'Klaus'}, 'Klaus')
map.set([], [1, 2, 3])

console.log(map)
// => Map(3) { 123 => 123, { name: 'Klaus' } => 'Klaus', [] => [ 1, 2, 3 ] }
```

```js
// Map构造函数的参数可以是空的参数列表 也可以是一个entries结构的二维数组
const map = new Map([['name', 'Klaus'], [{name: 'Alex'}, 'Alex']])

console.log(map)
// => Map(2) { 'name' => 'Klaus', { name: 'Alex' } => 'Alex' }
```



| 属性 | 说明                |
| ---- | ------------------- |
| size | 返回Map中元素的个数 |



| 方法                           | 说明                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| set(key, value)                | 在Map中添加key、value，并且返回整个Map对象                   |
| get(key)                       | 根据key获取Map中的value<br />如果不存在对应的值，则返回undefined |
| has(key)                       | 判断是否包括某一个key，返回Boolean类型                       |
| delete(key)                    | 根据key删除一个键值对，返回Boolean类型                       |
| clear()                        | 清空所有的元素                                               |
| keys()                         | 返回一个迭代器，用于迭代Map的keys                            |
| values()                       | 返回一个迭代器，用于迭代Map的values                          |
| entries()                      | 返回一个迭代器，用于迭代Map的entries (即[key, value])        |
| `[Symbol.iterator]()`          | 同Map.prototype.entries()                                    |
| forEach(callback, [, thisArg]) | 通过forEach遍历Map                                           |
| for-of                         | 任何的可迭代对象都可以通过for-of遍历，而map也是一种可迭代的数据结构<br />所以map也可以使用for-of迭代 |

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



## weakMap

和Map类型的另外一个数据结构称之为WeakMap，也是以键值对的形式存在的



### weakmap 和 Map

1. WeakMap的key只能使用对象，不接受其他的类型作为key
2. WeakMap对元素的引用是弱引用，如果没有其他引用对某个对象进行引用，那么GC可以对该对象进行回收



### 方法

| 方法            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| set(key, value) | 在Map中添加key、value，并且返回整个Map对象                   |
| get(key)        | 根据key获取Map中的value<br />如果不存在对应的值，则返回undefined |
| has(key)        | 判断是否包括某一个key，返回Boolean类型                       |
| delete(key)     | 根据key删除一个键值对，返回Boolean类型                       |



和 WeakSet一样， WeakMap一样不能保证每次迭代获取到的元素是一致的

所以WeakMap也是不可以打印其中的元素 和 遍历

