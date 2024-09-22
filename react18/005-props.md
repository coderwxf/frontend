props作用是 通过 父传子 实现父子组件通信·

props需要遵循单向数据流 => react中的props是只读属性 => props对象被冻结了

注意: 解构会破坏对象规则设置 「 例如: 丢失响应式，丢失了冻结特性 」

```js
const obj = { name: 'Klaus', age: 23 }

const frozenObj = Object.freeze(obj)

frozenObj.age++ // error

// 本质是用新变量接受属性值，不再是操作frozenObj => 因此丢失了冻结特性
let { age } = frozenObj // 等价于 const age = frozenObj.age
age++
console.log(age) // 24
```



## 规则校验

通过[官方库`prop-types`](https://www.npmjs.com/package/prop-types)来实现规则的校验

 「 `prop-types`本来内置于react中，后来为了更好的渐进式抽离成了单独的库 」



不管是默认值 还是 规则校验 都是通过 设置静态属性和方法来完成的



### 设置默认值

当props值严格等于`undefined`时，才会触发默认值选项

```js
<!-- 函数组件 -->
Child.defaultProps = {
  age: 18
}
```



### 规则校验

规则校验主要校对: 1. 数据类型格式 2. 是否是必传值

props的规则校验无论是否成功，都会成功传递给子组件，只不过控制台会报错，但不影响props错误

```jsx
// 函数组件

// PropTypes 之所以首字母大写，因为导入的是工具对象
import PropTypes from 'prop-types'

// 规则校验使用的是全小写，因为是普通对象
Child.propTypes = {
  // age需要是 number类型值 且是必传项
  // 一般必传项，没有必要设置默认值 => 触发不了
  age: PropTypes.number.isRequired.
  node: PropTypes.node, // 任何可以被JSX渲染的内容
  child: PropTypes.elemebt, // JSX元素
  sex: PropTypes.oneOf(['male', 'female']), // 多个值中的任意一个
  msg: PropTypes.oneOfType([ // 多个类型中任意一个类型
    PropTypes.func, // 函数类型 
    PropTypes.bool, // 布尔类型
    PropTypes.instanceOf(Message) // 传入的必须是Message的实例 「 限制传递的组件类型 」
  ])
}
```



## children 

children的值可能存在三种情况

1. 没有子元素 => undefined
2. 一个子元素 => 一个值
3. 多个子元素 => 子元素构成的数组



 React上有一个工具对象Children 

​	=> 上面包含了一系列用于操作props.children的方法，用于屏蔽children可能的三种情况下差异



```js
// toArray => 不管传入的props.children是什么类型，都统一转换为数组
// React.Children.toArray 只会处理合法的JSX子元素
// 如果传递了不合法的JSX子元素
// 1. 如果是函数 自动忽略并跳过
// 2. 如果是其它类型值，优先调用String方法转换为字符串后，再传入
const child = React.Children.toArray(children)

React.Children.count(children) // props.children的个数

// 对Children进行迭代或映射
// => 内部在循环之前会自己调用React.Children.toArray方法保证props.children一定是数组
React.Children.forEach(children, (item, index) => console.log(item, index))
React.Children.map(children, (item, index) => item)

// 如果props.chilren 只有一个元素且该元素是JSX元素 => 直接返回该元素
// 其余情况，一律报错
console.log(React.Children.only(children))
```



## slot

插槽(slot) 是一种特殊的props。是值为JSX元素的props

React采用的是深度优先解析规则，所以在执行父组件时，子组件已经全部都解析完毕了。

这也就意味着传入的`props.children`都是解析后的JSX对象，而不是HTML格式的JSX代码片段。

```jsx
<>
  <Child>
    {/* slot是自定义属性 => 用于标识各个JSX元素，进行区分 => 具名插槽  */}
    <h2 className="header" slot="header">Child Header</h2>
    <h2 className="footer" slot="footer">Child Header</h2>
  </ Child>

  <Child />
</>
```

```jsx
import React from 'react'

export default function Child(props) {
  const { children } = props

  const child = React.Children.toArray(children)

  // 获取slot 并设置 默认值
  const headerSlot = child.find(v => v.props.slot ==='header') 
  ?? <h2>default Header</h2>
  
  const footerSlot = child.find(v => v.props.slot ==='footer') 
  ?? <h2>default Footer</h2>

  return (
    <>
      { headerSlot }
      Children Content
      {footerSlot}
    </>
  )
}
```



具名插槽的另一种实现方式

```jsx
<>
 <Child
  headerSlot={ <h2>Header Slot</h2> }
  footerSlot={ <h2>Footer Slot</h2> }
 />

  <Child />
</>
```

```jsx
import React from 'react'

export default function Child(props) {
  const {
    headerSlot = <h2>default Header</h2>,
    footerSlot = <h2>default Footer</h2>
  } = props

  return (
    <>
      { headerSlot }
      Children Content
      {footerSlot}
    </>
  )
}
```



### 插槽作用域

```jsx
<>
  <Child>
    { headerMsg =>  <h2 slot="header">{headerMsg}</h2>}
    { footerMsg =>  <h2 slot="footer">{footerMsg}</h2>}
  </ Child>

  <Child />
</>
```

```jsx
import React from 'react'

export default function Child(props) {
  const { children } = props

  let headerMsg = 'Child Header'
  let footerMsg = 'Child Footer'

  // React.Children.toArray 在处理时，会自动过滤函数，所以在这里需要手动处理
  const child = children ? Array.isArray(children) ? [...children] : [children] : []

  // 获取slot 并设置 默认值
  const headerSlot = child.find(v => typeof v === 'function' && v().props.slot ==='header')
  ?? (() => <h2>default Header</h2>)

  const footerSlot = child.find(v => typeof v === 'function' && v().props.slot ==='footer')
  ?? (() => <h2>default Footer</h2>)

  return (
    <>
      { headerSlot(headerMsg) }
      Children Content
      {footerSlot(footerMsg)}
    </>
  )
}
```

另一种实现方式

```jsx
<>
  <Child
    headerSlot={ headerMsg => <h2>{ headerMsg }</h2> }
    footerSlot={ footerMsg => <h2>{ footerMsg }</h2> }
  />

  <Child />
</>
```

```jsx
import React from 'react'

export default function Child(props) {
  const {
    headerSlot = () => <h2>default Header</h2>,
    footerSlot = () => <h2>default Footer</h2>
  } = props

  let headerMsg = 'Child Header'
  let footerMsg = 'Child Footer'

  return (
    <>
      { headerSlot(headerMsg) }
      Children Content
      {footerSlot(footerMsg)}
    </>
  )
}
```

