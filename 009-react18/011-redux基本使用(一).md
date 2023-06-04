## 纯函数

纯函数必须满足如下条件：

1. 确定的输入，一定会产生确定的输出
2. 函数在执行过程中，不能产生副作用

 在执行一个函数时，除了返回函数值之外，还对调用函数产生了附加的影响， 比如修改了全局变量，修改参数或者改变外部的存储, 就被称之为产生了对应的副作用，而副作用往往是产生bug的 “温床”

当我们使用纯函数去编写组件的时候，就可以在写的时候保证组件的纯度，即只关心组件所需要完成的业务逻辑即可，不需要关心传入的内容或外部依赖是否发生了改变

同时因为纯函数对于确定的输入一定会产生确定的输出，所以其和外部环境的耦合度是非常低的，有利于打包工具进行对应的tree shaking操作

所以在React中就要求我们`无论是函数还是class声明一个组件，这个组件都必须像纯函数一样，保护它们的props不被修改`



## redux

JavaScript开发的应用程序，已经变得越来越复杂了

JavaScript需要管理的状态越来越多，越来越复杂

这些状态包括服务器返回的数据、缓存数据、用户操作产生的数据等等，也包括一些UI的状态

比如某些元素是否被选中，是否显示 加载动效，当前分页



而管理不断变化的state是非常困难的

1. 状态之间相互会存在依赖，一个状态的变化会引起另一个状态的变化，View页面也有可能会引起状态的变化
2. 当应用程序复杂时，state在什么时候，因为什么原因而发生了变化，发生了怎么样的变化，会变得非常难以控制和追踪



React是在视图层帮助我们解决了DOM的渲染过程，但是State依然是留给我们自己来管理

也就是说React主要负责帮助我们管理视图，state如何维护最终还是我们自己来决定



在React中使用state的方式有很多种

1. 组件定义自己的state
2. 组件之间的通信通过props进行传递
3. 通过Context进行数据之间的共享



而如果多个组件都需要使用某些state，而这些组件在组件树中的位置又相差较远的时候，使用上述几种方式就会使数据传递变得非常复杂，破坏了单向数据流的简洁性， 此时我们就可以使用redux

Redux就是一个帮助我们管理State的容器:  Redux是JavaScript的状态容器，提供了可预测(数据变化可追踪)的状态管理

Redux除了和React一起使用之外，它也可以和其他界面库一起来使用(比如Vue)，并且它非常小(包括依赖在内，只有2kb)



### 核心

| 核心    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| store   | 用于存储状态                                                 |
| action  | Redux要求我们通过action来更新数据<br />所有数据的变化，必须通过派发(dispatch)action来更新<br />action是一个普通的JavaScript对象，用来描述这次更新的type和content(也就是需要携带的参数)<br />强制使用action的好处是可以清晰的知道数据到底发生了什么样的变化，所有的数据变化都是可跟追、可预测的 |
| reducer | reducer应该是一个纯函数<br />reducer做的事情就是将传入的state和action结合起来生成一个新的state |



### 三大原则

1. 单一数据源
   + 整个应用程序的state被存储在一颗object tree中，并且这个object tree只存储在一个 store 中
   + Redux并没有强制让我们不能创建多个Store，但是那样做并不利于数据的维护
   + 单一的数据源可以让整个应用程序的state变得方便维护、追踪、修改
2. state应该是只读的
   + 唯一修改State的方法一定是触发action，不要试图在其他地方通过任何的方式来修改State
   + 这样就确保了View或网络请求都不能直接修改state，它们只能通过action来描述自己想要如何修改state
   + 这样可以保证所有的修改都被集中化处理，并且按照严格的顺序来执行，所以不需要担心race condition(竟态)的问题
3. 使用纯函数来执行修改
   + 通过reducer将 旧state和 actions联系在一起，并且返回一个新的State
   + 随着应用程序的复杂度增加，我们可以将reducer拆分成多个小的reducers，分别操作不同state tree的一部分
   + 但是所有的reducer都应该是纯函数，不能产生任何的副作用
   + 因为reducer内部仍然是将新旧state进行了浅层比较，从而来判断是否需要通知订阅者对应的state已经发生了更新

![image.png](https://s2.loli.net/2022/09/08/6eqxnVHTsy5B2Jh.png)



### 基本使用

**获取redux中定义的状态**

```jsx
const { createStore } = require('redux')

// 初始store
const initalState = {
	name: 'Klaus',
	age: 23
}

// 定义reducer，函数名可以随意，一般见名知意会取名为reducer
function reducer() {

	// reducer需要返回一个新的store
	// redux会自动将新的store和旧的store进行合并
	return initalState
}

// 在创建store的时候，需要传递对应的reducer函数
module.exports =  createStore(reducer)
```

`使用`

```jsx
const store = require('./store')

// 导入store后，通过getState方法获取对应的state数据
console.log(store.getState())
```



**更新数据并获取到最新的状态**

```js
// 需要派发的事件名需要在多个文件中被调用
// 为了保证这多处调用的时候，事件名是一致的
// 所以单独将对应的事件名抽离形成常量
exports.CHANGE_NAME =  'change_name'
exports.ADD_AGE  = 'add_age'
```

```jsx
const { createStore } = require('redux')
const { CHANGE_NAME, ADD_AGE } = require('./constants')

const initalState = {
	name: 'Klaus',
	age: 23
}

// reducer接收两个参数
// 1. 更新前的state，也就是当前state --- 初始的时候，state的值是undefined，也就是没有设置对应状态
// 2. 所需要执行的更新操作描述对象，也就是action
function reducer(state = initalState, action) {
	if (action.type === CHANGE_NAME) {
		return { ...state, name: action.name }
	} else if (action.type === ADD_AGE) {
		return { ...state, age: state.age + action.step }
	}

	// 如果什么都不做或者没有匹配上任何的action操作的时候，返回上一次的state
	// 例如: 第一次初始化操作的时候，就会执行此处逻辑，而对应的state所设置的默认值就是初始state
	return state
}

module.exports =  createStore(reducer)
```

`使用`

```jsx
const store = require('./store')
const { CHANGE_NAME, ADD_AGE } = require('./store/constants')

// 打印初始state
console.log(store.getState())

// 修改name，并打印新state
// 对于store中的数据改变必须通过action
// 当执行对应action的时候，说明状态需要发生改变，会自动调用render方法
store.dispatch({
	type: CHANGE_NAME,
	name: 'Alex'
})
console.log(store.getState())

// 修改age, 并打印新state
store.dispatch({
	type: ADD_AGE,
	step:  10
})
console.log(store.getState())
```



但是，在上述示例中，每执行一次action操作的时候，就需要自己手动去获取对应的最新的store，这部分逻辑是重复的

为此redux提供了subscribe方法，专门用于订阅store的更新

```jsx
const { createStore } = require('redux')
const { CHANGE_NAME, ADD_AGE } = require('./constants')

const initalState = {
	name: 'Klaus',
	age: 23
}

function reducer(state = initalState, action) {
	// reducer中的逻辑使用if进行编写的可读性没有使用switch编写来的好
	switch(action.type) {
		case CHANGE_NAME:
		 return { ...state, name: action.name }
	  case ADD_AGE:
			return { ...state, age: state.age + action.step }
		default:
			return state
	}
}

module.exports =  createStore(reducer)
```

```jsx
const store = require('./store')
const { CHANGE_NAME, ADD_AGE } = require('./store/constants')

// 添加订阅 - 内部会对新旧state进行浅拷贝，一旦state发生了更新，就会回调subscribe方法传入的回调函数
// 参数 - 回调函数 - 注意该回调函数不会将store作为参数传入，store需要自己以模块形式进行引入
// 返回值 - 用于取消订阅的处理函数
const unSubscribe = store.subscribe(() => console.log(store.getState()))

store.dispatch({
	type: CHANGE_NAME,
	name: 'Alex'
})

unSubscribe()

store.dispatch({
	type: ADD_AGE,
	step:  10
})
```



在上述示例中，每次调用action的时候, 都需要手动去编写对应的事件描述对象，而且这些事件描述对象中的type可能都是重复的

所以实际开发中，会使用如下方式进行使用

`constants.js --- 多处需要使用事件名，为了避免出错且保持一致，将事件名单独抽离成常量`

```jsx
exports.CHANGE_NAME =  'change_name'
exports.ADD_AGE  = 'add_age'
```



`reducer.js --- 随着业务增加，reducer中的判断逻辑会越来越多，所以单独抽离成一个文件`

```jsx
const { CHANGE_NAME, ADD_AGE } = require('./constants')

const initalState = {
	name: 'Klaus',
	age: 23
}

function reducer(state = initalState, action) {
	switch(action.type) {
		case CHANGE_NAME:
		 return { ...state, name: action.name }
	  case ADD_AGE:
			return { ...state, age: state.age + action.step }
		default:
			return state
	}
}

module.exports = reducer
```



`actionCreater.js --- 单独存放生成对应action的脚本文件`

```jsx
const { CHANGE_NAME, ADD_AGE } = require('./constants')

// 定义函数，该函数返回对应的action描述对象
exports.changeName = name => ({
	type: CHANGE_NAME,
	name
})

exports.addAge = step => ({
	type: ADD_AGE,
	step
})
```



`index.js --- 入口文件`

```jsx
const { createStore } = require('redux')
const reducer = require('./reducer')

module.exports =  createStore(reducer)
```



`使用`

```jsx
const store = require('./store')
const { changeName, addAge } = require('./store/actionCreator.js')

store.subscribe(() => console.log(store.getState()))

store.dispatch(changeName('Alex'))

store.dispatch(addAge(10))
```