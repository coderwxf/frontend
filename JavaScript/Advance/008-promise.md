Promise 是一种用于获取异步任务执行结果的机制。

在 Promise 出现之前，常用的方式是通过回调函数来处理异步操作，例如：

```js
function asyncFn(count, resolve, reject) {
  if (count <= 0) {
    reject('count不能小于0')
  } else {
    setTimeout(() => {
      resolve(count)
    }, 1000)
  }
}

asyncFn(
  100, 
  count => console.log(count), 
  err => { throw new Error(err) }
)
```

这种通过回调函数的方式没有统一的规范，导致代码可读性差、难以维护

虽然社区早期提出了 [Promises/A+](https://promisesaplus.com/) 规范来标准化 Promise 的行为，但它并不是强制的，开发者依然可以实现自己的版本。

为了解决这些问题，JavaScript 在 ES6（ECMAScript 2015）中引入了原生的 `Promise` 对象，以标准化和简化异步操作的处理



![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f70b1957d36443bfbc41a67416080cf2~tplv-k3u1fbpfcp-zoom-1.image)

Promise的状态是单向的，一旦改变就不可修改

Promise创建时，需要传入一个executor回调，该回调会被同步执行



### resolve 不同值的处理方式

1. **普通值**：如果 `resolve` 传入的是一个普通的值（包括基本数据类型和引用数据类型），该值将直接作为 `then` 回调的参数。
2. **Promise**：如果 `resolve` 传入的是另一个 Promise，那么这个新 Promise 的状态将决定原 Promise 的状态。
3. **thenable 对象**：如果 `resolve` 传入的是一个具有 `then` 方法的对象（这种对象称为 `thenable` 对象），则会调用该对象的 `then` 方法，并且根据 `then` 方法的结果来决定原 Promise 的状态。

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



## reject不同值

`reject` 方法会将 Promise 的状态直接设置为 `rejected`，并将传入的参数作为拒绝（rejection）的原因，通常用于表示错误信息。**重要的是，`reject` 不会对传入的参数进行任何形式的转换**。



## 实例方法

### then

`then(onResolved, onRejected)`方法接受两个参数：`onResolved` 和 `onRejected`。

这两个参数是可选的:

- `onResolved`: 当 Promise 处于 `fulfilled` 状态时调用。
- `onRejected`: 当 Promise 处于 `rejected` 状态时调用。

```js
asyncFn().then(res => console.log(res), err => console.log(err))
```




then 可以被多次调用，对应 resolve 方法和 reject 方法会被加入数组，在 promise 状态改变时，被依次回调。

```js
asyncFn().then(() => console.log('第一次调用'))
asyncFn().then(() => console.log('第二次调用'))
asyncFn().then(() => console.log('第三次调用'))
asyncFn().then(() => console.log('第四次调用'))
asyncFn().then(() => console.log('第五次调用'))
```



每次调用 `then` 都会返回一个新的 Promise 对象，这样就支持链式调用

新Promise的状态由then方法的返回值决定

+ 当then方法中的回调函数本身在执行的时候，那么它处于pending状态

+ 当then方法中的回调函数返回一个结果时:
  + 返回一个普通的值, 那么它处于fulfilled状态，并且会将结果作为resolve的参数
  + 返回一个Promise, 当前promise的状态由返回的这个新promise来决定
  + 返回一个thenable值，当前promise的状态由thenable对象的then方法执行结果决定
+ 当then方法抛出一个异常时，那么它处于reject状态

```js
function asyncFn() {
  return new Promise(resolve => resolve('success'))
}

asyncFn()
  .then(res => console.log(res))
  .then(res => {
		console.log(res)
  	return 'then方法返回的结果'
	})
  .then(res => console.log(res))
/*
  =>
    success
    undefined
    then方法返回的结果
*/
```



### catch

 当promise抛出异常的时候，会被离其最近的catch方法所捕获

```js
function asyncFn() {
  return new Promise(resolve => reject('success'))
}

asyncFn().then(res => console.log(res)).catch(err => console.log(err))
```



catch可以被多次回调，多个回调会被加入数组，并被依次调用

```js
function asyncFn() {
  return new Promise(resolve => reject('error'))
}

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



catch的返回值也是Promise, 以便链式调用。

后续Promise的状态判断规则和then方法完全一致

```js
function asyncFn() {
  return new Promise((resolve, reject) => {
    reject('error')
  })
}

asyncFn()
  .then(res =>  console.log(res))
  .catch(err => console.log(err)) // catch方法返回的也是一个Promsie
  .then(res => console.log(res))
/*
  =>
    error
    undefined
*/
```



promise抛出的异常会被传递到下一个promise中, 如果下一个promise也没有处理，那么这个异常会一直向下进行传递. 直到被catch方法所捕获，或者没有catch处理的时候，最终会抛给浏览器

```js
function asyncFn() {
  return new Promise((_, reject) => {
    reject('error')
  })
}

asyncFn()
  .then(res => console.log(res))
	.then(res => console.log(res))
  .then(res => console.log(res))
	.catch(err => console.log(err)) // => error
```

```js
function asyncFn() {
  return new Promise((_, reject) => reject('error'))
}

// 错误只能在自己promise链中传递，因此在实际书写的时候，每一个promise都应该加上对应的catch方法
asyncFn().then(res => console.log(res)) // error

asyncFn().then(res => console.log(res)).catch(err => console.log(err))
```



### finally

无论Promise对象无论变成fulfilled还是rejected状态，最终都会被执行的代码

finally方法是不接收参数的

finally本身返回的也是一个promise，所以从理论角度来讲，可以一直链式下去

```js
function asyncFn() {
  return new Promise((resovle, reject) => resovle('success'))
}

asyncFn()
  .then(res => console.log(res))
  .catch(err => console.log(err))
  .finally(() => console.log('finally'))
```



## 静态方法

### resolve

Promise.resolve的参数可以是普通值，也可以是一个新的promise对象或者是一个thenable对象, 对应的规则和之前是一致的

返回值是Promise

```js
Promise.resolve('resolve msg').then(res => console.log(res));

// 上述代码等价于
new Promise(resolve => resolve('resolve msg')))
  	.then(res => console.log(res);
```



### reject

返回值是Promise

```js
Promise.reject('error msg').catch(err => console.log(err))

// 等价于
!(new Promise((_, reject) => reject('error msg'))).catch(err => console.log(err))
```

Promise.reject传入的参数无论是什么形态，都会直接作为reject状态的参数传递到catch的

```js
Promise
  .reject(Promise.resolve('success'))
  .catch(err => err.then(res => console.log(res))) // => success
```



### all

`Promise.all` 方法接收一个可迭代对象作为参数，并返回一个新的 Promise。这个新的 Promise 在以下情况下会被处理：

1. **成功**：当所有传入的 Promise 都成功时，返回的 Promise 会被解决（resolved），并且解析值将是一个包含所有传入 Promise 解析值的数组。数组中元素的顺序与传入的顺序相同。

2. **失败**：如果传入的任何一个 Promise 被拒绝（rejected），返回的 Promise 也会被拒绝，并且拒绝原因是第一个被拒绝的 Promise 的拒绝原因。

> 如果Promise的静态方法接受多个promise，则
>
> 1. 参数是由promise组成的可迭代对象 「 如果任何一个数据项不是promise，会通过`Promise.resolve`方法转换为promise实例 」

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

Promise.all([p1, p2, p3]).then(res => console.log('res: ', res))
// => res:  [ 'p1 resolved', 'p2 resolved', 'p3 resolved' ]
```



### allSettled

`Promise.allSettled` 方法会等待所有的 Promise 都 settled（即都已经 fulfilled 或 rejected），然后返回一个新的 Promise，这个新的 Promise 的结果是一个数组，其中每个元素对应一个输入的 Promise 的结果对象，并且数组中元素的顺序与传入的顺序相同。

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
  =>
    res:  [
      { status: 'rejected', reason: 'p1 resolved' }, // 失败结果
      { status: 'fulfilled', value: 'p2 resolved' }, // 成功结果
      { status: 'fulfilled', value: 'p3 resolved' }
    ]
*/
```



### race

`Promise.race` 方法会返回一个新的 Promise，一旦迭代对象中的任何一个 Promise 解决或拒绝，`Promise.race` 返回的 Promise 就会解决或拒绝，无论其他 Promise 还处于待定状态。

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

Promise.race(promiseSet)
  .then(res => console.log('res: ', res))
	.catch(err => console.log(err))
```



### any

`Promise.any` 方法会接受一个可迭代对象（例如数组或 Set），其中包含多个 Promise。

它会返回一个新的 Promise

+ 该 Promise 在其中任何一个传入的 Promise 变为 fulfilled 状态时就会立即变为 fulfilled，并返回该 Promise 的值。
+ 如果所有传入的 Promise 都变为 rejected 状态，那么返回的 Promise 会变为 rejected，并返回一个 `AggregateError` 错误对象，该对象包含所有被拒绝的错误。

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
Promise.any(promiseSet)
  .then(res => console.log('res: ', res))
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

Promise.any(promiseSet)
  .then(res => console.log('res: ', res))
  .catch(err => {
    console.log(err)
    console.log(err.errors) // [ 'p1 rejected', 'p2 rejected', 'p3 rejected' ]
  })
```

