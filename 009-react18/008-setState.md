开发中我们并不能直接通过修改state的值来让界面发生更新

因为我们修改了state之后，希望React根据最新的State来重新渲染界面，但是这种方式的修改React并不知道数据发生了变化

React并没有实现类似于Vue2中的Object.defineProperty或者Vue3中的Proxy的方式来监听数据的变

我们必须通过setState来告知React数据已经发生了变化



## 基本使用

```jsx
export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			name: 'Klaus',
			age: 23
		}
	}


	render() {
		const { name, age } = this.state

		return (
			<div>
				<div>{ name }</div>
				<div>{ age }</div>
				<button onClick={ () => this.handleClick() }>click me</button>
			</div>
		)
	}

	handleClick() {
		// 对象形式传入新数据
		// 新数据会和旧数据执行Object.assign操作
		this.setState({
			name: 'Alex'
		})

		// 传入一个函数作为参数
		// 该函数会将之前的state和props传入
		// 如果新的state或props依赖于之前的state和props的时候
		// 可以在其中执行对应的业务逻辑
		this.setState((state, props) => {
			console.log(state, props)

			// 返回一个新的state
			// 新的state也会和原本的state执行Object.assign操作
			return {
				age: 18
			}
		})
	}
}
```



## setState是异步更新的

setState是异步的操作，我们并不能在执行完setState之后立⻢拿到最新的state的结果

setState设计为异步，可以显著的提升性能

因为当state中数据发送改变的时候，需要重新执行render函数

如果每次调用 setState都进行一次更新，那么意味着render函数会被频繁调用，界面重新渲染，这样效率是很低

最好的办法应该是获取到多个更新后加入到一个任务队列中，在合适的时机对其进行批量更新

而且如果仅仅只是更新state而不等待render函数之外完毕的话，会导致父组件中的state和对应子组件中的props很有可能是不同步的，这样就会导致数据的混乱

```jsx
export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			name: 'Klaus',
			age: 23
		}
	}


	render() {
		const { name, age } = this.state

		return (
			<div>
				<div>{ name }</div>
				<div>{ age }</div>
				<button onClick={ () => this.handleClick() }>click me</button>
			</div>
		)
	}

	handleClick() {
		// setState方法是异步操作，并不会立即更新state
		// 如果需要拿到更新后的新数据，可以传入第二个参数
		// 第二个参数是一个回调函数，当state中的数据更新完毕后
		// setState方法会自动回调该方法
    // 我们也可以在组件完全更新完毕后，也就是在componentDidUpdate中拿到更新后的props和state
		this.setState({
			name: 'Alex'
		}, () => {
			console.log('++++', this.state.name)
		})

		console.log('----', this.state.name)
	}
}
```



```jsx
export class App extends PureComponent {
	constructor() {
		super()

		this.state = {
			count: 0
		}
	}

	render() {
		const { count } = this.state

		return (
			<div>
				<div>{ count }</div>
				<button onClick={() => this.changeCount()}>changeCount</button>
			</div>
		)
	}

	changeCount() {
		// 在这里虽然执行了三次state
		// 但是因为state的更新是异步的
		// 所以每次在调用setState的时候，对应的this.state.count的值都是0
		// 所以最终结果 也是count的值由0转变为了1
		this.setState({ count: this.state.count + 1 })
		this.setState({ count: this.state.count + 1 })
		this.setState({ count: this.state.count + 1 })
	}
}
```



```jsx
export class App extends PureComponent {
	constructor() {
		super()

		this.state = {
			count: 0
		}
	}

	render() {
		const { count } = this.state

		return (
			<div>
				<div>{ count }</div>
				<button onClick={() => this.changeCount()}>changeCount</button>
			</div>
		)
	}

	changeCount() {
		this.setState({ count: this.state.count + 1 })

		// 如果执行setState方法的时候 传入的参数是一个函数
		// 那么其作为参数传入的state是批处理队列中，上一次的更新返回的state
		// 所以在本例中 count的值会由0转变为3
		this.setState(state => ({
			count: state.count + 1
		}))

		this.setState(state => ({
			count: state.count + 1
		}))
	}
}
```



## setState什么情况下是同步的

### 在react18之前

+ 在组件生命周期或React合成事件中，setState是异步
+ 在setTimeout或者原生dom事件以及Promise这类不是由React进行发起的事件中，setState是同步



### 在react18后

+ 在React18之后，默认所有的操作都被放到了批处理中(异步处理)
+ 如果希望代码可以同步会拿到，则需要执行特殊的flushSync操作

```jsx
export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			name: 'Klaus'
		}
	}

	render() {
		const { name } = this.state

		return (
			<div>
				<div>{ name }</div>
				<button onClick={ () => this.btnClick() }>click me</button>
			</div>
		)
	}

	btnClick() {
		// flushSync方法接收一个回调函数
		// 对于回调函数内部的代码都是批处理操作
		// 但是回调函数和外部的代码并不是在一起进行批处理操作的
		// flushSync方法会将内部的代码进行统一批处理操作后，执行一次render方法
		flushSync(() => {
			this.setState({
				name: 'Alex'
			})
		})

		console.log(this.state.name)
	}
}
```