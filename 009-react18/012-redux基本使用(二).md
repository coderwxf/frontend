## 在react中使用redux

`store/index.jsx`

```jsx
import { createStore } from 'redux'
import reducer from './reducer'

export default createStore(reducer)
```



`store/consts.jsx`

```jsx
export const INCREMENT = 'increment'
export const DECREMENT = 'decrement'
```



`store/actionCreator.js`

```jsx
import { INCREMENT, DECREMENT } from './consts'

export const incrementAction = num => ({
	type: INCREMENT,
	num
})

export const decrementAction = num => ({
	type: DECREMENT,
	num
})
```



`store/reducer.jsx`

```jsx
import { INCREMENT, DECREMENT } from './consts'

const initalState = {
	count: 0
}

// reducer需要返回一个新的state
export default function reducer(state = initalState, action) {
	switch (action.type) {
		case INCREMENT:
			return { ...state, count: state.count + action.num }
		case DECREMENT:
			return { ...state, count: state.count - action.num }
		default:
			return state
	}
}
```



`使用`

```jsx
import React, { PureComponent } from 'react'
import store from '../store'
import { incrementAction, decrementAction } from '../store/actionCreator'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.unSubscribe = () => {}

    // store.subscribe只会在state发生改变的时候才会被调用
    // 所以初始值需要自己主动去state中获取
		this.state = {
			count: store.getState().count
		}
	}


	componentDidMount() {
		this.unSubscribe = store.subscribe(() => {
			const { count } = store.getState()

			this.setState({
				count
			})
		})
	}


	componentWillUnmount() {
		this.unSubscribe()
	}

	render() {
		const { count } = this.state

		return (
			<div>
				<div>App count: { count }</div>
				<button onClick={() => store.dispatch(incrementAction(5))}>increment in app</button>
				<button onClick={() => store.dispatch(decrementAction(5))}>decrement in app</button>
			</div>
		)
	}
}

export default App
```



## react-redux

在上述示例中，我们需要在state中读取store中取出的数据，同时需要在componentDidMount添加订阅，在componentWillUnmount中取消订阅

而这部分的逻辑在每一个需要使用redux数据的组件中都需要完成，也就是说这部分是可以抽取的代码

为此社区提供了一个对应的库 `react-redux`，这个库使用HOC帮助我们对上述重复逻辑进行了抽离

```shell
npm install react-redux
```



`src/index.jsx`

```jsx
import { StrictMode } from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import store from './store'
import { Provider } from 'react-redux'

ReactDOM.createRoot(document.querySelector('#root')).render(
	<StrictMode>
		{/*
			Provider是react-redux提供的组件
			本质上内部使用了context进行数据共享
			Provider组件通过store属性传入对应的store对象
			目的是为了可以在全局使用对应的store
			这样就不需要在每一个需要使用store中状态的组件中单独再引入对应的store
		*/}
		<Provider store={store}>
			<App />
		</Provider>
	</StrictMode>
)
```



`使用`

```jsx
import React, { PureComponent } from 'react'
import { incrementAction, decrementAction } from '../../../store/actionCreator'

// connect函数的作用是将redux和需要使用redux中状态的组件联系起来
import { connect } from 'react-redux'


export class Home extends PureComponent {
	render() {
		return (
			<div>
				count: { this.props.count }
				<button onClick={ () => this.props.increment(5) }>+5</button>
				<button onClick={ () => this.props.decrement(5) }>-5</button>
			</div>
		)
	}
}

// connect本身是一个高阶函数， connect函数接收两个函数作为参数

// connect函数参数一 --- 一个将state映射为组件props的函数
// 在react-redux将对应状态和组件映射起来后
// 只要对应的状态发生了改变，就会重新调用对应组件的render方法，更新对应的UI
// 但实际开发中一个组件中只会使用store中的部分状态
// 如果直接映射store中的state, 就会导致组件中没有使用的状态发生改变的时候
// 组件也会被重新渲染，而这往往是没有必要的
// 所以connect方法的第一个参数方法 会将state做为参数传入
// 而对应的返回值就是我们组件中所需要使用的状态
// 也就是让我们自己决定那些状态需要映射，而那些组件不需要映射
// 而对应的映射值会被作为高级组件所返回的那个组件的props被传入
const mapStateToProp = state => ({
	count: state.count
})


//  2. 我们往往需要操作redux中对应的状态
//  react-redux对dispatch操作进行解耦操作
//  react-redux的第二个参数是一个将对应action操作映射到props中的函数
const mapActionToProps = dispatch => ({
	 increment: num => dispatch(incrementAction(num)),
	 decrement: num => dispatch(decrementAction(num))
})

// connect函数返回值是一个高阶组件
// 该高阶组件需要将需要使用redux中状态的组件传入
// 并返回一个注入了对应状态的新组件
export default connect(mapStateToProp, mapActionToProps)(Home)
```



## 异步请求

我们可以直接通过同步的操作来dispatch action，state就会被立即更新

但是真实开发中，redux中保存的很多数据可能来自服务器，我们需要进行异步的请求，再将数据保存到redux中

`发异步请求存数据的组件`

```jsx
import React, { PureComponent } from 'react'
import { addBannerAction } from '../../../store/actionCreator'
import axios from 'axios'
import { connect } from 'react-redux'


export class Home extends PureComponent {
  constructor(props) {
    super(props)

    this.banners = []
  }

	render() {
		return (
			<div>
				count: { this.props.count }
				<button onClick={ () => this.props.increment(5) }>+5</button>
				<button onClick={ () => this.props.decrement(5) }>-5</button>
			</div>
		)
	}

  // 异步请求在组件中完成
  async componentDidMount() {
    const { data: { data } } = await axios.get('http://www.example.com/home/multidata')
		this.props.addBanner(data.banner.list)
  }
}


const mapActionToProps = dispatch => ({
  addBanner: banners => dispatch(addBannerAction(banners)),
})

export default connect(null, mapActionToProps)(Home)
```



`取数据组件`

```jsx
import React, { PureComponent } from 'react'
import Home from './components/Home'
import { connect } from 'react-redux'

export class App extends PureComponent {
	render() {
		return (
			<div>
				<Home />

				<br />

				<div>banners</div>

				<ul>
					{
						this.props.banners.map(banner => <li key={banner.title}>{ banner.title }</li>)
					}
				</ul>
			</div>
		)
	}
}

const mapStateToProps = state => {
	return {
		banners: state.banners
	}
}

export default connect(mapStateToProps)(App)
```



`基本流程`

![image.png](https://s2.loli.net/2022/09/09/OrvaEjhLFAVi8dK.png)

但是如果发生网络请求的组件使用到了异步请求回来的数据的时候，这种方案是很合适的

但是在本例中，发生网络请求的组件，仅仅只是发送网络请求，把对应的数据存储到Redux中, 其自身并没有使用对应的数据

所以在这种情况下，网络请求到的数据也属于我们状态管理的一部分，更好的一种方式应该是将其也交给redux来管理

也就是将对应的代码请求流程修改为如下结构

![image.png](https://s2.loli.net/2022/09/09/d6zhjQZ9wusFcO1.png)

默认情况下的dispatch(action)，action需要是一个JavaScript的对象，也就是说默认情况下dispatch方法只能执行同步操作

如果需要让redux可以派发异步操作，需要使用到一个中间件 `redux-thunk`

所谓中间件，其功能主要是在请求和响应之间嵌入一些操作的代码，比如cookie解析、日志记录、文件压缩等操作

当使用redux-thunk的时候，可以让dispatch(action函数)，action可以是一个函数

在实际派发之前，redux-thunk的参数函数会被执行，并传入两个参数

1. dispatch函数用于我们在函数中派发action
2. getState函数考虑到我们之后的一些操作需要依赖原来的状态，用于让我们可以获取之前的一些状态

```shell
npm install redux-thunk
```



`store/index.jsx`

```jsx
import { createStore, applyMiddleware } from 'redux'
import reducer from './reducer'
import thunk from 'redux-thunk'

// applyMiddleware方法接收一个参数列表，用于添加对应的中间件
// applyMiddleware(中间件1，中间件2， 中间件3， ...)
// 这里添加对应的redux-thunk
export default createStore(reducer, applyMiddleware(thunk))
```



`请求数据的组件`

```jsx
import React, { PureComponent } from 'react'
import { fetchBannerAction } from '../../../store/actionCreator'
import { connect } from 'react-redux'


export class Home extends PureComponent {
  constructor(props) {
    super(props)

    this.banners = []
  }

	render() {
		return (
			<div>
				count: { this.props.count }
				<button onClick={ () => this.props.increment(5) }>+5</button>
				<button onClick={ () => this.props.decrement(5) }>-5</button>
			</div>
		)
	}

	componentDidMount() {
		// 发生网络请求，请求数据
		this.props.addBanner()
	}

}

// 实际的网络请求在redux中处理
const mapActionToProps = dispatch => ({
  addBanner: () => dispatch(fetchBannerAction()),
})

export default connect(null, mapActionToProps)(Home)
```



`实际的网络请求在redux中被处理`

```jsx
import axios from 'axios'
import { INCREMENT, DECREMENT, ADD_BANNER } from './consts'

export const incrementAction = num => ({
	type: INCREMENT,
	num
})

export const decrementAction = num => ({
	type: DECREMENT,
	num
})

export const addBannerAction = banners => ({
	type: ADD_BANNER,
	banners
})

// 该函数并不会被redux派发，而是会被交给redux-thunk执行
// 而该函数内我们可以去派发对应的同步操作
export const fetchBannerAction = () => {
	return async(dispatch, getState) => {
		// 通过调用getState来获取到最新的state数据
		console.log(getState())

		const { data: { data } } = await axios.get('http://www.example.com/home/multidata')
		dispatch(addBannerAction(data.banner.list))
	}
 }
```



## 拆分reducer

如果我们将所有的状态都放到一个reducer中进行管理，随着项目的日趋庞大，必然会造成代码臃肿、难以维护

所以我们必然希望reducer按照功能点进行拆分成一个个小的reducer，拆分完毕后再合并成一个完整的大的reducer，也就是赋予reducer类似模块化的功能

为此redux给我们提供了一个combineReducers函数可以方便的让我们对多个reducer进行合并

`store/home/index.js`

```jsx
import store from './reducer'

export * from './consts'
export * from './actionCreator'

export default store
```

`store/home/reducer.js`

```jsx
import { ADD_BANNER } from './consts'

const initalState = {
	banners: []
}

export default function reducer(state = initalState, action) {
	switch (action.type) {
		case ADD_BANNER:
		  return { ...state, banners: action.banners ?? []}
		default:
			return state
	}
}
```

`store/home/consts.js`

```jsx
export const ADD_BANNER = 'add_banner'
```

`store/home/actionCreator.js`

```jsx
import axios from 'axios'
import { ADD_BANNER } from './consts'


export const addBannerAction = banners => ({
  type: ADD_BANNER,
  banners
})

export const fetchBannerAction = () => {
  return async(dispatch, getState) => {
    console.log(getState())

    const { data: { data } } = await axios.get('http://www.example.com/home/multidata')
    dispatch(addBannerAction(data.banner.list))
  }
}
```



`store/index.js`

```jsx
import { createStore, applyMiddleware, combineReducers } from 'redux'
import thunk from 'redux-thunk'
// 引入 redux 模块
import Homereducer from './Home/reducer'

// 使用combineReducers函数 将多个redux模块 组合在一起
const reducer = combineReducers({
	home: Homereducer
})

export default createStore(reducer, applyMiddleware(thunk))
```



`使用`

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'

export class App extends PureComponent {
	render() {
		return (
			<div>
				<div>banners</div>

				<ul>
					{
						this.props.banners.map(banner => <li key={banner.title}>{ banner.title }</li>)
					}
				</ul>
			</div>
		)
	}
}

const mapStateToProps = state => {
	return {
    // 因为划分了模块，所以取数据的时候需要从对应的模块中取对应的数据
		banners: state.home.banners
	}
}

export default connect(mapStateToProps)(App)
```



`本例中的combineReducers方法实质上会形成类似于如下的函数`

```jsx
// combineReducer会被编译为类似于如下功能的函数
// reducer本质上是一个接收state和action，并返回新state的函数
const reducer = function(state = {}, action){
	return {
		// 1. 首次执行，state.home和action皆为undefined，所以直接使用默认state
		// 2. 第二次执行的时候，已经存在对应的state值，并且修改state，会传入对应的action值
		// 所以直接取出实际的state值，并将对应的state值和action所对应的值传入给home模块的reducer中
		// 此时就可以获取真正的state对象
		home: Homereducer(state.home, action)
	}
}
```



## devTools

react中的devTools并不像vue一样只有一个

而是将其拆分成了两个divtools，分别为 `react-devtools`(用于调试组件结构)和`redux-devtools`（用于调试redux中存储的数据）

对于redux-devtools默认情况下是不开启的，这样可以保证项目在上线的时候，用户无法直接在redux devtools中查看项目所存储的数据

如果需要开启，可以查看[文档](https://github.com/reduxjs/redux-devtools/tree/main/extension#installation), 在不同的配置环境下，使用不同的方式进行开启

> ps: devTools只需要在开发阶段开启，不应该在生产环境开启，所以在开启的时候，最好判断一下当前的运行环境
