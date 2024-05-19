父组件向子组件传递数据，以增强组件复用性

对props的类型和是否必传 需要使用react的官方库 [prop-types](https://www.npmjs.com/package/prop-types)

```jsx
//  PropTypes大写 是因为 PropTypes是一个工具对象
import PropTypes from 'prop-types'

function App(props) {
  console.log(props)

  return (
   <div>child</div>
  )
}

// react对 props进行规则校验和设置默认值 使用的是 静态属性

// 设置默认值 --- 当prop是undefined时，才会触发默认值
App.defaultProps = {
  age: 0 
}

// 对props类型和是否必传 进行校验
App.protoTypes = {
  name: PropTypes.string.isRequired,
  // 两个类型 二选一
  age: PropTypes.oneOfType([
    PropTypes.number,
    PropTypes.string
  ]),
  // PropTypes.node --- 任何可以被react渲染的东西
  children: PropTypes.node
}

export default App
```

1. props都是只读的，因为props被frozen了
2. 如果存在规则校验，会在传值之前优先进行校验
   + 规则校对失败，控制台抛出警告
   + 不管校验成功还是失败， props都是正常传递的



## 强制修改props

强制修改props会破坏单向数据流，是不被推荐的，但是依旧可以实现，因为冻结的是对象props

所以只要将props值赋值给某个变量或对props进行解构操作，就可以绕过对象props，也就绕过了冻结规则



假设props原本的值是{ title: 'Hello' }

修改props方式一 --- 直接修改props

```js
props.title = 'bye' // error --- props 已经被冻结了
```



修改props方式二 --- 将props值赋值给变量后再进行修改

```js
const { title } = props //  解构相当于 let title = props.title 
title = 'bye' // success 
// 此时修改值，修改的是title变量，而不是props对象 --- 不会报错

console.log(title) // => bye
console.log(props.title) // => hello
```



更常见的做法是 将 props 的值赋给组件的 state ，然后修改 state

也就是接收props，然后将props转换为组件内部状态，虽然去修改和使用转换后的state
