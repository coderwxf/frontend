Pinia(发音为/piːnjʌ/，如英语中的“peenya”)是最接近piña(西班牙语中的菠萝)的词

Pinia开始于大概2019年，最初是作为一个实验为Vue重新设计状态管理，让它用起来像组合式API(Composition API)

从那时到现在，最初的设计原则依然是相同的，并且目前同时兼容Vue2、Vue3，也并不要求你使用Composition API

Pinia本质上依然是一个状态管理的库，用于跨组件、页面进行状态共享

Pinia 最初是为了探索 Vuex 的下一次迭代会是什么样子，结合了 Vuex 5 核心团队讨论中的许多想法

最终，团队意识到Pinia已经实现了Vuex5中大部分内容，所以最终决定用Pinia来替代Vuex

 与 Vuex 相比，Pinia 提供了一个更简单的 API，具有更少的限制，提供了 Composition-API 风格的 API

最重要的是，在与 TypeScript 一起使用时具有可靠的类型推断支持

![image.png](https://s2.loli.net/2022/08/31/H8VmnqIKbwlY9Os.png)



## 基本使用

```shell
npm install pinia
```



`/src/stores/index.js`

```js
import { createPinia } from 'pinia'

// pinia在解析的时候，其实际结构和vuex一样会存在一个'根节点'
// 但是在定义的时候，这个根节点不需要我们来进行生成, 只需要定义对应的store即可
// pinia会自动将这些模块挂载到对应的'根节点'下
export default createPinia()
```



`/src/stores/counter.js`

```js
import { defineStore } from 'pinia'

// 使用defineStore定义store
// 参数一 - store的唯一name - Pinia 使用它来将 store 连接到 devtools
// 参数二 - 配置对象
// defineStore会返回一个hook函数，通过调用该hook函数，就可以创建对应的store实例
export default defineStore('counter', {
	// 用于定义共享的数据 - 定义方式和vuex一致
  // 在定义state的时候，推荐使用箭头函数
  // 以便于pinia在typescript环境下，给予更为友好的提示
	state: () => ({
		count: 100
	})
})
```



`main.js`

```js
import { createApp } from 'vue'
import App from './App.vue'
import pinia from './stores'

createApp(App).use(pinia).mount('#app')
```



`APP.vue`

```vue
<template>
	<div>
		<!-- 在使用对应state的时候，不需要在xxx.state.xxx 直接使用即可 -->
		{{ counterStore.count }}
		<button @click="changeCount">change count</button>
	</div>
</template>

<script setup>
  // pinia defineStore 返回的hook一般使用useXXX作为命名方案，这是约定的规范
	import useCounter from './stores/counter'

	const counterStore = useCounter()

	function changeCount() {
		// 如果需要修改pinia中的数据，直接修改就可以了
		// pinia中根本就不存在mutation这样的概念
		counterStore.count++
	}
</script>
```



## store

一个 Store (如 Pinia)是一个实体，它会持有为绑定到你组件树的状态和业务逻辑，也就是保存了全局的状态

它有点像始终存在，并且每个人都可以读取和写入的组件

你可以在你的应用程序中定义任意数量的Store来管理你的状态

store有三个核心概念: state、getters、actions 等同于组件的data、computed、methods

一旦 store 被实例化，你就可以直接在 store 上访问 state、getters 和 actions 中定义的任何属性



注意 创建处理的Store本质上是一个代理对象，所以Store获取到后不能被解构，会失去对应的响应式

为了从 Store 中提取属性同时保持其响应式，您需要使用Pinat提供的storeToRefs方法或vue提供的toRefs方法

```js
const { count } = storeToRefs(counterStore)
// 等价于
const { count } = toRef(counterStore.state)
```



### state

state 是 store 的核心部分，因为store是用来帮助我们管理状态

在 Pinia 中，状态被定义为返回初始状态的函数

默认情况下，您可以通过 store 实例访问状态来直接读取和写入状态

```js
function changeCount() {
  counterStore.count++
}
```



#### 重置state

可以通过调用 store 上的 $reset() 方法将状态 重置 到其初始值

```js
// 对于vue，vue-router, vuex, pinia而言
// 内部提供的属性和方法会使用$前缀，以区分自定义属性和方法
counterStore.$reset()
```



#### 改变state

除了直接用 store.counter++ 修改 store，你还可以调用 $patch 方法同时修改多个state属性

```js
// $patch的参数即可以是对象形式也可以是一个接收state为参数的函数
// 当$patch方法接收一个函数参数的时候，其对于复杂数据类型的操作会更方便
userInfoStore.$patch({
  name: 'Alex',
  age: 18
})

userInfoStore.$patch(state => {
  state.friends.push({ name: 'Alex', age: 32 })
  state.name = 'Steven'
})
```



#### 替换state

可以通过将其 $state 属性设置为新对象来替换 Store 的整个状态

```js
// 获取store中的state对象
console.log(userInfoStore.$state)

// 替换store中的state对象
// 在替换的时候，pinia内部会劫持我的替换操作
// 在pinia内部会使用我传入的对象和原始的state对象进行Object.assign操作
// 这样做的目的是为了 防止某些属性在template中使用，但是因为state被替换而不存在对应的值
userInfoStore.$state = {
  name: 'Steven'
}
```



### getters

Getters相当于Store的计算属性

它们可以用 defineStore() 中的 getters 属性定义

getters中可以定义接受一个state作为参数的函数

```js
import { defineStore } from 'pinia'
import useCounter from './counter'

export default defineStore('userInfo', {
	state() {
		return {
			name: 'Klaus',
			age: 23,
			firends: [
				{
					name: 'Klaus',
					age: 23
				},
				{
					name: 'Alex',
					age: 32
				}
			]
		}
	},

	getters: {
		// 基本使用
		doubleAge(state) {
			// getter内部有对应的this，其值和传入的state是相同的对象
      // 可以通过this来访问到当前store实例的所有其他属性
			console.log(this === state)

			return state.age * 2
		},

		// 在一个getter中使用本store中的另一个getter
		doubleCountAddOne(state) {
			return this.doubleAge + 1
		},

		// getter返回一个函数，用于接收参数
		findFriendByName(state) {
			return name => state.firends.find(firend => firend.name === name)
		},

		// getter中调用另一个state中的getter
		joinAgeAndCount(state) {
			const counterStore = useCounter()
			return state.age + counterStore.count
		}
 	}
})
```

```vue
<template>
	<div>
		<div>{{ userInfoStore.doubleAge }}</div>
		<div>{{ userInfoStore.doubleCountAddOne }}</div>
		<div>{{ userInfoStore.findFriendByName('Steven') }}</div>
		<div>{{ userInfoStore.joinAgeAndCount }}</div>
	</div>
</template>

<script setup>
	import useUserInfoStore from './stores/userInfoStore'

	const userInfoStore = useUserInfoStore()
</script>
```



### actions

Actions 相当于组件中的 methods

可以使用 defineStore() 中的 actions 属性定义，并且它们非常适合定义业务逻辑

和getters一样，在action中可以通过this访问整个store实例的所有操作

并且Actions中是支持异步操作的，在actions中，我们可以编写异步函数，在函数中使用await

```js
import { defineStore } from 'pinia'

// 模拟异步请求
function asynFn() {
	return new Promise(resolve => {
		setTimeout(() => resolve('asynFn result'), 2000)
	})
}

export default defineStore('userInfo', {
	state() {
		return {
			name: 'Klaus',
			age: 23
		}
	},

	actions: {
		incremnetAge(num) {
			this.age += num
		},
		async asyncFoo() {
			return await asynFn()
		}
	}
})
```

```vue
<template>
	<div>
		<div>{{ userInfoStore.age }}</div>
		<button @click="incrementAge">click me</button>
		<button @click="asynFnTest">asyn fun test</button>
	</div>
</template>

<script setup>
	import useUserInfoStore from './stores/userInfoStore'

	const userInfoStore = useUserInfoStore()

	function incrementAge() {
		userInfoStore.incremnetAge(10)
	}

	async function asynFnTest() {
		const res = await userInfoStore.asyncFoo()
		console.log(res)
	}
</script>
```