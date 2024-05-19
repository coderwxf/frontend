## 基本使用

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  // props传入构造函数，并通过父类挂载到组件实例上
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



即使我们没有通过构造函数来接收props，react内部也会通过父类构造函数来接收传入的props

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  constructor() {
    super()
    // 没有显示接收props，在构造函数内无法获取props
    console.log(this.props) // => undefined
  }

  render() {
    // 没有显示接收props，除构造函数外的任何方法内都可以获取props
    console.log(this.props) // => {name: 'Klaus', age: 23}
    console.log(Object.isFrozen(this.props)) // => true
    return (
      <>
        this is a class component
      </>
    )
  }
}
```



因为默认存在一个无参的构造函数，内部接收props后，就可以省略构造函数，减少编写的代码

所以上述代码可以被简写为

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  render() {
    console.log(this.props) // => {name: 'Klaus', age: 23}
    return (
      <>
        this is a class component
      </>
    )
  }
}
```



 react遵循单向数据流，所以props也是被冻结的，意味着props应该是只读的



## 规则校验

规则校验使用方式和函数组件的规则校验方式一致

```jsx
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



