Redux Toolkit 是官方推荐的编写 Redux 逻辑的方法

早期的redux的编写逻辑过于的繁琐和麻烦并且代码通常分拆在多个文件中

虽然也可以放到一个文件中进行管理，但是代码量过多，不利于管理

Redux Toolkit包旨在成为编写Redux逻辑的标准方式，从而解决上面提到的问题

所以Redux Toolkit本质上就是对原生Redux的二次封装

在很多地方为了称呼方便，也将之称为“RTK”

```shell
# 安装
# RTK本质上是官方对redux的封装
# RTK内部集成了react-redux的功能，也就是说RTK内部依赖react-redux，所以需要一起下载
npm install @reduxjs/toolkit react-redux
```



## 核心API

RTK的几个核心API如下;

| API              | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| configureStore   | createStore以提供简化的配置选项和良好的默认值<br />它可以自动组合你的 slice reducer，添加你提供 的任何 Redux 中间件<br />默认集成了redux-thunk，并默认启用了Redux DevTools Extension |
| createSlice      | 接受reducer函数的对象、切片名称和初始状态值，并自动生成切片reducer，并带有相应的actions |
| createAsyncThunk | 接受一个动作类型字符串和一个返回承诺的函数，并生成一个pending/fulfilled/rejected基于该承诺分派动作类型的 thunk |



### configureStore

| 参数       | 说明                                                    |
| ---------- | ------------------------------------------------------- |
| reducer    | 将slice中的reducer可以组成一个对象传入此处              |
| middleware | 可以使用参数，传入其他的中间件，默认已经集成react-redux |
| devTools   | 是否配置devTools工具，默认为true                        |



### createSlice

| 参数         | 说明                                                         |
| ------------ | ------------------------------------------------------------ |
| name         | 用户标记slice的名词<br />在之后的redux-devtool中会显示对应的名词 |
| initialState | 初始值<br />第一次初始化时的值                               |
| reducers     | reducers本质上一个对象，对象中可以配置多个函数<br />该reducers类似于之前的reducer函数，而其中的一个个函数相当于之前的一条条case语句<br />每一个函数都会接收两个参数，state和action |

createSlice返回值是一个对象，包含所有的actions和reducer函数



## 示例

`store/index.js`

```js
import { configureStore } from '@reduxjs/toolkit'
import countReducer from './modules/counter'

// 生成store
export default configureStore({
	// 将所有子模块匹配值在这里
	reducer: {
		counter: countReducer
	}
})
```



`store/module/conter.js`

```jsx
import { createSlice } from '@reduxjs/toolkit'

// 定义切片
const countSlice = createSlice({
	name: 'home', // slice的唯一标识 - 用于在redux devtool中进行唯一标识
	initialState: { // 初始state
		count: 0
	},
	reducers: { // 一个个的action操作
		// 参数一： state
		// 参数二：action
		// action中有两个参数
		// action.type - 执行的方法名称，为模块名/方法名，例如 home/increment
		// action.payload - 外部传入的参数
		increment(state, action){
			// 在这里直接修改即可
			// RTK内部会确保返回的是一个新的state
			state.count += action.payload
		},
		decrement(state, action){
			state.count -= action.payload
		}
	}
})

// 导出对应的aciton方法，以供外部使用
// 为了避免action和外部自定义方法重名
// 所以导出的action 一般以Action作为后缀名

// 在解构的同时 进行导出
export const { increment: incrementAction, decrement: decrementAction } = countSlice.actions

// 实际导出的是slice中的reducer
export default countSlice.reducer
```



`/src/index.js`

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
// react-redux的使用方法和之前是一致的
import { Provider } from 'react-redux';
import App from './App';
import store from './store'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <React.StrictMode>
    <Provider store={store}>
			<App />
		</Provider>
  </React.StrictMode>
);
```



`使用者`

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'
import { incrementAction, decrementAction } from '../store/modules/counter'

export class Home extends PureComponent {
	render() {
		const { count, increment, decrement } = this.props

		return (
			<div>
				<h1>Home Count: { count }</h1>
				<button onClick={() => increment(5)}>+5</button>
				<button onClick={() => decrement(5)}>-5</button>
			</div>
		)
	}
}

const mapStateToProps = state => {
	return {
		count: state.counter.count
	}
}

const mapDispatchToProps = dispatch => {
	return {
		increment: num => {
			dispatch(incrementAction(num))
		},
		decrement: num => {
			dispatch(decrementAction(num))
		}
	}
}

export default connect(mapStateToProps, mapDispatchToProps)(Home)
```



## 异步请求

在不是有RTK的时候，我们通过redux-thunk中间件让dispatch中可以进行异步操作

而Redux Toolkit默认已经给我们集成了Thunk相关的功能:  createAsyncThunk



当createAsyncThunk创建出来的action被dispatch时，会存在三种状态

+ pending:action被发出，但是还没有最终的结果
+ fulfilled:获取到最终的结果(有返回值的结果)
+ rejected:执行过程中有错误或者抛出了异常



```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'
import axios from 'axios'

// createAsyncThunk 接收两个参数
// 参数一: 异步action的唯一名称 - 会在redux devtool中显示
// 参数二: 一个回调函数，可以在其中执行我们所需要的异步操作
export const fetchBannersAction = createAsyncThunk('fetch/bannersAction', async() => {
	const res = await axios.get('http://www.example.com:8000/home/multidata')

	// 异步action方法返回的值 会被作为对应extraReducer的action.payload的对应值
	// ps: 如果使用axios发生网络请求的时候，不能直接返回res，而是需要返回res.data
	// 因为res中除了服务器返回的数据而包括一些别的信息，如果headers，configs...
	// 而redux-toolkit中使用了Serializability中间件，该中间件会强制redux只存储可序列化数据
	// 也就是那些将对象转换为文本后再重新转换为对象后不会丢失信息的对象，也就是说该对象中不可以存在函数，symbol等
	// 所以如果使用axios发生了相应的网络请求，需要返回res.data 而不是res
	return res.data
})

const bannerSlice = createSlice({
	name: 'home',
	initialState: {
		banners: []
	},
	// reducer中定义的action 一般都是同步action
	// extraReducers中定义的action 一般都是异步action
	extraReducers: {
		// 异步action对应方法在被回调的时候会将state和action作为参数传入
		// 对于异步action，会存在对应的三种状态，其状态值和promise的三种状态是一致的
		// 分别为 pending resolved rejected
		[fetchBannersAction.pending]() {
			console.log('fetchBannersAction.pending')
		},
		// action依旧是使用type和payload组成, 所以在使用的时候可以直接解构
		[fetchBannersAction.fulfilled](state, { payload }) {
			state.banners = payload.data.banner.list
		},
		[fetchBannersAction.rejected]() {
			console.log('fetchBannersAction.rejected')
		}
	}
})

export default bannerSlice.reducer
```

`使用者`

```jsx
import React, { PureComponent } from 'react'
import { connect } from 'react-redux'
import { fetchBannersAction } from '../store/modules/banner'

export class Profile extends PureComponent {
	componentDidMount() {
		this.props.fetchBanners()
	}

	render() {
		return (
			<>
				<div>Profile</div>
				<ul>
					{
						this.props.banners.map(banner => <li key={banner.acm}>{banner.title}</li>)
					}
				</ul>
			</>
		)
	}
}

const mapStateToProps = state => {
	return {
		banners: state.banner.banners
	}
}

const mapDispatchToProps = dispatch => {
	return {
		fetchBanners: () => {
			dispatch(fetchBannersAction())
		}
	}
}

export default connect(mapStateToProps, mapDispatchToProps)(Profile)
```



### 异步请求的其它写法

`写法一 - 链式调用extraReducer`

extraReducer还可以传入一个函数，函数接受一个builder参数

我们可以向builder中添加case来监听异步操作的结果，也就是使用链式调用的方式来编写对应的case语句

```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'
import axios from 'axios'

export const fetchBannersAction = createAsyncThunk('fetch/bannersAction', async() => {
	const res = await axios.get('http://www.example.com:8000/home/multidata')

  // 这里返回的结果，会让对应的action的状态转变为resolved
  // 并使用该方法的返回值作为resolved的参数值
	return res.data
})

const bannerSlice = createSlice({
	name: 'home',
	initialState: {
		banners: []
	},
	extraReducers(builder) {
		builder.addCase(fetchBannersAction.pending, () => console.log('fetchBannersAction.pending'))
					 .addCase(fetchBannersAction.fulfilled, (state, { payload }) => { state.banners = payload.data.banner.list })
					 .addCase(fetchBannersAction.rejected, () => console.log('fetchBannersAction.rejected'))
	}
})

export default bannerSlice.reducer
```



`写法二 - 直接在异步action中操作同步action`

```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit'
import axios from 'axios'

// createAsyncThunk方法的第二个回调函数可以传入一个回调函数
// 参数一是调用fetchBannersAction方法传入的参数对象
// 参数二是store实例对象，通过解构可以获取对应的dispatch和getState方法
export const fetchBannersAction = createAsyncThunk('fetch/bannersAction', async(payload, { dispatch }) => {
	const res = await axios.get('http://www.example.com:8000/home/multidata')

	// 异步结果拿到结果后，调用同步action设置对应状态
	dispatch(bannerSlice.actions.addBanner(res.data.data.banner.list))
})

const bannerSlice = createSlice({
	name: 'home',
	initialState: {
		banners: []
	},
	reducers: {
		addBanner(state, { payload }) {
			state.banners = payload
		}
	}
})

export default bannerSlice.reducer
```