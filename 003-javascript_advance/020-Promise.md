## 异步任务的处理

在ES5之前，我们一般会自己编写异步处理函数，来处理异步任务

```js
function asyncFn(count, resolve, reject) {
  if (count <= 0) {
    reject('count不能小于0')
  } else {
    // 使用定时器模拟异步请求
    setTimeout(() => {
      // 异步处理函数并不能马上得到对应的结果
      // 而是需要等待一定的时间后才会得到对应的处理结果
      // 因此需要传入回调函数，在异步任务处理完毕后，去执行回调函数中的逻辑
      resolve(count)
    }, 1000)
  }
}

asyncFn(100, count => console.log(count), err => {
  throw new Error(err)
})
```



所以在ES5之前，异步任务处理函数，都是自行进行编写对应的处理函数

这就导致对于不同的人、不同的框架设计出来的方案是不同的，

为此社区出现过对于异步函数封装的规定 ， 即[promises/A+](https://promisesaplus.com/)

但社区规范毕竟只是约定输出的规范，并不是语法层面的规范

为此ES6中提出了Promise，用于提供官方的异步任务处理方案



## Promise

Promise是一个类，可以翻译成 承诺、许诺 、期约，凭证

```js
// 异步任务的参数在函数参数列表中传入
function asyncFn(count = 0) {
  // 对应的成功和失败回调在异步函数内部返回一个Promise
  // 并在Promise中定义即可
  // resolve - 成功时候触发的回调
  // reject - 失败时候触发的回调
  return new Promise((resolve, reject) => {
    // Promise函数传入的回调被称之为executor函数
    // executor函数会在Promise执行的时候同步执行
    // 也就是说Promise传入的回调函数是同步代码
    // 而在executor函数内部再去编写对应的异步代码
    if ( count <= 0 ) {
      reject('count的值必须大于0')
    } else {
      setTimeout(() => {
        resolve(count)
      }, 3000)
    }
  })
}

// 当我们调用resolve回调函数时，会执行Promise对象的then方法传入的回调函数
// 当我们调用reject回调函数时，会执行Promise对象的catch方法传入的回调函数
// then方法 返回的也是一个Promise对象，所以可以进行链式调用
asyncFn(10).then(count => console.log(count))
           .catch(err => { throw new Error(err) })
```





### 状态

![image.png](https://s2.loli.net/2022/06/30/ASmEnj4F8MBL7gu.png)

一个Promise在执行过程中，存在如下三种状态

| 状态              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| 待定(pending)     | 初始状态，既没有被兑现，也没有被拒绝<br />当执行executor中的代码时，处于该状态 |
| 已兑现(fulfilled) | 意味着操作成功完成<br />执行了resolve时，处于该状态，Promise已经被兑现 |
| 已拒绝(rejected)  | 意味着操作失败<br />执行了reject时，处于该状态，Promise已经被拒绝 |

Promise的状态只能从 pending 转变为 fulfilled 或 rejected 而且状态一旦发生转变就已经被确定，不可以再次改变

```js
function asyncFn(count = 0) {
  return new Promise((resolve, reject) => {
    resolve(count)
    console.log('-----')
    reject()

    /*
      =>
        ----
        10

      exectuor的代码会优先被执行完毕，因为executor中的代码位于主线程中, 所以会先输出 ----
      随后执行宏任务队列中的resolve方法，输出10
      Promise中的状态已经被转换，无法再次被改变，所以转换改变后的resolve和reject方法会静默失效
    */
  })
}

asyncFn(10).then(count => console.log(count))
           .catch(err => { throw new Error(err) })
```



### executor

Executor是在创建Promise时需要传入的一个回调函数，这个回调函数会被立即执行，并且传入两个参数 resolve和reject

通常我们会在Executor中确定我们的Promise状态:

+ 通过resolve，可以兑现(fulfilled)Promise的状态，我们也可以称之为已决议(resolved)
+ 通过reject，可以拒绝(reject)Promise的状态



这里需要注意: 一旦状态被确定下来，Promise的状态会被锁死，该Promise的状态是不可更改的

+ 在我们调用resolve的时候，如果resolve传入的值本身不是一个Promise，那么会将该Promise的状态变成 兑现(fulfilled)
+  在之后我们去调用reject时，已经不会有任何的响应了(并不是这行代码不会执行，而是无法改变Promise状态)



### resolve不同值

+ 如果resolve传入一个普通的值(即基本数据类型和引用数据类型)，那么这个值会作为then回调的参数

+ 如果resolve中传入的是另外一个Promise，那么这个新Promise会决定原Promise的状态

+ 如果resolve中传入的是一个对象，并且这个对象有实现then方法(这种对象被称之为thenable对象)

  那么会执行该then方法，并且根据then方法的结果来决定Promise的状态

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve([123, 222, 333])
  })
}

asyncFn().then(res => console.log(res)) // => [ 123, 222, 333 ]
```

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve(new Promise((_, reject) => {
        reject('error message')
    }))
  })
}

asyncFn().then(res => console.log(res)).catch(err => console.log('error: ' + err))
// => error: error message
```

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve({
      then(resolve, reject) {
        resolve('thenable msg')
      }
    })
  })
}

asyncFn().then(res => console.log(res)).catch(err => console.log('error: ' + err))
// => thenable msg
```



### 实例方法

#### then

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve('success')
  })
}

// then方法接受两个参数:
// 1. fulfilled的回调函数:当状态变成fulfilled时会回调的函数
// 2. reject的回调函数:当状态变成reject时会回调的函数
asyncFn().then(res => console.log(res), err => console.log(err)) // => success
```



##### then方法多次调用

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve('success')
  })
}

// 一个Promise的then方法是可以被多次调用的
// 相对于给Promise绑定了多个fulfilled监听函数
// 所以当promise的状态改变为resolved的时候
// 对应的事件回调会依次被执行
asyncFn().then(() => console.log('第一次调用'))
asyncFn().then(() => console.log('第二次调用'))
asyncFn().then(() => console.log('第三次调用'))
asyncFn().then(() => console.log('第四次调用'))
asyncFn().then(() => console.log('第五次调用'))
/*
  =>
    第一次调用
    第二次调用
    第三次调用
    第四次调用
    第五次调用
*/
```



##### 链式调用

then方法本身是有返回值的，它的返回值是一个Promise，所以我们可以进行如下的链式调用:

```js
// 伪代码
// 每一个then的状态是由上一个then方法的返回值的状态所决定的
// 当then方法中的回调函数返回一个结果时，那么它处于fulfilled状态，并且会将结果作为resolve的参数
promise.then().then().then().catch()
```

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve('success')
  })
}

// 1. 第一个then的状态是由asyncFn方法中的promise决定的

// 2. 第二个then方法的状态是由第一个then方法的状态所决定的
//    第一个then方法返回了undefined，所以返回的Promise实际等价Promise.resolve(undefined)
//    因此 第二个then方法的res的值为 undefined

// 3. 第三个then方法的状态是由第二个then方法的状态所决定的
//    第二个方法返回了 'then方法返回的结果'， 等价于 Promise.resolve('then方法返回的结果')
//    因此 第三个then方法的res的值为  'then方法返回的结果'
asyncFn().then(res => console.log(res)).then(res => {
  console.log(res)
  return 'then方法返回的结果'
}).then(res => console.log(res))
/*
  =>
    success
    undefined
    then方法返回的结果
*/
```



##### 返回值

then方法的返回值也是一个Promise,

+ 当then方法中的回调函数本身在执行的时候，那么它处于pending状态

+ 当then方法中的回调函数返回一个结果时:
  + 返回一个普通的值, 那么它处于fulfilled状态，并且会将结果作为resolve的参数
  + 返回一个Promise, 当前promise的状态由返回的这个新promise来决定
  + 返回一个thenable值，当前promise的状态由thenable对象的then方法执行结果决定
+ 当then方法抛出一个异常时，那么它处于reject状态



#### catch

```js
function asyncFn() {
  return new Promise(resolve => {
    reject('success')
  })
}

// 当promise抛出异常的时候，会被离其最近的catch方法所捕获
asyncFn().then(res => console.log(res)).catch(err => console.log(err))
```



##### catch方法多次调用

```js
function asyncFn() {
  return new Promise(resolve => {
    reject('error')
  })
}

// 一个Promise的catch方法是可以被多次调用的
// 相对于给Promise绑定了多个reject监听函数
// 所以当promise的状态改变为rejected的时候
// 对应的事件回调会依次被执行
asyncFn().catch(() => console.log('error 1'))
asyncFn().catch(() => console.log('error 2'))
asyncFn().catch(() => console.log('error 3'))
asyncFn().catch(() => console.log('error 4'))
asyncFn().catch(() => console.log('error 5'))
/*
  =>
    error 1
    error 2
    error 3
    error 4
    error 5
*/
```



##### 返回值

catch方法也是会返回一个Promise对象的，所以catch方法后面我们可以继续调用then方法或者catch方法

```js
function asyncFn() {
  return new Promise(resolve => {
    resolve('success')
  })
}

asyncFn().then(() => {
  // 在then方法中抛出的异常，会被返回的Promise所捕获
  // 并将异常作为reject的参数被传入，即作为catch方法的参数被传入
  throw new Error('then中抛出的异常')
}).catch(err => console.log(err))
// catch方法返回的也是一个Promsie
.then(res => console.log(res))
/*
  =>
    Error: then中抛出的异常
    undefined
*/
```



```js
function asyncFn() {
  return new Promise((_, reject) => {
    reject('error')
  })
}

// promise抛出的异常会被传递到下一个promise中
// 如果下一个promise也没有处理，那么这个异常会一直向下进行传递
// 知道被catch方法所捕获，或者没有catch处理的时候，会抛给浏览器
// 而此时浏览器会将没有处理的异常作为错误 输出到控制台中
asyncFn().then(res => console.log(res))
.then(res => console.log(res)).then(res => console.log(res))
.catch(err => console.log(err)) // => error
```



```js
function asyncFn() {
  return new Promise((_, reject) => {
    reject('error')
  })
}

// 虽然在这行语句后边存在catch方法
// 但是那是针对于第二个asyncFn()返回的promise的异常进行处理的
// 也就是对于第一个asyncFn()方法返回的promise只监听了resolved方法
// 并没有对reject方法进行监听，因此会报错
// 即 如果没有写then的onRejected方法，则会层层throw Error
// 因此在实际书写的时候，每一个promise都应该加上对应的catch方法
asyncFn().then(res => console.log(res)) // error

asyncFn().then(res => console.log(res)).catch(err => console.log(err))
```



#### finally

finally是在ES9(ES2018)中新增的一个特性

表示无论Promise对象无论变成fulfilled还是rejected状态，最终都会被执行的代码

finally方法是不接收参数的，因为无论前面是fulfilled状态，还是rejected状态，它都会执行

```js
function asyncFn() {
  return new Promise((resovle, reject) => {
    resovle('success')
  })
}

// finally本身返回的也是一个promise
// 所以从理论角度来讲，可以一直链式下去
asyncFn().then(res => console.log(res))
  .catch(err => console.log(err))
  .finally(() => console.log('finally'))
```



### 静态方法

#### resolve

有时候我们已经有一个现成的内容了，希望将其转成Promise来使用，这个时候我们可以使用 Promise.resolve 方法来完成

Promise.resolve的用法相当于new Promise，并且执行resolve操作 这种行为的语法糖

```js
Promise.resolve('resolve msg').then(res => console.log(res))

// 上述代码等价于
!(new Promise(resolve => resolve('resolve msg'))).then(res => console.log(res))
```

Promise.resolve的参数可以是普通值，也可以是一个新的promise对象或者是一个thenable对象

对应的规则和之前是一致的

```js
Promise.resolve(Promise.reject('error'))
  .then(res => console.log(res))
  .catch(err => console.log(err)) // => error
```



#### reject

reject方法类似于resolve方法，只是会将Promise对象的状态设置为reject状态

Promise.reject的用法相当于new Promise，并且调用reject方法 的 语法糖

```js
Promise.reject('error msg').catch(err => console.log(err))

// 等价于
!(new Promise((_, reject) => reject('error msg'))).catch(err => console.log(err))
```

Promise.reject传入的参数无论是什么形态，都会直接作为reject状态的参数传递到catch的

```js
Promise.reject(Promise.resolve('success'))
  .catch(err => err.then(res => console.log(res))) // => success
```



#### all

它的作用是将多个Promise包裹在一起形成一个新的Promise

新的Promise状态由包裹的所有Promise共同决定

+ 当所有的Promise状态变成fulfilled状态时，新的Promise状态为fulfilled，并且会将所有Promise的返回值组成一个数组
+ 当有一个Promise状态为reject时，新的Promise状态为reject，并且会将第一个reject的返回值作为参数

```js
const p1 = new Promise(resolve => {
  setTimeout(() => resolve('p1 resolved'), 1000)
})

const p2 = new Promise(resolve => {
  setTimeout(() => resolve('p2 resolved'), 2000)
})

const p3 = new Promise(resolve => {
  setTimeout(() => resolve('p3 resolved'), 3000)
})

// 1. Promise.all方法的返回值 也是一个Promise
// 2. Promise.all方法的参数需要是一个可迭代对象
//    如果迭代对象的组成成员并不是promise，那么会使用Promise.resolve对其进行包裹
// 3. Promise.all的resolve的参数 是一个数组
//    且数组中值的顺序和返回的先后顺序无关，之和在 Promise.all中传入的顺序有关
Promise.all([p1, p2, p3]).then(res => console.log('res: ', res))
// => res:  [ 'p1 resolved', 'p2 resolved', 'p3 resolved' ]
```



```js
const p1 = new Promise(resolve => {
  setTimeout(() => resolve('p1 resolved'), 1000)
})

const p2 = new Promise(resolve => {
  setTimeout(() => resolve('p2 resolved'), 2000)
})

const p3 = new Promise(resolve => {
  setTimeout(() => resolve('p3 resolved'), 3000)
})

const promiseSet = new Set([p1, p2, p3])

// Promise.all的参数只要是一个可迭代对象即可，只不过最为常见的参数形式是数组
Promise.all(promiseSet).then(res => console.log('res: ', res))
```



#### allSettled

all方法有一个缺陷: 当有其中一个Promise变成reject状态时，新Promise就会立即变成对应的reject状态

那么对于resolved的，以及依然处于pending状态的Promise，我们是获取不到对应的结果的

于是 在ES11(ES2020)中，添加了新的API Promise.allSettled

+ 该方法会在所有的Promise都有结果(settled)，无论是fulfilled，还是rejected时，才会有最终的状态
+ 并且这个Promise的结果一定是fulfilled的

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p1 resolved'), 1000)
})

const p2 = new Promise(resolve => {
  setTimeout(() => resolve('p2 resolved'), 2000)
})

const p3 = new Promise(resolve => {
  setTimeout(() => resolve('p3 resolved'), 3000)
})

const promiseSet = new Set([p1, p2, p3])

Promise.allSettled(promiseSet).then(res => console.log('res: ', res))
/*
  Promise.allSettled 对于所有的Promise都有对应的结果(settled)
  成功结果对应 status 和 value
  失败结果对应 status 和 reason
  =>
    res:  [
      { status: 'rejected', reason: 'p1 resolved' },
      { status: 'fulfilled', value: 'p2 resolved' },
      { status: 'fulfilled', value: 'p3 resolved' }
    ]
*/
```



#### race

如果有一个Promise有了结果，我们就希望决定最终新Promise的状态，那么可以使用race方法

race是竞技、竞赛的意思，表示多个Promise相互竞争，谁先有结果，那么就使用谁的结果

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p1 resolved'), 1000)
})

const p2 = new Promise(resolve => {
  setTimeout(() => resolve('p2 resolved'), 2000)
})

const p3 = new Promise(resolve => {
  setTimeout(() => resolve('p3 resolved'), 3000)
})

const promiseSet = new Set([p1, p2, p3])

// Promise.race会将第一个状态确定的promise的结果作为返回值进行返回
// 无论这个promise的结果是resolved，还是rejected
Promise.race(promiseSet).then(res => console.log('res: ', res))
.catch(err => console.log(err))
```



#### any

any方法是ES12中新增的方法，和race方法是类似的

any方法会等到一个fulfilled状态，才会决定新Promise的状态

如果所有的Promise都是reject的，那么也会等到所有的Promise都变成rejected状态

如果所有的Promise都是reject的，那么会报一个AggregateError的错误

```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p1 resolved'), 1000)
})

const p2 = new Promise(resolve => {
  setTimeout(() => resolve('p2 resolved'), 2000)
})

const p3 = new Promise(resolve => {
  setTimeout(() => resolve('p3 resolved'), 3000)
})

const promiseSet = new Set([p1, p2, p3])

// any方法会将第一个resolved的promise作为返回值返回
Promise.any(promiseSet).then(res => console.log('res: ', res))
.catch(err => console.log(err)) // => res:  p2 resolved
```



```js
const p1 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p1 rejected'), 1000)
})

const p2 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p2 rejected'), 2000)
})

const p3 = new Promise((resolve, reject) => {
  setTimeout(() => reject('p3 rejected'), 3000)
})

const promiseSet = new Set([p1, p2, p3])

// any方法如果所有的promise的状态全部都是rejected
// 那么会在内部创建一个AggregateError
// 并将对应的结果作为catch方法的参数进行返回
Promise.any(promiseSet).then(res => console.log('res: ', res))
.catch(err => console.log(err)) // => [AggregateError: All promises were rejected]
```

