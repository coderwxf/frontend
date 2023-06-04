## Redux Toolkit的数据不可变性

在React开发中，我们总是会强调数据的不可变性:

+ 无论是类组件中的state，还是redux中管理的state
+ 事实上在整个JavaScript编码过程中，数据的不可变性都是非常重要的

因此我们经常会进行浅拷贝来完成某些操作，但是浅拷贝事实上也是存在问题的

+ 比如过大的对象，进行浅拷贝也会造成性能的浪费
+ 比如浅拷贝后的对象，在深层改变时，依然会对之前的对象产生影响

而Redux Toolkit底层使用了immerjs的一个库来保证数据的不可变性

immer 实现的算法方式是Persistent Data Structure（持久化数据结构），也就是使用旧数据创建新数据时，要保证旧数据同时可用且不变。同时为了避免 deepCopy 把所有节点都复制一遍带来的性能损耗，Immutable 使用了（结构共享），即如果对象树中一个节点发生变化，只修改这个节点和受它影响的父节点，其它节点则进行共享

![Persistent Data Structure](https://mmbiz.qpic.cn/mmbiz_gif/O8xWXzAqXuuPtxc2VNSb80zpYnIGMuvn6vRJMGliaqLp8wWNEgKOVutM4vjiaiaGD0iba6tYMQ8DFV8MsYzC7via0bg/640?wx_fmt=gif)





## 自定义connect函数

```jsx
import { PureComponent } from "react"
import store from '../store'

export function connect(mapStateToProps, mapDispatchToProps) {
	return function(Cpn) {
		return class enhanceCpn extends PureComponent {
			constructor(props) {
				super(props)

				// 调用mapStateToProps方法，返回的就是我们所实际需要使用的store中的数据
				this.state = mapStateToProps(store.getState())
			 }

			 componentDidMount() {
				store.subscribe(() => {
					// 一旦发生更新重新执行mapStateToProps得到最新的state
					this.setState(mapStateToProps(store.getState()))
				})
			 }

			render() {
				let mapState = {}
        let mapDispatch = {}

				if (typeof mapStateToProps === 'function') {
					mapState = mapStateToProps(store.getState())
				}

        if (typeof mapDispatchToProps === 'function') {
          mapDispatch = mapDispatchToProps(store.dispatch)
        }

				return <Cpn { ...this.props } { ...mapState } {...mapDispatch} />
			}
		}
	}
}
```



虽然这么做依旧模拟实现了connect函数的功能，但是该函数存在一个致命的问题，那就是它需要自己主动引入store

也就是说该文件和store的位置是强耦合的

为此我们还需要模拟实现一些Provider组件

`context.js`

```jsx
import { createContext } from 'react'
export const StoreCtx = createContext()
```



`Provider.jsx`

```jsx
import React, { PureComponent } from 'react'
import { StoreCtx } from './context'

export class Provider extends PureComponent {
	render() {
		return (
			<StoreCtx.Provider value={this.props.store}>
				{ this.props.children }
			</StoreCtx.Provider>
		)
	}
}

export default Provider
```



`/src/index.jsx`

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from './utils';
import App from './App';
import store from './store'

const root = ReactDOM.createRoot(document.getElementById('root'))

root.render(
  <React.StrictMode>
    <Provider store={store}>
			<App />
		</Provider>
  </React.StrictMode>
)
```



`使用store中数据的组件`

```jsx
import { PureComponent } from "react"
import { StoreCtx } from './Provider'

export function connect(mapStateToProps, mapDispatchToProps) {
	return function(Cpn) {
		return class enhanceCpn extends PureComponent {
			static contextType = StoreCtx

      // 构造方法有第二个参数，为指定的context
			constructor(props, context) {
        super(props)

        this.unsubscribe = () => {}

        this.state = mapStateToProps(context.getState())
      }

      componentDidMount() {
        this.unsubscribe = this.context.subscribe(() => {
          this.setState(mapStateToProps(this.context.getState()))
        })
      }

      componentWillUnmount() {
        this.unsubscribe()
      }

			render() {
				let mapState = {}
        let mapDispatch = {}

				if (typeof mapStateToProps === 'function') {
					mapState = mapStateToProps(this.context.getState())
				}

        if (typeof mapDispatchToProps === 'function') {
          mapDispatch = mapDispatchToProps(this.context.dispatch)
        }

				return <Cpn { ...this.props } { ...mapState } {...mapDispatch} />
			}
		}
	}
}
```



## 中间件

react的中间件本质上就是对dispatch操作进行拦截，在对派发的操作进行二次处理后，再将处理后的结果实际派发给redux

在拦截后，可以进行任何所需要的自定义操作，从而扩展对应的功能



需求： 在dispatch之前，打印一下本次的action对象，dispatch完成之后可以打印一下最新的store state

 也就是我们需要将对应的代码插入到redux的某部分，让之后所有的dispatch都可以包含这样的操作

```js
import store from './store'

function log(store) {
	// 该函数本质上就可以看成是一种中间件
	// 实际调用的看上去是dispatch，但是本质上是dispatchAndLog
	// 对原本的dispatch方法扩展了打印日志操作后，在调用原本的dispatch方法
	function dispatchAndLog(action) {
		console.log('正在派发', action)
		next(action)
		console.log('派发结果为', store.getState())
	}

	// 这种行为被称之为monkey patching 猴补丁
	// 也就是篡改现有对象，对整体执行逻辑进行修改
	const next = store.dispatch
	store.dispatch = dispatchAndLog
}

log(store)
```



所以 redux-thunk 的 核心代码为

```jsx
function thunk(store) {
	const next = store.dispatch

	function dispatchThunk(action) {
		// 如果传入的是函数，直接调用
    // 并传入修改后的dispatch --- 方便继续派发函数
    // 同时传入store.getState函数 --- 方便调用者获取最新state
    if (typeof action === 'function') {
			action(store.dispatch, store.getState)
		} else {
      // 如果不是函数，是对象形式，直接调用原生派发
			next(action)
		}
	}
	store.dispatch = dispatchThunk
}

thunk(store)
```



单个调用某个函数来合并中间件并不是特别的方便，我们可以封装一个函数来实现所有的中间件合并:

```jsx
// 这里之所以需要多传递一个store
// 是因为redux的applyMiddleWare是直接在createStore的时候传入的参数，其可以直接获取到内部的store
// 而自己封装无法直接获取到对应的store，所以需要显式的传入
function applyMiddleWare(store, ...fns) {
	fns.forEach(fn => fn(store))
}
```