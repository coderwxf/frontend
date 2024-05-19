react在调用合成事件对象时，使用默认函数调用，类似于`fn()`

以下情况下，会默认开启严格模式:

1. ES6中的模块
2. 经过babel编译的代码

在严格模式下，全局this的值是undefined

JSX需要通过babel转换为`React.createElement()`， 因此react合成事件中的this默认值为`undefined`



## bind

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  handleClick() {
    console.log(this)
  }

  // render方法会被组件实例调用，所以render中的this指向是正确的
  render() {
    return (
      <>
      	{/* 使用bind方法预处理this，从而修正this指向 */}
        <button onClick={this.handleClick.bind(this)}>click me</button>
      </>
    )
  }
}
```



## 类表达式

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  // 类表达式会被挂载到组件实例上，而不是组件实例的原型上
  // 经过编译后，类表达式实际是在构造函数中被执行的，所以类表达式中的this就是组件实例
  handleClick = () => {
    console.log(this)
  }

  render() {
    return (
      <>
        <button onClick={this.handleClick}>click me</button>
      </>
    )
  }
}
```



## 箭头函数

```jsx
import { PureComponent } from 'react'

export default class App extends PureComponent {
  handleClick() {
    console.log(this)
  }

  render() {
    return (
      <>
      	{/* 箭头函数没有自己的this，使用外层作用域中的this，也就是render方法中的this */}
        <button onClick={() => this.handleClick()}>click me</button>
      </>
    )
  }
}
```

