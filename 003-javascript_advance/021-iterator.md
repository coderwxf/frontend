**迭代器**(iterator)是帮助我们`对某个数据结构或容器对象(container，例如链表或数组)进行遍历`的`对象`， 其行为类似于数据库中的光标

使用迭代器遍历某一个对象的时候，我们不需要关心被遍历的对象的内部实现细节

简而言之 迭代器是一个`帮助我们遍历另一个容器对象`的`对象`

在各种编程语言的实现中，迭代器的实现方式各不相同，但是基本都有迭代器，比如Java、Python等

在JS中如果一个对象正确实现了的next方法，那么我们就可以认为这个对象实现了迭代器协议(iterator protocol)， 而这个对象就可以被认为是迭代器对象

迭代器协议定义了产生一系列值(有限个或无限个值)的标准方式

这个next方法需要满足如下几点:

1. 一个无参数或者一个参数的函数
2. 该对象需要返回如下两个属性

| 属性  | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| done  | boolean类型值<br />done主要用于标识该迭代器是否将对象遍历完毕<br />如果done的值为false，表示迭代没有完成<br />此时行为和没有设置done的值是一致的 |
| value | 迭代器返回的任何 JavaScript 值<br />当done为true时，也就是迭代完成时，对应的值为undefined<br />此时value属性可以省略<br />ps: 迭代完成后，返回的value值并不强制是undefined，可以是任意值，作为迭代完成后的返回值<br />但是一般迭代完成后的返回值都是undefined |

```js
const users = ['Klaus', 'Alex', 'Steven']

function createIterator(arr) {
  let index = 0

  return {
    next() {
      return {
        done: index >= arr.length,
        value: arr[index++]
      }
    }
  }
}

const iterator = createIterator(users)

console.log(iterator.next()) // => { done: false, value: 'Klaus' }
console.log(iterator.next()) // => { done: false, value: 'Alex' }
console.log(iterator.next()) // => { done: false, value: 'Steven' }
console.log(iterator.next()) // => { done: true, value: undefined }
// 迭代器 迭代完毕后，依旧可以继续调用next方法
// 只不过返回值都是 { done: true, value: undefined }
console.log(iterator.next()) // => { done: true, value: undefined }
```

当然，并不是每一个迭代器都可以将数据遍历完毕，存在无限的迭代器，只不过比较少见

```js
function createIterator() {
  let index = 0

  return {
    next() {
      return {
        done: false,
        value: index++
      }
    }
  }
}

const iterator = createIterator()

console.log(iterator.next()) // => { done: false, value: 0 }
console.log(iterator.next()) // => { done: false, value: 1 }
console.log(iterator.next()) // => { done: false, value: 2 }
console.log(iterator.next()) // => { done: false, value: 3 }
console.log(iterator.next()) // => { done: false, value: 4 }
```



## 可迭代对象

在`iterable protocol`中, 如果一个对象正确实现了`@@iterator`方法的时候, 这个对象就被称之为可迭代对象，而`@@iterator`方法就是这个迭代这个对象时候需要使用到的迭代器

所以迭代器和可迭代对象并不是同一种东西

在JS中，`@@iterator`对应的就是`Symbol.iterator`方法，也就是说如果一个对象正确实现了`Symbol.iterator`方法，那么这个对象就被称之为可迭代对象

```js
// 只有当迭代器和迭代对象放在一起，组成一个完整的对象的时候，这个对象才是可迭代对象
// 如果一个对象，在该对象外有可以迭代该对象的对象(如之前所举的例子)，则这个对象并不是可迭代对象
// 只有当迭代器和对应迭代对象组合成一个对象的时候，对应的对象才可以被称之为可迭代对象
// 所以在这里 iterableObj 是可迭代对象
const iterableObj = {
  users: ['Klaus', 'Alex', 'Steven'],
 
  // 这里的Symbol.iterator是一个方法
  // 用于生成迭代iterableObj的迭代器
  // 所以Symbol.iterator方法需要返回迭代对象自身的迭代器
  // Symbol.iterator是通过iterableObj[Symbol.iterator]()来获取的
  // 所以Symbol.iterator中的this就是iterableObj
  [Symbol.iterator]() {
    let index = 0

    // 返回的是一个对象，并不是一个代码块，所以没有自己的作用域
    return {
      // next方法是通过iterator.next()来进行调用的
      // 所以next方法中的this是iterator
      // 所以 在next方法中需要使用箭头函数来修正内部this的指向
      next: () => ({
        done: index >= this.users.length,
        value: this.users[index++]
      })
    }
  }
}

const iterator = iterableObj[Symbol.iterator]()

console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
```



## for - of

任意一个可迭代对象，都可以使用`for ... of`方法遍历可迭代对象中的值

可以认为`for ... of`是直接调用next方法进行遍历的一种语法糖

事实上我们平时创建的很多原生对象已经实现了可迭代协议，也就是说他们本身就是可迭代对象

String、Array、Map、Set、伪数组对象(如arguments对象、NodeList集合)在默认情况下都是一个可迭代对象

```js
const iterableObj = {
  users: ['Klaus', 'Alex', 'Steven'],
  [Symbol.iterator]() {
    let index = 0

    return {
      next: () => ({
        done: index >= this.users.length,
        value: this.users[index++]
      })
    }
  }
}

for (const value of iterableObj) {
  console.log(value)
}
```

这也就是为什么对象默认情况下，不能直接使用`for ... of`进行遍历，因为对象默认情况下并不是一个可迭代对象

```js
const user = {
  name: 'Klaus',
  age: 23
}

// 以下代码会直接报错 --- 是不合法的
// 注意 对象默认不可以使用for...of进行遍历，因为JS原生对象并不是可迭代对象
// 但是对象是可以通过for-in遍历获取到所有的属性名
for (const value of user) {
  console.log(value)
}
```

```js
// 需要注意的是，虽然对象默认情况下不可以使用for...of遍历或作为剩余参数进行传递
// 但是对象是可以使用展开运算符和解构操作的
// 这是ES9(ES2018)中对对象进行的特殊处理，其内部使用的并不是迭代器进行实现

const user = {
  name: 'Klaus',
  age: 23
}

const { name, age } = user
console.log(name, age) // => Klaus 23

console.log({...user}) // => { name: 'Klaus', age: 23 }
```

```js
// 一些JS默认内置对象就是可迭代对象，也就是默认实现了迭代器
// 例如数组，字符串，map，set，arguments, NodeList等
const users = ['Alex', 'Steven', 'Joy']

// 获取数组原生迭代器
const iterator = users[Symbol.iterator]()

let isFinish = false

// 所以遍历属性有一种全新的方式
do {
  const { done, value } = iterator.next()
  isFinish = done
  if (value) console.log(value)

} while(!isFinish)
```

```js
const user =  {
  name:  'Klaus',
  age: 23,
  [Symbol.iterator]() {
    let index = 0

    // Object.keys(), Object.values(), Object.entries()
    // 这三个方法在获取属性的时候，是不包括原型上的属性和symbol类型属性的
    // 所以在这里获取的结果中没有 Symbol.iterator 属性
    let entries = Object.keys(user)
    const iterator = {
      next: () => ({
        value: index < entries.length ? entries[index] : undefined,
        // index在done中依旧需要使用，所以应该在done中++，而不是在value中++
        done: index++ >= entries.length
      })
    }

    return iterator
  }
}

for (const entry of user) {
  console.log(entry)
}
```



### for - in vs for - of

| 迭代方式         | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| for key in xxx   | 用于迭代对象或数组的属性<br />迭代set或map，不会报错，但是没有结果<br />for - in可以迭代一个`对象及其原型链上`的所有`可迭代`属性<br />for - in迭代出的属性是有序的(先是对象自身的再是对象的原型上的，依次类推) |
| for value of xxx | 用于迭代可迭代对象<br />迭代本质调用的是可迭代对象的`[Symbol.iterator]`方法 |



## 使用场景

1. JavaScript中语法: for ...of、展开语法(spread syntax)、yield*、解构赋值(Destructuring_assignment)
2. 创建一些对象时: new Map([Iterable])、new WeakMap([iterable])、new Set([iterable])、new WeakSet([iterable])
3. 一些方法的调用: Promise.all(iterable)、Promise.race(iterable)、Array.from(iterable)

```js
const iterableObj = {
  users: ['Klaus', 'Alex', 'Steven'],
  [Symbol.iterator]() {
    let index = 0

    return {
      next: () => ({
        done: index >= this.users.length,
        value: this.users[index++]
      })
    }
  }
}

// 可迭代对象可以使用for ... of循环
for (const value of iterableObj) {
  console.log(value)
}

// 可迭代对象可以使用展开运算符
console.log([...iterableObj]) // => [ 'Klaus', 'Alex', 'Steven' ]

// 可迭代对象可以使用解构运算符
const [user1, user2, user3] = iterableObj
console.log(user1, user2, user3) // => Klaus Alex Steven
```

```js
const iterableObj = {
  users: ['Klaus', 'Alex', 'Steven'],
  
  [Symbol.iterator]() {
    let index = 0

    return {
      next: () => ({
        done: index >= this.users.length,
        value: this.users[index++]
      })
    }
  }
}

// 某些对象在创建的时候，也可以传入迭代器作为创建时候的参数
const set = new Set(iterableObj)
console.log(set) // => Set(3) { 'Klaus', 'Alex', 'Steven' }

// 某些方法的参数也可以接收可迭代对象
Promise.all(iterableObj).then(res => console.log(res)) // => [ 'Klaus', 'Alex', 'Steven' ]
// 等价于
/*
  // 基本数据类型会使用Promise.resolve方法进行包裹
  Promise.all([
    Promsie.resolve('Klaus'),
    Promsie.resolve('Alex'),
    Promsie.resolve('Steven')
  ]).then(res => console.log(res))
*/
```



## 自定义类的迭代

我们知道Array、Set、String、Map等类创建出来的对象默认都是可迭代对象

因此我们也可以通过实现类上的`Symbol.iterator`方法，从而确保该类所创建出来的对象默认是可迭代的

```js
class School {
  constructor(students) {
    this.students = students
  }

  push(student) {
    this.students.push(student)
  }

  // 因为School类实现了正确的Symbol.iterator方法
  // 所以使用School类创建出来的方法默认都是可迭代的
  // 对School类的实例进行遍历的时候，默认会依次取出对应students数组中的各个值
  [Symbol.iterator]() {
    let index = 0

    return {
      next: () => ({
        done: index >= this.students.length,
        value: this.students[index++]
      })
    }
  }
}

const school = new School(['Alex', 'Klaus', 'Steven'])

school.push('Jhon')

for (const stu of school) {
  console.log(stu)
}
/*
  =>
    Alex
    Klaus
    Steven
    Jhon
*/
```



## 迭代器的中断

迭代器在某些情况下会在没有完全迭代的情况下中断，比如遍历的过程中通过break、return、throw中断了循环操作

也就是说和forEach之类的方法不同，使用for-of之类的方法迭代可迭代对象的时候，是可以提前中止的

迭代器的`return` 方法是在迭代器中途需要提前终止迭代时需要调用的方法

它允许迭代器在不正常终止的情况下终止迭代，并返回一个给定的值

在for-of循环被提前中断，或对可迭代对象进行解构操作(无论是完全解构还是部分解构)，JS内部都会去调用`iterator.return`方法

```js
const iterableObj = {
  users: ['Klaus', 'Alex', 'Steven'],
  [Symbol.iterator]() {
    let index = 0

    return {
      next: () => ({
        done: index >= this.users.length,
        value: this.users[index++]
      }),

      // 如果迭代器在遍历的时候被终止了，那么会执行return函数
      // 以下是默认的return函数
      // 如果我们需要在iterator在遍历被终止的时候执行某些操作可以在return回调中编写对应的逻辑
      return() {
        // 迭代器中的方法都需要返回由done和value组成的对象
        return {
          done: true,
          value: undefined
        }
      }
    }
  }
}

for (const user of iterableObj) {
  if (user === 'Alex') {
    return
  } else {
    console.log(user)
  }
}
```

当解构一个可迭代对象的时候，当迭代完成后，对应的break方法也会被执行

```js
const iterableObj = {
  users: ['Klaus', 'Alex', 'Steven'],
  [Symbol.iterator]() {
    let index = 0

    return {
      next: () => ({
        done: index >= this.users.length,
        value: this.users[index++]
      }),

      return() {
        console.log('iterator breaked')

        return {
          done: true,
          value: undefined
        }
      }
    }
  }
}

// 即使所有的值全部被解构完全了，return方法依旧会被触发
const [user1, user2, user3] = iterableObj
console.log(user1, user2, user3)
```



