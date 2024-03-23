## 基本使用

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  // 将props传递给父组件进行初始化操作
  constructor(props) {
    super(props)
  }

  render() {
    return (
      <>
        this is a class component
      </>
    )
  }
}
```



注意:

即使我们没有通过super方法，将props传递给父组件，react依旧会自己完成props的初始化操作

目的是方便我们编写时，省略构造函数 「 默认继承自带的构造函数在执行super时，是无参的 」

```js
import { PureComponent } from 'react'

export default class App extends PureComponent {
  constructor(props) {
    super()
    console.log(this.props) // => undefined
  }

  render() {
    // 在构造函数外，我们依旧可以通过this.props 获取传入的props
    console.log(this.props) // => {name: 'Klaus', age: 23}
    // 和函数组件一样，类组件的props，也是被冻结的 --- 是只读的
    console.log(Object.isFrozen(this.props)) // => true
    return (
      <>
        this is a class component
      </>
    )
  }
}
```



## 规则校验

1. 属性规则校验 是 警告错误，不是强制错误 -- 出错依旧可以继续运行
2. 如果存在规则校验，会先进行规则校验，再进行props传递

```js
import { PureComponent } from 'react'
import PropTypes from 'prop-types'

export default class App extends PureComponent {
  // 和函数组件一样，属性规则校验也是通过静态属性来实现的
  static defaultProps = {
    age: 23
  }

  static propTypes = {
    name: PropTypes.string.isRequired,
    age: PropTypes.number
  }

  render() {
    const { name, age } = this.props
    return (
      <>
        {name} - {age}
      </>
    )
  }
}
```

