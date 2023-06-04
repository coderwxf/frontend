生成器是ES6中新增的一种`函数控制、使用的方案`，它可以让我们`更加灵活的控制函数什么时候继续执行、暂停执行`等

在没有生成器之前，我们只能使用return返回一个值或抛出异常的方式，来结束函数的执行

有了生成器后，我们可以自主控制函数在哪里暂停执行，在哪里又重新开始执行



### 生成器函数

我们一般不会主动去创建一个生成器，而是使用生成器函数来返回一个生成器

**生成器函数也是一个函数，但是和普通的函数有一些区别：**

1. 生成器函数需要在function的后面加一个符号:`* `
   + `*`的前边和后边需不需要有空格，需要加上几个空格，这些都是没有明确定义的
   + 也就是`*`前后加不加空格，加几个都是可以的
   + 推荐使用 `function* funName`
2. 生成器函数可以通过yield关键字来控制函数的执行流程
   + 可以认为yield就是生成器函数中的一种类似于debugger的机制
3. 生成器函数的返回值是一个Generator(生成器）对象
   + 生成器本质上就是一种特殊的迭代器，也就是说生成器也是可以调用next方法的

```js
function* foo() {
  const value1 = 'value1'
  console.log('第一段: ', value1)
  yield

  const value2 = 'value2'
  console.log('第二段: ', value2)
  yield

  const value3 = 'value3'
  console.log('第三段: ', value3)
  yield

  console.log('foo执行完毕')
}

// 此时，foo函数中任意一行代码都没有被执行
// foo这个生成器方法在执行后只会返回一个生成器对象
const generator = foo()

// 我们可以使用生成器对象上的next方法来依次执行对应的代码段
generator.next() // 第一段:  value1 --- 通过yield中断执行 --- done = false
generator.next() // 第二段:  value2 --- 通过yield中断执行 --- done = false
generator.next() // 第三段:  value3 --- 通过yield中断执行 --- done = false
generator.next() // foo执行完毕 --- 通过return中断执行 --- done = true

// 生成器对应片段全部执行完成后，再次调用next方法会静默失效
generator.next() // 执行该函数后 返回值为 { value: undefined, done: true }
```

```js
function* foo() {
  const value1 = 'value1'
  // 我们可以使用yield关键字来临时终止一段代码的执行
  // 同时我们可以使用yield来为一段代码返回对应的值
  // 和return一样，默认情况下yield的返回值是undefined
  yield value1

  const value2 = 'value2'

  // yield后边可以跟上任何的合法的js表达式
  yield value2 + 200

  const value3 = 'value3'
  yield value3

  // 最后一个return的返回值会被作为最后一个next方法返回的value值
  // 默认情况下不写就是undefined
  // 而return 在生成器中就可以被认为是一种特殊的yield
  return 123
}

const generator = foo()

console.log(generator.next()) // => { value: 'value1', done: false }
console.log(generator.next()) // => { value: 'value2200', done: false }
console.log(generator.next()) // => { value: 'value3', done: false }

// 当生成器执行到return的时候，会返回 { value: <return的返回值>, done: true }
// 此时会自动将done设置为true并终止迭代
// 而不是像迭代器一样，最后一个值的时候done的值为false
// 需要在迭代一次，对应的done值才会被改成true
console.log(generator.next()) // => { value: 123, done: true }
```



### next方法

函数是可以传递参数的，生成器中对应的每一个片段也是可以传递对应参数的

我们可以在执行next方法的时候，给next方法传递对应的参数，所传入的参数会作为上一个yield语句的返回值

```js
function* foo(param) {
  console.log(param) // => 10
  const x = yield param

  console.log(x) // => 20
  const y = yield x

  console.log(y) // => 30
  const z = yield y

  console.log(z) // => 40
  return z
}

// 如果需要给第一个代码段的执行传递对应的参数
// 可以在执行生成器函数的时候，作为函数的参数进行传入
const generator = foo(10)

// 第一个next方法传参数是没有意义的
// 因为第一个next方法之前是没有yield存在的
// next中的参数会作为yield表达式的返回值
console.log(generator.next())

console.log(generator.next(20))
console.log(generator.next(30))
console.log(generator.next(40))
```



### return方法

```js
function* foo(param) {
  const x = yield param

  // 使用了return方法之后，类似于在第二段代码执行之前直接执行了return x
  // 后续代码将不再被执行

  const y = yield x

  const z = yield y
  return z
}

const generator = foo(10)

console.log(generator.next()) // => { value: 10, done: false }
// 执行return方法，生成器立即停止执行，并将return方法中的参数作为返回的对象的value属性值
console.log(generator.return(20)) // => { value: 20, done: true }

// 使用return方法后，我们依旧可以使用return方法返回对应的值
console.log(generator.return(20)) // => { value: 20, done: true }
console.log(generator.return(20)) // => { value: 20, done: true }

// 但是使用return方法后，generator函数的执行就被彻底终止了
console.log(generator.next()) // => { value: undefined, done: true }
console.log(generator.next()) // => { value: undefined, done: true }
```



### throw方法

```js
function* foo(param) {
  try {
    // 外部在执行生成器的时候 使用了generator.throw
    // 相当于yield param的时候 出现了错误 抛出了异常
    // 所以会直接进入catch语句
    const x = yield param
  } catch(e) {
    // catch语句中不可以使用yield关键字
    // 在catch语句中的yield关键字会静默失效
    console.log(e)
  }

  // 因为之前已经使用try - catch捕获了对应的错误
  // 所以后续的代码依旧会被正常执行
  const y = yield 111

  const z = yield y
  return z
}

const generator = foo(10)

console.log(generator.next())

console.log(generator.throw('error message'))

console.log(generator.next(22))
console.log(generator.next(33))
```



### 生成器替代迭代器

之前我们自定义数据结构的遍历使用的是自定义迭代器

```js
const users =  ['Alex', 'Klaus', 'Steven']

function createIterator(v) {
  let index = 0

  return {
    next() {
      return {
        done: index >= users.length,
        value: users[index++]
      }
    }
  }
}

const iterator = createIterator(users)

console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
```



而既然生成器是一种特殊的迭代器，也就是生成器和迭代器在使用层面是完全一致的

所以在绝大多数情况下，我们可以使用生成器函数来替代原本的迭代器代码

```js
const users =  ['Alex', 'Klaus', 'Steven']

function* createIterator(v) {
  for (let i = 0; i < v.length; i++) {
    yield v[i]
  }
}

const iterator = createIterator(users)

console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
```

事实上我们还可以使用yield*来自动迭代一个可迭代对象

```js
const users =  ['Alex', 'Klaus', 'Steven']

function* createIterator(v) {
  // yield* 后边需要加上一个可迭代对象
  // yield* 是一种yield的语法糖，只不过yield*会依次迭代这个可迭代对象，每次迭代其中的一个值
  yield* v
}

const iterator = createIterator(users)

console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
console.log(iterator.next())
```

```js
class School {
  constructor(students) {
    this.students = students
  }

  // 在类上定义生成器
  *[Symbol.iterator]() {
    yield* this.students
  }
}

const school = new School(['Alex', 'Klaus', 'Steven'])

for(const stu of school) {
  console.log(stu)
}
```

