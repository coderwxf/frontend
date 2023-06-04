[React](https://zh-hans.reactjs.org/)是一个用于构建用户界面的 JavaScript 库

1. 声明式编程

   它允许我们只需要维护自己的状态，当状态改变时，React可以根据最新的状态去渲染我们的UI界面

   ![ImynnQ.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/ddf0ef764437462daab80f41c6f63c65~tplv-k3u1fbpfcp-zoom-1.image)  

2. 组件化开发

   组件化开发页面目前前端的流行趋势，我们会将复杂的界面拆分成一个个小的组件

   ![Imy5Jh.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/7c53e455897a46589b2fcb8ae0b66a89~tplv-k3u1fbpfcp-zoom-1.image) 

3. 多平台适配

   React可以开发多平台的应用程序，包括但不限于web端(SPA，SSR), ReactNative等

   React会将JSX编译为对应的虚拟DOM, 随后根据不同的平台可以将虚拟DOM渲染成不同的内容
   
   如web端，虚拟DOM会被渲染为原生DOM对象，而APP端，虚拟DOM会被渲染为原生控件
   
   ![Imy7zS.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f4b3cc1776a44cc59df7746027ff145d~tplv-k3u1fbpfcp-zoom-1.image)  



## 开发依赖

react将对应的依赖库进行了拆分，目的就是让每一个库只单纯做自己的事情，各司其职

| 库        | 说明                                |
| --------- | ----------------------------------- |
| react     | 包含react所必须的核心代码           |
| react-dom | react渲染在不同平台所需要的核心代码 |
| babel     | 将jsx转换成React代码的工具          |

babel在react开发中并不是必须的

react在编写界面的时候使用的是JSX（JavaScript XML)

通过JSX，我们可以很方便的在JS中以HTML形式编写对应的代码

但是这些代码浏览器本身是不认识的，所以需要通过babel将JSX转换为`React.createElement`

所以如果我们直接使用`React.createElement`进行代码的编写，我们就可以不使用bebel进行预先编译

```html
<script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script> 
<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>
```



## 初体验

`1. 在界面上显示Hello World`

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Document</title>
</head>
<body>
	<!-- react.js 会在全局挂载一个React对象 -->
	<script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
	<!-- react-dom.js 会在全局挂载一个ReactDOM对象 -->
	<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
	<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

	<!-- 
		定义挂载点 在vue中一般为div#app 在react中一般为div#root
		挂载点中的内容，会被react渲染出的内容所覆盖
	-->
	<div id="root"></div>

	<!-- type="text/babel"表示对应的js代码在交给浏览器解析之前，需要先交给babel进行编译处理 -->
	<script type="text/babel">
		// react18之前的渲染方式 - 在react18中依旧可以使用，但是会有警告
		// ReactDOM.render(<h2>Hello World</h2>, document.querySelector('#root'))

		// react18中的写法
		// 1. 创建一个根 - 在react18中可以创建多个根，虽然一般情况下，一个web application只要有一个根即可
		const root = ReactDOM.createRoot(document.querySelector('#root'))
		// 2. 在根元素下，挂载我们所需要的内容
		// 在render函数中编写的就是我们的JSX代码
		root.render(<h2>Hello World</h2>)
	</script>
</body>
</html>
```



`2. 点击按钮，使文本由Hello World 转变为 Hello React` 

![ImydIW.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/95a2d191c1484dcba2650a017aedaec3~tplv-k3u1fbpfcp-zoom-1.image)  

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Document</title>
</head>
<body>
	<script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
	<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
	<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

	<div id="root"></div>

	<script type="text/babel">
		let message = 'Hello World'

		function handleClick() {
			message = 'Hello React'
			// 界面发生改变的时候，react并不会像vue一样进行响应式
			// 所以需要手动进行界面的重写渲染
			render()
		}

		const root = ReactDOM.createRoot(document.querySelector('#root'))

		function render() {
			// render函数内部添加一个小括号，以表示内部的内容是一个整体 - 这里不添加也是可以的
			root.render((
				<div>
					{/* 这是JSX中的注释 */}
					{/* JSX中使用大括号来绑定对应的变量 */}
					<h2>{ message }</h2>
					{/*
						1. react中绑定内置事件使用的是小驼峰命名法，因为其对默认事件进行了二次封装
							 这样就可以根据不同平台渲染成不同的事件
							 如web端渲染成原生浏览器事件, app端，渲染成对应的控件事件
						2. 在JSX中，只要使用变量或表达式就需要使用大括号语法
							 所以在这里进行事件绑定的时候使用的也是大括号语法
					*/}
					<button onClick={ handleClick }>click me</button>
				</div>
			))
		}

		// 页面初始渲染
		render()
	</script>
</body>
</html>
```



上述示例中渲染UI的逻辑和业务处理的逻辑是十分分散的，而且每当状态发生改变的时候

都需要手动调用render方法重新渲染界面，为此我们可以将对于的UI界面，业务逻辑和样式封装在一起

这个封装的地方就被称之为组件

`在react中，组件分为类组件和函数式组件，无论是哪一种，对于的组件必须首字母大写，否则会被识别为普通html元素`

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta http-equiv="X-UA-Compatible" content="IE=edge">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<title>Document</title>
</head>
<body>
	<script src="https://unpkg.com/react@18/umd/react.development.js" crossorigin></script>
	<script src="https://unpkg.com/react-dom@18/umd/react-dom.development.js" crossorigin></script>
	<script src="https://unpkg.com/babel-standalone@6/babel.min.js"></script>

	<div id="root"></div>

	<script type="text/babel">
		const root = ReactDOM.createRoot(document.querySelector('#root'))

		// React中的类组件必须满足如下两个条件:
		// 1. 类必须继承自React.Component 或 React.pureComponent
		// 2. 类必须实现一个返回虚拟DOM的render方法
		class App extends React.Component {
			constructor(props) {
				super(props)

				// 不参与界面更新的数据 也就是那些当数据变量时，不需要更新将组件渲染的内容 可以定义在state外部
				this.notUIData = false

				// 参与界面更新的数据（参与数据流），也就是那些当数据变量时，需要更新组件渲染的内容必须定义在state中
				this.state = {
					message: 'Hello World'
				}
			}

			// 在这里定义对应的事件
			handleClick() {
				// 修改完状态，需要调用this.setState以通知react执行update操作
				// 当react执行update操作的时候，react会自动使用最新的数据重新渲染界面
				// 其内部传入的对象会和this.state执行Object.assign合并
				this.setState({
					message: 'Hello React'
				})
			}

			// 界面渲染的时候，会自动去调用组件的render方法
			// 该方法需要返回一个JSX对象
			render() {
				// return后边添加小括号是表明后续内容是一个整体, 并且可以换行编写
				return (
					<div>
						{/* 在react中, JSX有且只能有一个根元素 */}
						<h2>{ this.state.message }</h2>
						{/*
							因为React并不是直接渲染成真实的DOM，我们所编写的button只是一个语法糖
							它的本质是ReactElement对象
							在这里发生事件监听的时候，react在执行函数时并没有绑定this，默认情况下就是一个undefined
							之所以这里是undefined，是因为使用babel进行编译后的代码
							以及原生js中类和模块中的代码默认就会开启严格模式
							而在render方法中我们可以获取到正确的this指向
							所以我们需要将render中的this进行传入，以修正对应的this指向
						*/}
						<button onClick={ this.handleClick.bind(this) }>click me</button>
					</div>
				)
			}
		}

		// 渲染app组件 - render方法的参数即可以是html元素也可以是对应的组件
		root.render(<App />)
	</script>
</body>
</html>
```



`3.列表渲染`

```jsx
class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      users: ['Klaus', 'Alex', 'Steven']
    }
  }

  render() {
    return (
      <ul>
        {
          // 1. JSX中可以放任何合法的js变量和js表达式 在表达式内部也可以继续使用JSX
          // 2. 在JSX中 如果需要渲染的是数组，react会执行类似arr.join('')操作后再进行渲染
          this.state.users.map(user => <li key={user}>{ user }</li>)
        }
      </ul>
    )
  }
}
```



`4. 计数器`

```jsx
class App extends React.Component {
  constructor(props) {
    super(props)

    this.state = {
      count: 0
    }
  }

  changeCount(step) {
    this.setState({
      count: this.state.count + step
    })
  }

  render() {
    return (
      <div>
        <div>{ this.state.count }</div>
        {/* 使用箭头函数修正this 可以更为方便的进行参数的传递 */}
        <button onClick={() => this.changeCount(1)}>+1</button>
        <button onClick={() => this.changeCount(-1)}>-1</button>
      </div>
    )
  }
}
```

