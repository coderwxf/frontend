## 作用

父子通信 -- 也就是父组件将数据传递给子组件

从而 提升 组件的 可复用性



## 对象规则设置

### 冻结

```jsx
const obj = {
  name: 'Klaus',
  age: 23
}

// 冻结对象 -- 不可进行任何修改(包裹 增删改 和 修改属性描述符)
Object.freeze(obj)

// 判断对象是否被冻结
console.log(Object.isFrozen(obj)) // true
```



### 密封

```jsx
const obj = {
  name: 'Klaus',
  age: 23
}

// 密封对象 -- 对象属性不能进行添加、删除 也不能修改属性的描述符
Object.seal(obj)

// 判断对象是否被密封
console.log(Object.isSealed(obj)) // true
```



### 扩展

```jsx
const obj = {
  name: 'Klaus',
  age: 23
}

// 禁止扩展 -- 不可添加新属性，其余操作都可以
Object.preventExtensions(obj)

// 判断对象是否被阻止扩展
// 被阻止扩展，isExtensible 返回 false
console.log(Object.isExtensible(obj)) // false
```



1. 对象一旦冻结，则该对象即是密封的，也是无法扩展的
2. 对象一旦被密封，则该对象也是无法扩展的
3. react推荐使用单向数据流，所以props被冻结了，也就是说无法通过子组件来修改父组件传入的props



`父组件`

```jsx
<App className="app"> App </App>
```

`子组件`

```jsx
export default function App(props) {
  return (
    // react 不支持 className='child {props.className}'
    // 也就是不支持插值和普通字符串共用，需要自己进行字符串拼接
    <span className={`child ${props.className}`}>
      this is a function component
    </span>
  )
}
```



## props 规则校验

```jsx
// 对props的类型和是否必传 需要使用react的官方库 prop-types
import PropTypes from 'prop-types'

function App(props) {
  console.log(props)

  return (
   <div>child</div>
  )
}

// react对 props进行规则校验和设置默认值 使用的是 静态属性

// 设置默认值
App.defaultProps = {
  age: 0 // 当且仅当props的值是undefined的时候，才会使用默认值
}

// 对props类型和是否必传 进行校验
App.protoTypes = {
  name: PropTypes.string.isRequired,
  // 两个类型 二选一
  age: PropTypes.oneOfType([
    PropTypes.number,
    PropTypes.string
  ])
}

export default App
```

1. 规则校验错误 抛出的是警告 不是错误

   例如: 必传props A 没有传递对应值，控制台会抛出警告，

   但是对应A的值是undefined，而不是因为错误终止程序执行

2. 将`prop-types`单独抽取为一个库，是为了符合渐进式特征

   可以使用官方库，也可以使用第三方库，甚至可以自己实现对应功能

3. 更多的校验规则 --- [prop-types](https://www.npmjs.com/package/prop-types)



### 强制修改props

强制修改props会破坏单向数据流，是不被推荐的

但是依旧可以实现，因为冻结的是对象props

所以只要将prop值赋值给某个变量，就可以绕过对象props，也就绕过了冻结规则



`假设props原本的值是{ title: 'Hello' }`

修改props方式一 --- 直接修改props

```js
props.title = 'bye' // error --- props 已经被冻结了
```



修改props方式二 --- 将props值赋值给变量后再进行修改 

```jsx
const { title } = props
title = 'bye' // success  结构相当于 let title = props.title 
// 此时修改值，修改的是title变量，而不是props对象 --- 不会报错

console.log(title) // => bye
console.log(props.title) // => hello
```

