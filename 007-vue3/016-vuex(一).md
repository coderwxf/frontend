在开发中，我们会的应用程序需要处理各种各样的数据，当数据变得十分庞大时 或者 需要进行状态传递的组件关系过于松散时，数据的传递和维护就变得不是那么方便，所以我们需要一个单独的地方，用于管理和维护这样的应用程序状态数据，对于这些数据的管理我们就称之为`状态管理`

在一个组件中，我们可以通过如下方式进行简单的状态管理

+ 在组件中我们定义data或者在setup中返回使用的数据，这些数据我们称之为state
+ 在模块template中我们可以使用这些数据，模块最终会被渲染成 DOM，我们称之为View
+ 在模块中我们会产生一些行为事件，处理这些行为事件时，有可能会修改state，这些行为事件我们称之为actions

![image.png](https://s2.loli.net/2022/08/27/ZAwnPvelNykap7H.png) 



但是随着应用程序的越来越复杂，在组件中管理的状态会变得越来越复杂，越来越庞大，且状态之间相互会存在依赖，一个状态的变化会引起另一个状态的变化，View页面也有可能会引起状态的变化

其次对于父子组件进行状态管理的时候，可以通过props或emits进行状态的管理操作和维护

但是对于没有直接关联关系的组件，例如那些兄弟组件，或距离很远的组件，进行状态的传递就变得非常不容易，而且非常容易破坏单向数据流的简便性

所以当应用程序复杂时，state在什么时候，因为什么原因而发生了变化，发生了怎么样的变化，会变得非常难以控制和追踪

为此我们需要将组件的内部状态抽离出来，以一个全局单例的方式来管理

+ 在这种模式下，我们的组件树构成了一个巨大的 “试图View”
+ 不管在树的哪个位置，任何组件都能获取状态或者触发行为

这个全局单例就是全局状态管理，常见的全局状态管理库有 redux，vuex，pinia等

![image.png](https://s2.loli.net/2022/08/27/apWNrveLDwU3FTQ.png) 



## state

![image.png](https://s2.loli.net/2022/08/27/AtJoHWXzm9QlKun.png) 



### 基本使用

`安装`

```shell
npm install vuex
```

`创建store`

每一个Vuex应用的核心就是store(仓库)，store本质上是一个容器，它包含着你的应用中大部分的状态(state)

store中的状态是响应式的，当组件从store中读取状态的时候 如果数据发生变化 则相应的组件也会重新渲染

`/src/stores/index.js`

```js
import { createStore } from 'vuex'

// 创建一个store(仓库)
export default createStore({
	// state用于存储store中的数据状态
	// 其定义形式和组件中的data是一致的
	state() {
		return {
			counter: 100
		}
	},
	// 在vuex中修改state只能通过mutations
	// 只有这样对应的修改才会被devtools所监听到
	mutations: {
    // mutation会将对应的state作为参数传入
		increment(state) {
			state.counter++
		}
	}
})
```



`main.js`

```js
import { createApp } from 'vue'
import App from './App.vue'

import router from './routes'
import store from './stores'

// vuex和vue-router一样是vue提供的官方插件
// 所以使用的时候，需要使用use在全局进行挂载
createApp(App).use(router).use(store).mount('#app')
```



`App.vue`

```vue
<template>
  <div>
    <!-- 在template和options api中如果需要使用vuex中的数据,可以通过$store -->
    <h2>计数器: {{ $store.state.counter }}</h2>
    <button @click="increment">+1</button>
  </div>
</template>

<script setup>
  import { toRefs } from 'vue'
  // 如果需要在composition api中使用对应vuex中的数据，需要使用对应的hook函数
  import { useStore } from 'vuex'

  // useStore 只能在script setup顶层使用
  const store = useStore()

  const increment = () => {
    // 在setup函数中获取对应的state值
    // 从vuex中取出的状态是一个proxy代理，所以如果需要获取其中具体的属性，需要使用toRef或toRefs
    const { counter } = toRefs(store.state)
    console.log(counter)

    // 在setup函数中触发对应的mutation
    store.commit('increment')
  }
</script>
```



### Vuex VS 单纯的全局对象

1. Vuex的状态存储是响应式的
   + 当Vue组件从store中读取状态的时候，若store中的状态发生变化，那么相应的组件也会被更新
2. 不能直接改变store中的状态
   + 改变store中的状态的唯一途径就显示**提交** **(commit) mutation**
   + 这样使得我们可以方便的跟踪每一个状态的变化，从而让我们能够通过一些工具帮助我们更好的管理应用的状态



### 单一状态树

vuex采用SSOT，Single Source of Truth (单一数据源 或者 单一状态树)

即用一个对象就包含了全部的应用层级的状态，不同逻辑块中的状态采用module方式来进行拆分



### 辅助函数

#### mapState

默认情况下，如果我们需要在组件中获取store中对应的属性的时候，需要借助store对象

但是如果需要使用的数据状态过多的情况下，就需要重复编写类似于`$store.state.xxx`之类的重复代码

为此vuex提供了mapState辅助函数来简化我们的代码



`在vue2中的写法如下`

```js
<script>
import { mapState } from 'vuex'

export default {
	computed: {
		// mapState返回的是一个个的计算属性函数
		/**
		 * 类似于
		 * name() {
		 * 	return this.$store.name
		 * }
		 */
		...mapState(['name', 'age'])
	}
}
</script>
```

```js
import { mapState } from 'vuex'

export default {
	computed: {
		// mapState也支持函数式写法
		// 每一个key对应一个函数，在执行该函数的时候会默认将state作为参数传入
    // 使用对象形式，可以给取出的状态起别名，避免和组件内部状态命名冲突
		...mapState({
			name: state => state.name,
			age: state => state.age
		})
	}
}
```



`在vue3中的写法如下:`

```js
import { computed } from 'vue'
import { useStore, mapState } from 'vuex'

// mapState中导出的依旧是和之前一样的用于计算属性的函数
const { name: nameFn, age: ageFn } = mapState(['name', 'age'])

const store = useStore()

// 因为导出的函数内部依旧是使用this.$store来进行执行
// 而setup函数内部没有this，所以需要手动绑定正确的this
const name = computed(nameFn.bind({ $store: store }))
const age = computed(ageFn.bind({$store: store}))
```

对上述代码进行封装

```js
import { computed } from 'vue'
import { useStore, mapState } from 'vuex'

export function useState(mapper) {
	const store =  useStore()
	const computedObj = mapState(mapper)
	const newComputedObj = {}
	const keys = Object.keys(computedObj)

	for(let key of keys) {
		newComputedObj[key] = computed(computedObj[key].bind({ $store: store }))
	}

	return newComputedObj
}
```

`使用`

```js
import { useState } from './hooks/useState'
const { name, age } = useState(['name', 'age'])
```



但是mapState本质上还是给options api提供的函数，所以在composition api中这么使用比较依旧是十分麻烦的

推荐在composition api中 通过如下方式使用state

```js
import { toRefs } from 'vue'
import { useStore } from 'vuex'

const store = useStore()

// 我们的需求是
// 1. 方便的从vuex中取出我们所需要的状态
// 2. 对应的状态是响应式的
const { name, age } = toRefs(store.state)
```



## getters

某些属性我们可能需要经过变化后来使用，这个时候可以使用getters，所以getters类似于vuex中的计算属性

`vue`

```vue
<template>
	<div>
		<div>{{ $store.getters.message }}</div>
		<div>{{ $store.getters.findFriendByName('Alex') }}</div>
	</div>
</template>

<script setup>
	import { reactive, toRefs } from 'vue'
	import { useStore } from 'vuex'

	const store = useStore()

	// store.state是一个proxy代理对象
	// 但是store.getters只是一个普通对象，所以需要使用reactive进行包裹，使其成为一个响应式对象
	const { message } = toRefs(reactive(store.getters))
  // 也可以使用 const message = computed(() => store.getters.message)
	console.log(message)
</script>
```

`js`

```js
import { createStore } from 'vuex'

export default createStore({
  state() {
    return {
      name: 'Klaus',
      age: 23,
      friends: [
        {
          name: 'Alex',
          age: 24
        },
        {
          name: 'Steven',
          age: 43
        }
      ]
    }
  },
  getters: {
    doubleAge(state) {
      return state.age * 2
    },

    // getters会传入两个参数
    // 参数一 - state
    // 参数二 - getters 也就是说在getters中可以使用其它的getters
    message(state, getters) {
      return `my name is ${state.name}, age is ${getters.doubleAge}`
    },

    // 如果getters中的某些数据需要外部来决定，可以使用函数柯里化的方式由外部来传入对应的参数
    findFriendByName(state) {
      return name => state.friends.find(friend => friend.name === name) ?? {}
    }
  }
})
```



### mapGetters

和state一样，vuex为getters提供了对应的辅助函数`mapGetters`

`options api`

```js
import { mapGetters } from 'vuex'

export default({
  computed: {
    ...mapGetters(['message', 'findFriendByName'])
  }
})
```

```js
import { mapGetters } from 'vuex'

export default({
  computed: {
    // mapGetter的对象写法不需要将值写成函数形式
    ...mapGetters({
      message: 'message',
      findFriendByName: 'findFriendByName'
    })
  }
})
```



`composition api`

```vue
<template>
	<div>
		<div>{{ message }}</div>
		<div>{{ findFriendByName('Alex') }}</div>
	</div>
</template>

<script setup>
	import { computed } from 'vue'
	import { mapGetters, useStore } from 'vuex'

	const store = useStore()

	const { message: messageFn, findFriendByName: findFriendByNameFn } = mapGetters(['message', 'findFriendByName'])

	const message = computed(messageFn.bind({ $store: store, $getters: store.getters }))
	const findFriendByName = computed(findFriendByNameFn.bind({ $store: store }))
</script>
```

同样的，在实际使用中，推荐和toRefs一起结合进行使用

```js
import { reactive, toRefs } from 'vue'
import { useStore } from 'vuex'

const store = useStore()
const { message, findFriendByName } = toRefs(reactive(store.getters))
```



## mutation

在vuex中修改state的唯一方法是提交对应的mutation

`mutation-type.js`

```js
// mutation 事件名需要在多处使用
// 为了以后修改mutation事件名方便，推荐将对应的事件名抽离成一个单独的常量
export const CHANGE_AGE = 'changeAge'
```



`vuex`

```js
import { createStore } from 'vuex'
import { CHANGE_AGE } from './mutations-types'

export default createStore({
	state() {
		return {
			name: 'Klaus',
			age: 23,
			friends: [
				{
					name: 'Alex',
					age: 24
				},
				{
					name: 'Steven',
					age: 43
				}
			]
		}
	},
	// mutations类似于methods选项
	// 在vuex中，所有对于state的操作都必须通过mutations
	mutations: {
		[CHANGE_AGE](state, payload) {
			state.age = payload.age
		}
	}
})
```



`vue`

```vue
<template>
	<div>
		<div>{{ $store.state.age }}</div>
		<button @click="changeAge">click me</button>
	</div>
</template>

<script setup>
	import { useStore } from 'vuex'
	import { CHANGE_AGE } from '/src/stores/mutations-types'

	const store = useStore()

	function changeAge() {
		// 在提交mutation事件的时候
		// 参数一 为事件名
		// 参数二 参数对象 - 原则上可以是任意数据类型
		//              - 但是为了方便传入多个参数，所以一般会传入一个参数对象
		store.commit(CHANGE_AGE, {
			age: 18
		})
    
    // 这是vuex支持的另一个对象风格的修改mutations的方式
    store.commit({
			type: CHANGE_AGE,
			age: 18
		})
	}
</script>
```



### mapMutations

有时候，我们希望从vuex直接导入对应的mutations后就可以直接使用

而不是使用自己的函数在器外层进行包裹后才可以使用 ( 例如上述案例中changeAge对CHANGE_AGE进行了包裹 )

俄日此vuex提供了对应的辅助函数mapMutations

`options api`

```js
methods: {
 // mapMutations的参数同样支持数组形式和对象形式
 // 对象形式的时候，对应的属性值是对应的函数名即可
 // mapMutations返回的是普通的函数，不再是那些computed函数
 ...mapMutations({
		addNumber: ADD_NUMBER,
 }) ,
 // 所有的辅助函数，都是可以多次使用的
 // 因为他们返回的都是对象，解构后合并到同一个对象中而已，所以他们并不冲突
...mapMutations(['increment', 'decrement'])
}
```



`composition api`

```vue
<template>
	<div>
		<div>{{ $store.state.age }}</div>
		<!-- 实际传参的位置 -->
		<button @click="changeAge({ age: 19 })">click me</button>
	</div>
</template>

<script setup>
	import { useStore, mapMutations } from 'vuex'
	import { CHANGE_AGE } from '/src/stores/mutations-types'

	const store = useStore()

	// 使用mapMutations导出的对象是普通对象
	// 所以在解构的时候，需要使用实际名称，不可以使用常量
  // store只向外暴露了getters和state，所以这里不可以使用toRefs直接进行解构
	let { changeAge } = mapMutations([CHANGE_AGE])

	// 解构出的对象内部默认使用的依旧是this.$store
	// 所以依旧需要手动绑定对应的this
	changeAge = changeAge.bind({ $store: store })
</script>
```


