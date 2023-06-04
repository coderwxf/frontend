在日常开发，我们经常需要发生一个异步请求后，拿到异步请求的结果以后，在通过这个结果再去另一个接口中进行查找对应的值

```js
// 需求: 
// 1> url: exmaple -> res: example
// 2> url: res + "aaa" -> res: example-aaa
// 3> url: res + "bbb" => res: exmaple-aaa-bbb
```



`解决方法一 - 多次调用`

这种解决方案是最简单也是最直接的，但是如果我们需要的异步请求过多的时候，就很容易出现下图的`回调地狱`，这对于后期的维护和扩展是非常不利的

```js
// 使用定时器模拟请求
function request(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve(url)
      } else {
        reject('url is not defined')
      }
    }, 1000)
  })
}

request('example').then(res => {
  request(`${res}-aaa`).then(res => {
    request(`${res}-bbb`).then(res => {
      console.log(res)
    })
  })
})
```

![I3XnBu.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71303ea8ff4d40efac2f43da74f0f53f~tplv-k3u1fbpfcp-zoom-1.image) 



`解决方法二 - Promise中then的返回值`

```js
// 使用定时器模拟请求
function request(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve(url)
      } else {
        reject('url is not defined')
      }
    }, 1000)
  })
}

// 这种解决方式是将回调函数的嵌套变为了同层调用
// 但是依旧没有从根本上解决多次回调产生的不利于维护和扩展的问题
request('example').then(res => request(`${res}-aaa`))
.then(res => request(`${res}-bbb`))
.then(res => console.log(res))
```



`解决方法三 - Promise + generator`

这种解决方式可以使用同步的方式去编写异步代码

在`async + await`出现之前，这是最好的解决方式

```js
function asyncFn(res) {
  return new Promise(resolve => {
    setTimeout(() => resolve(res), 2000)
  })
}

// 利用生成器函数来处理异步方法
function* fetchData() {
  const res1 = yield asyncFn(10)
  const res2 = yield asyncFn(20)
  const res3 = yield asyncFn(30)

  console.log(res1 + res2 + res3) // => 60
}

const generator = fetchData()

// generator.next().value 就是asyncFn 返回的那个promise对象
// 发送三次网络请求，就有三个yield，也就对应着三个next方法
generator.next().value
  .then(res1 => generator.next(res1).value)
  .then(res2 => generator.next(res2).value)
  .then(res3 => generator.next(res3).value)
  .catch(err => console.error(err))
```

上述代码的生成器执行是重复的逻辑，所以我们可以将上述生成器执行过程，封装成一个自动化执行函数

```js
// 使用定时器模拟请求
function request(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve(url)
      } else {
        reject('url is not defined')
      }
    }, 1000)
  })
}

function* fetchData(params) {
  let res = yield request(params)
  res = yield request(`${res}-aaa`)
  res = yield request(`${res}-bbb`)
  return res
}

// 执行generator函数
function execGenerator(generatorFn, params) {
  // 为了让外部函数调用者，可以获取到异步函数执行的结果, 所以需要通过promise进行返回
  return new Promise(resolve => {
    const generator = generatorFn(params)

    function exec(res) {
      const { done, value: promise } = generator.next(res)

      if (done) {
        return resolve(promise)
      }

      promise.then(res => {
        exec(res)
      })
    }

    exec()
  })
}

execGenerator(fetchData, 'example').then(res => console.log(res)) 
// => example-aaa-bbb
```



在实际使用的时候，存在第三方库`co`可以主动帮助我们来执行generator函数，而不需要我们手动编写对应的`execGenerator`

```shell
# 安装
$ npm i co
```

```js
// 使用定时器模拟请求
function request(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve(url)
      } else {
        reject('url is not defined')
      }
    }, 1000)
  })
}

function* fetchData() {
  let res = yield request('example')
  res = yield request(`${res}-aaa`)
  res = yield request(`${res}-bbb`)
  console.log(res)
}

const co = require('co')

co(fetchData)
```



`async + await`

`async+await`是ES8中引入的关键字，是目前实际开发中，实际用来解决异步处理请求的方式

`async+await`在进行解析的时候，依旧会被转换为`Promise+Generator`的方式

`async+await`仅仅只是`Promise+Generator`的一种语法糖形式

```js
function request(url) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (url) {
        resolve(url)
      } else {
        reject('url is not defined')
      }
    }, 1000)
  })
}

async function fetchData() {
  let res = await request('example')
  res = await request(`${res}-aaa`)
  res = await request(`${res}-bbb`)
  console.log(res)
}


fetchData()
```



### 基本使用

async关键字用于声明一个异步函数

```js
async function foo() {}

const bar = async() => {}

class Person {
  async baz() {   
  }
}
```



### 执行流程

异步函数的内部代码执行过程和普通的函数是一致的，默认情况下也是会被同步执行

```js
async function foo() {
  console.log('foo中的脚本')
}

console.log('script started')
foo()
console.log('script endedd')

/*
  =>
    script started
    foo中的脚本
    script endedd
*/
```



`异步函数和普通函数的区别`

`区别一: ` 异步函数也可以有返回值，但是异步函数的返回值会被包裹到Promise.resolve中

```js
// 就算异步函数中声明都不写
// 默认的返回值也是一个promise，只不过是Promise.resolve(undefined)
async function foo() {
}

const promise = foo()
promise.then(res => console.log('异步函数返回值：', res))
```

```js
async function foo() {
  // 如果异步函数的返回值是一个thenable对象或者promise对象
  // 那么会自动执行返回的thenable对象或者promise对象以便于获取最终的状态
  return {
    then(resolve, reject) {
      resolve(123)
    }
  }
}

const promise = foo()
promise.then(res => console.log(res)) // => 123
```



`区别二:` 异步函数中抛出了异常，那么程序它并不会像普通函数一样报错，而是会作为Promise的reject来传递

```js
async function foo() {
  throw new Error('something error in async function')
}

const promise = foo()
promise.catch(e => console.log(e))
```



`区别三:` 可以在异步函数内部使用await关键字，而在普通函数中是不可以的

```js
function request() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(123)
    }, 1000)
  })
}

async function foo() {
  // await是后面会跟上一个表达式，这个表达式会返回一个Promise 
  // 当await后边返回的promise的状态转变为resolved的时候
  // 其结果就会作为await表达式的返回值的值赋值给res
  const res = await request()

  // await表达式后面的逻辑会等待await表达式对应的异步代码执行完毕以后再进行执行
  // 可以认为await表达式后边的逻辑代码其实就是需要写在await表达式返回的promise的then方法中的逻辑代码
  console.log(res)
}

foo()
```

```js
// 如果await后面是一个普通的值，那么会直接返回这个值
function request() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      resolve(123)
    }, 1000)
  })
}

async function foo() {
  // 如果await后边跟上的是一个基本数据类型
  // 那么就等价于 await Promise.resolve(123)
  const res = await 123

  console.log(res) // => 123
}

foo()
```


```js
function request() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(123)
    }, 1000)
  })
}

async function foo() {
  try {
    // request执行失败，对应的错误会被抛出
    // 1. 可以在异步函数中使用 try-catch进行捕获
    // 2. 如果异步函数中没有捕获，因为异步函数返回一个状态为reject的promise
    //    所以可以在foo().catch() 中进行捕获
    // 3. 如果依旧没有捕获对应的异常，会一层层向上传递
    //    最终交给浏览器进行处理，浏览器会将对应的错误输出在console中  
    const res = await request()
    console.log(res)
  } catch(err) {
    console.log('err', err)
  }
}

foo()
```



### await的编写位置

await关键字只能使用在async函数内部，或ES模块的顶层

`使用在async函数内部`

```js
async function main() {
  const asyncMsg = Promise.resolve('hello world!')
  console.log(asyncMsg) // => hello world!
}

main()
```



`在ES模块顶层使用`

因为顶层await只能使用在ES模块中，所以需要进行特殊设置

`方式一: 在package.json中设置type: module`
`方式二: 将需要开启顶层await的模块的文件后缀名设置为mjs`

`index.mjs`

```js
const asyncMsg = await Promise.resolve('hello world!')
console.log(asyncMsg) // "hello world!"
```

