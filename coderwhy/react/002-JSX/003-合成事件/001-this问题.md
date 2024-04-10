```jsx
import { Component } from 'react'

export default class App extends Component {
  state = {
    name: 'React'
  }

  changeName() {
    // 1. react内部使用事件委托的方式调用changeName方法 --- 实际调用形式是 changeName()
    // 2. 在ES6模块和类中 this默认指向undefined
    // 3. 在babel转换后的代码，默认开启严格模式，全局this指向undefined
    // 4. 所以在changeName中的this指向undefined，需要进行this修正
    console.log(this)
  }

  render() {
    return (
      <>
        <div>{ this.state.name }</div>
        {/*
          react事件是合成事件 -- 在不同浏览器对应事件的基础上进行了二次封装，以屏蔽他们之间的差异
          所以react中的事件使用小驼峰命名法
        */}
        <button onClick={this.changeName}>change name</button>
      </>
    )
  }
}
```



`修正方式`

```jsx
import { Component } from 'react'

export default class App extends Component {
  state = {
    name: 'React'
  }

  changeName() {
    console.log(this)
  }

  changeName1 = () => {
    console.log(this)
  }

  // render是类组件主动调用的方法，内部this就是组件实例
  render() {
    return (
      <>
        <div>{ this.state.name }</div>
        {/* 修正方式一 --- bind */}
        <button onClick={this.changeName.bind(this)}>change name</button>
        {/* 修正方式二 --- 类成员表达式 */}
        <button onClick={this.changeName1}>change name</button>
        {/* 修正方式三 --- 箭头函数 */}
        <button onClick={() => { this.changeName() }}>change name</button>
      </>
    )
  }
}
```

