## actions

vuex将数据状态单独抽离出来进行管理

那也就意味着需要交给vuex管理的状态对应的逻辑就应该放到vuex中进行管理，而不是交给对应的组件

而mutations有一个非常重要的原则，在mutations中定义的函数必须是同步函数

因为只有这样，对应的state状态的改变才会被devTools所监听识别

如果是异步函数，需要编写在action中，等待异步函数执行完毕后，再提交一个mutation来修改对应的state

actions的特点:

1. Action提交的是mutation，而不是直接变更状态
2. Action可以包含任意异步操作, 而mutations不能



actions中存在一个参数context，context是一个类似于store实例的对象

```js
import { createStore } from 'vuex'
import { CHANGE_AGE } from './mutations-types'

export default createStore({
	state() {
		return {
			name: 'Klaus',
			age: 23
		}
	},

	mutations: {
		changeName(state, { name }) {
			state.name = name
		}
	},

	actions: {
		// action方法通常以action结尾
		// 以区分mutations中对应的方法
		changeNameAction(ctx, payload) {
			// console.log(ctx.getters) // 获取对应的getters
			// console.log(ctx.state) // 获取对应的state

			// 提交对应的mutation
			ctx.commit('changeName', payload)
      
      /*
      	actions 也支持对象形式提交
      	
      	ctx.commit({
          type: 'changeName',
          ...payload
        })
      */
		},
    
    changeAgeAction(ctx, payload) {
			// 很多时候, action方法会返回一个promise
			// 或者使用async异步函数 - async异步函数会默认返回一个promise
			// 这样组件内部在调用的时候，就可以通过then方法或await关键字来监听异步函数什么时候执行完成
			return new Promise(resolve => {
				ctx.commit('changeAge', payload)
				resolve()
			})
		}
	}
})
```



`使用`

```vue
<template>
	<div>
		<div>{{ $store.state.name }}</div>
		<button @click="changeName">click me</button>
	</div>
</template>

<script setup>
	import { useStore } from 'vuex'

	const store = useStore()

	function changeName() {
		store.dispatch('changeNameAction', {
			name: 'Alex'
		})
	}
</script>
```



### mapActions

同样为了避免我们重复编写this.$store，vuex提供了对应的辅助函数`mapActions`

```vue
<template>
	<div>
		<div>{{ $store.state.name }}</div>
		<!-- 实际传参的位置 -->
		<button @click="changeName({ name: 'Alex' })">click me</button>
	</div>
</template>

<script setup>
	import { useStore, mapActions } from 'vuex'

	const store = useStore()

	// mapActions 同样支持数组写法和对象写法
	// const { changeNameAction } = mapActions(['changeNameAction'])

	// 对象语法的属性值依旧是对应的函数名
	const { changeNameAction } = mapActions({
		changeNameAction: 'changeNameAction'
	})

  // mapAction返回的函数内部调用的时候依旧使用的是this.$store
	const changeName = changeNameAction.bind({ $store: store })
</script>
```



## modules

由于使用单一状态树，应用的所有状态会集中到一个比较大的对象，当应用变得非常复杂时，store 对象就有可能变得相当臃肿

为了解决以上问题，Vuex 允许我们将 store 分割成`模块(module)`

每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块



`store/main.js`

```js
// vuex模块需要导出一个对象，里面的配置项也是state，mutations，actions 和 modules
export default {
	state() {
		return {
			name: 'Klaus',
			age: 23
		}
	},

	mutations: {
		changeName(state, { name }) {
			state.name = name
		}
	},

	actions: {
		changeNameAction(ctx, payload) {
			ctx.commit('changeName', payload)
		}
	}
}
```



`store/index.js`

```js
import { createStore } from 'vuex'
import main from './main'

export default createStore({
	// 挂载模块
	modules: {
		main
	}
})
```



`App.vue`

```vue
<template>
	<div>
		<!-- 获取state中的状态，需要使用$store.state[moduleName][xxx]来进行获取 -->
    <!-- 
				但是对于getters，mutations，actions而言，默认都是会被挂载到全局
				对于同名函数，会被合并为一个数组，在触发对应函数的时候，数组中的每一个方法都会被依次执行
				但是这样在vuex中很有可能会引发冲突，所以一般会开启vuex的命名空间
				这样每一个模块中定义的内容就不会和另一个模块中定义的内容产生冲突
		-->
		<div>{{ $store.state.main.name }}</div>
		<button @click="changeName({ name: 'Alex' })">click me</button>
	</div>
</template>

<script setup>
	import { useStore, mapActions } from 'vuex'

	const store = useStore()
	const { changeNameAction } = mapActions(['changeNameAction'])

	const changeName = changeNameAction.bind({ $store: store })
</script>
```



## 命名空间

默认情况下，模块内部的action和mutation 以及getters仍然是注册在**全局的命名空间**中的

这样使得多个模块能够对同一个 action 或 mutation 和 getters 作出响应

如果我们希望模块具有更高的封装度和复用性，可以添加 namespaced: true 的方式使其成为带命名空间的模块

当模块被注册后，它的所有 getter、action 及 mutation 都会自动根据模块注册的路径调整命名

`store/main.js`

```js
// vuex模块需要导出一个对象，里面的配置项也是state，mutations，actions 和 modules
export default {
	namespaced: true,
	state() {
		return {
			name: 'Klaus',
			age: 23
		}
	},

	getters: {
		// 开启模块化后，getters中的rootState会存在所有的state，包括root自身的和所有模块的
		// rootGetter也会存在所有的getters，包括自身的和所有模块的
		doubleAge(state, payload, rootState, rootGetters) {
			console.log(rootState) // => { rootCount: 100, main: { ... } }
			console.log(rootGetters) // => { doubleRootCount: xxx, 'main/doubleCount': xxx }
			return state.age
		}
	},

	mutations: {
		changeName(state, { name }) {
			state.name = name
		}
	},

	actions: {
		// action的参数 context 内部含有以下六个对应属性
		changeNameAction({ commit, dispatch, state, getters, rootState, rootGetters }, payload) {
			commit('changeName', payload)
		}
	}
}
```

`store/index.js`

```js
import { createStore } from 'vuex'
import main from './main'

export default createStore({
	state() {
		return {
			rootCounter: 100
		}
	},
	getters: {
		doubleRootCounter(state) {
			return state.rootCounter
		}
	},
	modules: {
		main
	}
})
```

`App.vue`

```vue
<template>
	<div>
		<!-- 获取state中的状态，需要使用$store.state[moduleName][xxx]来进行获取 -->
		<div>{{ $store.state.main.name }}</div>
		<!-- 开启模块化后，触发对应getters，mutations和actions 需要使用 模块名/对应函数名 -->
		<div>{{ $store.getters['main/doubleAge'] }}</div>
		<button @click="changeName({ name: 'Alex' })">click me</button>
	</div>
</template>

<script setup>
	import { useStore, mapActions } from 'vuex'

	const store = useStore()

	/* 
		// 如果需要在辅助函数中，使用命名空间，辅助函数的参数需要使用对象形式 
		const { changeNameAction } = mapActions({
			changeNameAction: 'main/changeNameAction'
		})
  
  	// 或者在辅助函数的第一个参数传入模块名，第二个参数传入对应的数组形式参数或对象形式参数
  	const { changeNameAction } = mapActions('main', ['changeNameAction'])
	*/
	const changeName = changeNameAction.bind({ $store: store })
</script>
```



```vue
<template>
	<div>
		<div>{{ $store.state.main.name }}</div>
		<div>{{ $store.getters['main/doubleAge'] }}</div>
		<button @click="changeName({ name: 'Alex' })">click me</button>
	</div>
</template>

<script setup>
	import { useStore, createNamespacedHelpers } from 'vuex'

	const store = useStore()

	// 我们也可以通过vuex提供的函数createNamespacedHelpers来生成对应模块特有的辅助函数
	const { mapActions } = createNamespacedHelpers('main')
	const { changeNameAction } = mapActions(['changeNameAction'])
	const changeName = changeNameAction.bind({ $store: store })
</script>
```



### module修改或派发根组件

```js
// 如果我们希望在模块中的action中修改root中的state
// 也就是在模块中触发root的mutations和actions
// 需要在第三个配置参数中开启对应的配置项
commit ("changeRootName", null, { root: true }) ;
dispatch ("changeRootNameAction", null, { root: true })
```



