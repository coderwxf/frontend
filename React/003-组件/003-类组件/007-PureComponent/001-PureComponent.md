默认情况下，`Component`没有实现`shouldComponentUpdate`, 所以只要调用`setState`，组件一定会更新

而`PureComponent`实现了自己的`shouldComponentUpdate`

```js
// 默认情况下，会对新老props和新老state进行浅比较
shouldComponentUpdate(nextProps, nextState) {
  return nextState !== this.state && nextProps !== this.props
}
```



```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  state = {
    users: ['Klaus', 'Alex', 'Steven']
  }

  render() {
    return (
      <>
        <h1>Users</h1>
        <ul>
          {this.state.users.map((user, index) => (
            <li key={index}>{user}</li>
          ))}
        </ul>
        <button onClick={() => this.addUser}>add user</button>
      </>
    )
  }

  addUser = () => {
    // 以下操作 oldState和newState 内存地址相同，SCU返回false，不会触发render
    // this.state.users.push('Peter')
    // this.setState({ users: this.state.users })

    this.setState({
      // 在浅拷贝的同时添加新的用户 -- 新state和旧state内存地址不同，SCU返回true，触发render
      users: [...this.state.users, 'Peter']
    })
  }
}
```

