React是一个用于构建用户界面的 JavaScript 库

+ 声明式编程 --- 只需要维护自己的状态，当状态改变时，React可以根据最新的状态去渲染我们的UI界面
+ 组件化开发 --- 将一个复杂页面拆分为一个个的小的组件
+ 多平台适配 --- react开发web端，react-native开发app端，react VR 开发 VR App

![image-20230429045440563](https://s2.loli.net/2023/04/29/w1DemWEknXoOx95.png)



### 开发依赖

| 库           | 说明                           |
| ------------ | ------------------------------ |
| react        | React核心代码 --- 用于生成VDOM |
| react=dom    | 将VDOM渲染为真实DOM            |
| react-native | 将VDOM渲染为移动端原生控件     |
| babel        | 将JSX转换为原生JavaScript      |



### 使用示例

```html
<script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

<!-- 
	挂载点 
		--- react 一般为 root 
		--- vue 一般为 app 

	实际通过react渲染出的内容会覆盖挂载点下的内容
-->
<div id="root"></div>

<!--
  babel的目的是 JSX -> React.createElement
  所以babel并不是必须的

	text/babel 表示script中的内容需要先交给babel处理后再交给浏览器进行解析
-->
<script type="text/babel">
  // 定义挂载的根节点
  // ReactDOM --- DOM都是大写
  const root = ReactDOM.createRoot(document.getElementById('root'))

  // 将需要渲染的内容挂载到根节点下
  root.render(<h2>Hello React</h2>)

  // 在react18之前 挂载是通过 React.render(<root VDOM>, <root element>)
</script>
```



### 初体验

```jsx
// 定义挂载root节点
// render函数可以被多次调用， 但是挂载根节点的定义只需要定义一次即可
const root = ReactDOM.createRoot(document.getElementById('root'))

let message = 'Hello World'

function changeMsg() {
  message = 'Hello React'

  // React是MVC框架，并不会像Vue一样监听状态改变并实时更新副作用
  // 所以在React中 只要状态发生改变 就需要手动调用render函数 以便于使用最新状态重新渲染界面
  render()
}

function render() {

  // 1. 在vDom外包裹小括号, 表示他们是一个整体, 方便换行编写
  // 2. 在react中有且只能有一个根元素
  // 3. 在react中使用{}来绑定方法和变量以及属性
  // 4. 在react中使用 {/* */} 来添加注释
  //    + 在react中 {} 中可以编写 JS变量 和 合法的JS表达式
  //    + 所以{/* */}中 外面的{} 表示里面是JS语法 中间的/* */才是真正的注释
  root.render((
    <div>
      <h2>{ message }</h2>
      <button onClick={ changeMsg }>change</button>
    </div>
  ))
}

// 初始渲染
render()
```



```jsx
// 我们可以将渲染的内容封装成一个组件 --- 拥有自己HTML, CSS 和 JavaScript的自包含代码块
// 在React中, 组件分为类组件和函数组件 --- 无论类组件还是函数组件，首字母都要大写 以区分普通元素

// 只有继承自React.PureComponent或React.Component并正确实现了render函数的类才是react的类组件
class App extends React.PureComponent {
  // 在react中的数据分为两种
  // 1. 参与界面更新的数据: 当数据变化的时候，需要更新组件渲染的内容
  // 2. 不参与界面更新的数据: 当数据变化的时候，不需要更新将组建渲染的内容
  constructor() {
    super()

    // 在这里定义不参与界面更新的数据
    // this.cpnName = 'App'

    // 在state中 定义参数界面更新的数据
    this.state = {
      message: 'Hello World'
    }
  }

  handleClick() {
    // 需要使用setState方法来更新数据 --- 更新数据 + 重新执行render方法
    // setState中的对象 会和 原本的state对象 执行 Object.assign操作
    this.setState({
      message: 'Hello React'
    })
  }

  // render方法是react类组件必须实现的方法
  // 该方法所返回的内容就是实际需要被渲染的内容
  // 所以render方法一般需要返回虚拟DOM
  render() {
    return (
      <div>
        {/* 在render函数中的this就是当前组件实例对象 */}
        <h2>{ this.state.message }</h2>
        {/*
          1. 在react中事件名使用小驼峰，这是因为react在原生事件的基础上进行了二次封装
              + 如果是浏览器 执行原生DOM事件
              + 如果是移动端 执行原生控件事件
          2. 在react中 事件对应的绑定函数中this的值是undefined
                --- 因为react并不知道方法调用者是原生DOM还是移动端控件
                --- 所以在react中的事件回调在调用的时候形式类似于默认调用
                --- 而在类组件中，实例函数在默认调用的时候，对应的this就是undefined
             所以在实际调用的时候推荐使用箭头函数在外层包裹一层
              --- 因为箭头函数内部不绑定this，所以内部的this会顺着作用域链进行查找
              --- 而事件绑定函数的上一层就是render函数，所以其this就是当前组件实例对象
          3. 类中的方法会默认开启严格模式
          		--- 如果通过实例调用，方法中的this是实例
          		--- 如果通过默认调用, 方法中的this是undefined，不是globalThis
          4. 使用babel编辑的JavaScript会默认开启严格模式
          5. 模块化JavaScript代码也会默认开启严格模式
        */}
        <button onClick={() => this.handleClick()}>change</button>
      </div>
    )
  }
}

const root = ReactDOM.createRoot(document.getElementById('root'))

<!-- App为当前组件树的根组件 -->
root.render(<App />)
```