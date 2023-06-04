## readux hook

在之前的redux开发中，为了让组件和redux结合起来，我们使用了react-redux中的connect

但是这种方式必须使用高阶函数结合返回的高阶组件, 并且必须编写:mapStateToProps和 mapDispatchToProps映射的函数



而在redux7.1中，提供了对应的hook写法

```jsx
import React, { memo } from 'react'
import { useSelector, useDispatch } from 'react-redux'
import { addNumAction } from './store/slices/counter'

const App = memo(() => {
  // useSelector的作用和 mapStateToProps的功能是一致的
  const { count } = useSelector(state => ({ count: state.counter.count }))

  // 通过调用useDispatch方法 可以直接获取dispatch方法
  const dispatch = useDispatch()

  return (
    <div>
      <div>{ count }</div>
      <button onClick={ () => dispatch(addNumAction(1)) }>+1</button>
    </div>
  )
})

export default App
```



### redux hook的重复渲染问题

```jsx
import React, { memo } from 'react'
import { useSelector, useDispatch } from 'react-redux'
import { addNumAction } from './store/slices/counter'

// 此时 会发现在修改count的时候，Child组件也发生了重新渲染
// 因为默认情况下，只要useSelector使用了某一个slice
// 那么当对应slice中的任意一个状态发生改变的时候
// 所以引入该slice的组件都需要被重新渲染
const Child = memo(() => {
  const { message } = useSelector(state => ({ message: state.counter.message }))

  console.log('child render')

  return (
    <div>{ message }</div>
  )
})

const App = memo(() => {
  const { count } = useSelector(state => ({ count: state.counter.count }))
  const dispatch = useDispatch()

  console.log('app render')

  return (
    <div>
      <div>{ count }</div>
      <button onClick={ () => dispatch(addNumAction(1)) }>+1</button>
      <Child />
    </div>
  )
})

export default App
```



```jsx
import React, { memo } from 'react'
import { useSelector, useDispatch, shallowEqual } from 'react-redux'
import { addNumAction } from './store/slices/counter'

const Child = memo(() => {
  // 为了解决重复渲染问题，useSelector可以传入第二个参数
  // 该参数的值的类型是一个函数
  // 在函数被回调的时候，会将上一次的第一个参数返回的对象和这一次的第一个参数返回的对象作为参数一并传入
  // 在本例中就会将旧的{message: xxx} 和 新的 {message：xxx}作为参数传入给useSelector的第二个参数
  // 第二个参数返回一个boolean类型的值，如果是true则返回新的message，否则则什么都不操作
  // 此时我们就可以通过对其进行浅层比较来判断是否需要更新组件
  // 而react-redux导出了实现浅层比较的方法shallowEqual
  // 所以在绝大多数情况下，直接将shallowEqual作为useSelector的第二个参数即可
  const { message } = useSelector(state => ({ message: state.counter.message }), shallowEqual)

  console.log('child render')

  return (
    <div>{ message }</div>
  )
})

const App = memo(() => {
  const { count } = useSelector(state => ({ count: state.counter.count }), shallowEqual)
  const dispatch = useDispatch()

  console.log('app render')

  return (
    <div>
      <div>{ count }</div>
      <button onClick={ () => dispatch(addNumAction(1)) }>+1</button>
      <Child />
    </div>
  )
})

export default App
```



## react18 hook

### useId

#### csr vs ssr

SSR(Server Side Rendering，服务端渲染)，指的是页面在服务器端已经生成了完成的HTML页面结构，不需要浏览器通过执行对应的JS代码来创建对应的页面结构

对应的是CSR(Client Side Rendering，客户端渲染), 也就是浏览器请求到html后，再由浏览器去请求和加载其它资源，整个的渲染是在客户端完成的



对于CSR页面，其页面中，往往仅有对于的meta标签和div#root或div#app这类简单的html元素，并没有完整的html类型

```html
<!DOCTYPE html>
<!-- React CSR app的基本html页面结构-->
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <title>React App</title>
  </head>
  <body>
    <div id="root"></div>
  </body>
</html>
```

对应的内容是等到浏览器加载到html页面后，再根据html页面中的script标签等再去加载对应的js脚本，随后再由浏览器去执行这些js脚本，最终在浏览器上渲染出最终的页面

所以对于CSR页面存在两个问题

1. 首屏渲染速度比SSR慢

   因为CSR的渲染都是在客户端执行的，而SSR在服务器端已经生成了对应的html结构，客户端直接加载执行即可

   所以对于SSR而言，CSR的渲染速度较慢

2. 不利于SEO优化

   虽然googlebot在抓取页面的时候，不仅仅会抓取页面的html文件，也会去加载和解析页面的css文件和js脚本文件

   但是在国内百度爬虫去进行页面抓取的时候(不考虑收费的情况下)，百度的爬虫引擎仅会去抓取html文件，并不会去

   加载和解析对应的js文件和css文件

   这也就意味着，对应CSR项目而言， 百度爬虫只能识别到没有任何渲染内容的html, 并没有办法有效的去抓取CSR通过js脚本渲染出来的内容

   对于SSR页面而言，因为整个的页面内容在百度爬虫抓取的时候，已经被服务器渲染成了完整的HTML页面结构

   此时爬虫就可以根据网页的内容建立对应的索引，在用户通过关键字索引的时候，可以更好的匹配和呈现对应的网页内容

   所以SSR相比于CSR，更有利于SEO

所以对于不需要在外部传播的页面，如后台管理系统，推荐是CSR

对于需要在外部传播的页面，如门户网站，推荐使用SSR

![image.png](https://s2.loli.net/2022/10/29/KtHMYnv3UOc7Zyz.png)



#### 同构应用

一套代码既可以在服务端运行又可以在客户端运行，这就是同构应用

当用户发出请求时，先在服务器通过SSR渲染出首页的内容， 但是对应的代码同样可以在客户端被执行

目的是因为node返回给浏览器的html本质上仅仅只是对应的网页结构，实质上仅仅只是一串字符串

这也就意味着这个网页是没有任何交互的，所以需要将对应的代码在浏览器端在运行一遍，以给页面绑定上对应的js代码

从而给浏览器提供对应的交互功能和对应的交互逻辑

![image.png](https://s2.loli.net/2022/10/29/9FfcvE7sLyKVgTr.png)



而在这个过程中，通过在浏览器端执行对应的js脚本给页面添加交互功能的过程就被称之为**hydration**



useId 是一个用于生成横跨服务端和客户端的稳定的唯一 ID 的同时避免 hydration 不匹配的 hook

useId可以保证应用程序在客户端和服务器端生成唯一的ID，这样可以有效的避免通过一些手段生成的id不一致，造成 hydration mismatch

```jsx
// 这个id在ssr渲染的时候和hydration的时候生成的一定是相同的，且唯一的
// 如果客户端的id和服务器端生成的id不一致会导致hydration mismatch错误
// 所以我们可以使用useId，在一个组件中只能生成一个id，这个id无论渲染几次还是在服务端渲染还是客户端渲染 对应的id值都是不变的
// 因为对应的id是根据组件树对应的结构来生成的，因为是同构应用所以对应的组件树是一致的，因此对应生成的id也是一致的
// 如果需要在一个组件中使用不同的id，可以在useId()生成的id基础上添加对应的前缀或后缀
const id = useId()
```



### useTransition

告诉react对于某部分任务的更新优先级较低，可以稍后进行更新

`fake.js`

```js
// 生成mock数据
import { faker } from '@faker-js/faker';

const users = Array(10000).fill(undefined).map(() => {
  return faker.helpers.unique(faker.name.fullName);
})

export default users
```

`App.jsx`

```jsx
import React, { memo, useState } from 'react'
import fakeUsers from './fake'

const App = memo(() => {
  const [users, setUsers] = useState(fakeUsers)

  function handleInput(e) {
    const keyword = e.target.value

    const filterUsers = fakeUsers.filter(user => user.includes(keyword))

    setUsers(filterUsers)
  }

  return (
    <>
      <input type="text" onInput={handleInput} />
      <ul>
        {
          users.map(user => <li key={user}>{ user }</li>)
        }
      </ul>
    </>
  )
})

export default App
```



此时我们将浏览器开发工具中的`preference`中的cpu调至四倍慢速后，再在页面中进行测试

我们会发现在用户输入后，需要等待一段时间后，界面才会做出响应(包括过滤后的数据和输入框中的数据)

这是因为输入框中的数据改变本质上触发的是handleInput，而数据过滤也是在handleInput中被执行

所以只有等到数据过滤完毕后，输入框的input事件才会被执行完毕，对应的内容也才会被修改



但是这是非常不合理的，我们希望的是input中的数据在修改后即可进行更新，界面可以加上loading动画等待数据出现后再进行显示

```jsx
import React, { memo, useState, useTransition } from 'react'
import fakeUsers from './fake'

const App = memo(() => {
  const [users, setUsers] = useState(fakeUsers)
  const [isLoading, setTransition] = useTransition()

  function handleInput(e) {
    // setTransition中的代码会等到空闲的时候再去执行其中的代码
    setTransition(() => {
      const keyword = e.target.value

      const filterUsers = fakeUsers.filter(user => user.includes(keyword))

      setUsers(filterUsers)
    })
  }

  return (
    <>
      <input type="text" onInput={handleInput} />
      { isLoading && <h1>loading ...</h1> }
      <ul>
        {
          users.map(user => <li key={user}>{ user }</li>)
        }
      </ul>
    </>
  )
})

export default App
```



### useDeferredValue

useDeferredValue 接受一个值，并返回该值的新副本，该副本将推迟到更紧急地更新之后

也就是说useDeferredValue的功能和setTransition所对应的功能是类似的

```jsx
import React, { memo, useState, useDeferredValue } from 'react'
import fakeUsers from './fake'

const App = memo(() => {
  const [users, setUsers] = useState(fakeUsers)
  const deferUsers = useDeferredValue(users)

  function handleInput(e) {
    // setTransition中的代码会等到空闲的时候再去执行其中的代码
    const keyword = e.target.value

    const filterUsers = fakeUsers.filter(user => user.includes(keyword))

    setUsers(filterUsers)
  }

  return (
    <>
      <input type="text" onInput={handleInput} />
      <ul>
        {
          deferUsers.map(user => <li key={user}>{ user }</li>)
        }
      </ul>
    </>
  )
})

export default App
```


