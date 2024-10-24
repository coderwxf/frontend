## props

通过`super`方法进行初始化

1. 本质是借用构造函数继承 => props会被挂载到组件实例上
2. props依旧是单向数据流  => props是被冻结的 => 只读的 「 解构和属性赋值可以破坏单向数据流 」

```jsx
export default class App extends Component {
  constructor(props) {
    super(props)

    console.log(this.props)
  }

  render() {
    return (
      <>
        This is a class component
      </>
    )
  }
}
```



不通过`super()`初始化props，react内部也会在执行render方法之前对props进行初始化操作

1. `constructor`中无法获取props
2. 其余方法中，只要this是组件实例，就可以正确获取props

```js
import { Component } from 'react'

export default class App extends Component {
  constructor() {
    super()

    console.log(this.props) // undefined
  }

  // render方法中this一定是组件实例
  render() {
    console.log(this.props) // 可以正常获取

    return (
      <>
        This is a class component
      </>
    )
  }
}
```



### 规则校验

```jsx
import { Component } from 'react'
import PropTypes from 'prop-types'

export default class Child extends Component {
  static defaultProps = {
    age: 18
  }

  static propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number
  }

  render() {
    const { name, age } = this.props

    return (
      <>
        <h2>{ name }</h2>
        <h3>{ age }</h3>
      </>
    )
  }
}
```

