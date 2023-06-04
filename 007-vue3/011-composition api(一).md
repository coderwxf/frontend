Options API的一大特点就是在对应的属性中编写对应的功能模块

当我们实现某一个功能时，这个功能对应的代码逻辑会被拆分到各个属性中

当我们组件变得更大、更复杂时，逻辑关注点的列表就会增长，那么同一个功能的逻辑就会被拆分的很分散

这种碎片化的代码使用理解和维护这个复杂的组件变得异常困难，并且隐藏了潜在的逻辑问题

并且当我们处理单个逻辑关注点时，需要不断的跳到相应的代码块中

为此Vue3提供了Composition Api (简称VCA) 将同一个逻辑关注 点相关的代码收集在一起

![IbrvEO.gif](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/158220a666424cd693a89c0f159feb76~tplv-k3u1fbpfcp-zoom-1.image)



options api vs composition api

1. compositon api 相比 options api 有更好的内聚性，可以将相关的逻辑组合在一起
2. composition api更贴近于函数式编程，所以composition api于typescript的结合程度更高
3. composition api因为是一个个的函数，所以更有利于打包工具在打包的时候进行tree-shaking，可以更好的压缩和优化打包后的代码
4. composition api不使用this，避免了this指向不明的问题



## setup

为了开始使用Composition API，我们需要有一个可以实际使用它(编写代码)的地方，这个位置就是 setup 函数

`setup函数本质上也是一个vue的option`, 只不过这个选项强大到我们可以用它来替代之前所编写的`大部分`其他选项

setup的执行时机位于beforeCreate和created之间

`tips: 在setup函数中是不绑定this的`



setup函数存在两个参数

| 参数    | 说明                                                         |
| ------- | ------------------------------------------------------------ |
| props   | 父组件传递过来的属性会被放到props对象中，我们在setup中如果需要使用，那么就可以直接 通过props参数获取<br />对于定义props的类型，我们还是和之前的规则是一样的，在props选项中定义<br />在template中依然是可以正常去使用props中的属性 |
| context | setup执行上下文(SetupContext), 值是一个对象，包含下边三个属性:<br />attrs:所有的非prop的attribute<br />slots:父组件传递过来的插槽（在以渲染函数返回的时候使用）<br />emit:当我们组件内部需要发出事件时会用到emit |



`setup的返回值`

setup的返回值可以在模板template中被使用， 可以通过setup的返回值来替代data选项

```vue
<template>
	<div>
		<!-- 在vue的template模板中，ref对象在使用的时候不需要通过value属性，vue在解析模板的时候会自动进行解包操作 -->
		<div>{{ count }}</div>
		<button @click="increment">+1</button>
	</div>
</template>

<script>
import { ref } from 'vue'

export default {
	setup() {
		// 在setup函数中定义的数据默认不是响应式的
		// 需要将其使用ref或reactive包裹后，转变为响应式数据
		let count = ref(0);

		// 因为count是一个ref对象，所以获取值的时候需要使用ref对象的value属性
		const increment = () =>  count.value++

		// 只有setup函数返回的状态和方法才可以在template中使用
		return {
			count,
			increment
		}
	}
}
</script>
```



## 响应式数据

默认情况下，vue中定义的数据并不是响应式的，vue会认为他们仅仅用于展示使用

如果希望这些数据被vue所劫持，被搜集成对应的依赖，当数据发生改变时，所有收集到的依赖都进行对应的响应式操作

那么就需要将对应的数据转换为响应式数据



### reactive

```js
import { reactive } from 'vue'

export default {
	setup() {
		// reactive函数可以将一些复杂类型数据转换为响应式数据
		// 所以reactive方法的参数必须是复杂类型数据，如数组，对象等
    // 事实上，我们编写的data选项，也是在内部交给了reactive函数将其编程响应式对象
    // reactive返回的是一个深层代理对象，其内部是使用递归依次对值为对象的属性和自身进行proxy代理(proxy代理是一种浅层代理)
		const userInfo = reactive({
			name: 'Klaus',
			age: 24
		})

		return {
			userInfo
		}
	}
}
```



### ref

reactive API对传入的类型是有限制的，它要求我们必须传入的是一个对象或者数组类型

如果我们传入一个基本数据类型(String、Number、Boolean)会报一个警告

为了可以将基本数据类型转换为响应式数据，vue提供了另一个函数，即ref函数

ref 会返回一个可变的响应式对象，该对象作为一个 **响应式的引用** 维护着它内部的值，这就是ref名称的来源

如果我们需要获取到对应的值，可以通过ref.value

因为reactive使用proxy进行代理操作，而proxy所代理的内容必须是复杂数据类型

为此 ref 在本质上为 `ref(xxx) ---> reactive({ value: xxx })`



注意:

1. 在模板中引入ref的值时，Vue会自动帮助我们进行解包操作，所以我们并不需要在模板中通过 ref.value 的方式来使用
2. 在 setup 函数内部，它依然是一个 ref引用， 所以对其进行操作时，我们依然需要使用 ref.value的方式



```vue
<template>
	<div>
		{{ count }}
	</div>
</template>

<script>
import { ref } from 'vue'

export default {
	setup() {
		// ref的参数即可以是一个基本数据类型数据
		// 也可以是一个复杂数据类型数据
		let count = ref(100)

		return {
			count
		}
	}
}
</script>
```





模板中的解包是浅层的解包

```vue
<template>
	<div>
		<div>
			<span>{{ count }}</span>
			<!-- 在修改的时候，可以自动解包 -->
			<button @click="count++">+1</button>
		</div>

		<div>
			<!-- 在使用的时候可以自动解包 -->
			<span>{{ counter.count }}</span>
			<!-- 修改的时候，不会自动解包，需要手动进行解包操作 -->
			<button @click="counter.count.value++">+1</button>
		</div>
	</div>
</template>

<script>
import { ref } from 'vue'

export default {
	setup() {
		let count = ref(100)
		let count1 = ref(300)
		const counter = { count: count1 }

		return {
			count,
			counter
		}
	}
}
</script>
```



### readonly

通过reactive或者ref可以获取到一个响应式的对象，但是某些情况下，我们传入给其他地方(组件)的这个响应式对象希望在另外一个地方(组件)被使用，但是不能被修改，比如作为组件props传递给另一个组件的时候，这个时候就可以使用readonly

readonly会返回原始对象的只读代理

+ readonly方法返回的依然是一个Proxy，这是一个proxy的set方法被劫持，并且不能对其进行修 改
+ readonly返回的对象不可以修改，但是原始对象依旧可以被修改
+ readonly方法可以被传递的参数有 `普通对象，ref对象，reactive对象`

`父组件`

```vue
<template>
	<div>
		<Cpn :userInfo="userInfo" @change="change" />
	</div>
</template>

<script>
import { readonly, ref } from 'vue'
import Cpn from './components/Cpn'

export default {
	// 引入的组件依旧需要在这里进行注册
	components: {
		Cpn
	},

	setup() {
		const userInfo = ref({
			name: 'Klaus',
			age: 23
		})

		// 子组件emit自定义方法传递的参数会依次被传入过来
		const change = (key, value) => userInfo.value[key] = value

		return {
			// 使用readonly方法可以在语法层面阻止子组件来修改父组件数据，从而保证单向数据流
			userInfo: readonly(userInfo),
			change
		}
	}
}
</script>
```



`子组件`

```vue
<template>
	<div>
		<div>name: {{ userInfo.name }}</div>
		<div>age: {{ userInfo.age }}</div>
		<button @click="changeName">changeName</button>
	</div>
</template>

<script>
export default {
	// composition中接收props的方法和options中是一致的
	props: {
		userInfo: {
			type: Object,
			required: true
		}
	},

	emits: ['change'],

	setup(props, { emit }) {
		// 可以通过emit来向父组件触发对应的事件
		// 参数1 为 事件名
		// 之后的参数 依次为 需要传递给对应事件的参数列表
		const changeName = () => emit('change', 'name', 'Alex')


		return {
			userInfo: props.userInfo,
			changeName
		}
	}
}
</script>
```



### 其它方法

| 方法            | 说明                                                         |
| --------------- | ------------------------------------------------------------ |
| isProxy         | 对象是否是由 reactive 或 readonly创建的 proxy                |
| isReactive      | 对象是否是由 reactive创建的响应式代理<br />如果该代理是 readonly 建的，但包裹了由 reactive 创建的另一个代理，它也会返回 true |
| isReadonly      | 对象是否是由 readonly 创建的只读代理                         |
| toRaw           | 返回 reactive 或 readonly 代理的原始对象<br />不建议保留对原始对象的持久引用。请谨慎使用 |
| shallowReactive | 创建一个响应式代理，它跟踪其自身 property 的响应性<br />但不执行嵌套对象的深层响应式转换 (深层还是原生对象) |
| shallowRef      | 创建一个浅层的ref<br />只有修改整个ref值的时候才会触发响应式，修改ref中某个属性值并不会触发对应的响应式 |
| customRef       | 自定义ref                                                    |



#### customRef

创建一个自定义的ref，并对其依赖项跟踪和更新触发进行显示控制

1. 参数是一个工厂函数，应该返回一个带有 get 和 set 的对象

2. 该工厂函数接受 track 和 trigger 函数作为参数
   + 当使用该ref的时候，应该调用track方法以收集对应的依赖
   + 当设置该ref的值时，应该执行trigger方法以更新所有依赖所使用的值

`使用示例`

```js
import { customRef } from 'vue'

// 使用示例 -> useDebounceRef('Hello Wolrd', 1000)
export default function(value, delay = 2000) {
  // 返回一个自定义的ref实例对象
  return customRef((track, trigger) => {
    let timer = null

    return {
      set(newValue) {
        if (timer) {
          clearTimeout(timer)
        }

        // 设置值
        timer = setTimeout(() => {
          value = newValue // 新值覆盖旧值
          trigger() // 触发更新
        }, delay)
      },

      get() {
        // 取值的时候，调用track函数，vue会自动将使用的部分收集为value的依赖
        // 在vue中所有使用某一个值的地方(这里以value为例)
        // 被称为value的依赖(dependencies 简写为 deps），或者说是value的副作用
        track()
        return value
      }
    }
  })
}
```



## toRefs

对于reactive对象进行解构的时候，默认情况下，解构出来的对象并不是响应式的

如果我们希望解构出来的对象也是响应式的对象，可以使用toRefs方法

```js
<template>
	<div>
		{{ name }} - {{ age }}
		<button @click="change">change</button>
		<button @click="changeName">change name</button>
	</div>
</template>

<script>
import { toRefs, reactive } from 'vue'

export default {
	setup() {
		const userInfo = reactive({
			name: 'Klaus',
			age: 23
		})

		// 将一个响应式对象转换为一个普通对象，这个普通对象的每个属性都是指向源对象相应属性的 ref
    // 每个单独的 ref 都是使用 toRef() 创建的
    // toRefs 在调用时只会为源对象上可枚举属性创建 ref
		const { name, age } = toRefs(userInfo)

		// toRefs相当于已经在state.name和ref.value之间建立了 链接，任何一个修改都会引起另外一个变化
		const change = () => name.value = 'Alex'
		const changeName = () => userInfo.name = 'Alex'

		return {
			name,
			age,
			change,
			changeName
		}
	}
}
</script>
```



如果我们只希望转换一个reactive对象中的属性为ref, 那么可以使用toRef的方法:

```js
import { toRef, reactive } from 'vue'

export default {
	setup() {
		const userInfo = reactive({
			name: 'Klaus',
			age: 23
		})


		// toRef返回的也是一个新的ref对象
		const name = toRef(userInfo, 'name')

		const change = () => name.value = 'Alex'
		const changeName = () => userInfo.name = 'Alex'

		return {
			name,
			change,
			changeName
		}
	}
}
```



### ref的一些其它方法

| 方法       | 说明                                                         |
| ---------- | ------------------------------------------------------------ |
| unref      | 如果参数是一个 ref，则返回内部值，否则返回参数本身<br />val = isRef(val) ? val.value : val 的语法糖函数 |
| isRef      | 判断值是否是一个ref对象                                      |
| shallowRef | 创建一个浅层的ref对象                                        |
| triggerRef | 手动触发和 shallowRef 相关联的副作用                         |



```vue
<template>
	<div>
		<div>{{ info.name }}</div>
		<button @click="changeName">change</button>
	</div>
</template>

<script>
import { shallowRef, triggerRef } from 'vue'

export default {
	setup() {
		// shallowRef({ name: 'Klaus' }) -> { value: { name: 'Klaus' } }
		// 只有value属性值发生改变的时候才会触发自动更新，也就是说只有给整个对象重新赋值的时候，才会触发自动更新
		// 单单修改value属性对应的值的时候是不会触发自动更新的，但是可以使用triggerRef来手动触发对应的更新
		const info = shallowRef({ name: 'Klaus' })

		const changeName = () => {
			info.value.name = 'Alex'

			// 手动触发更新
			triggerRef(info)
		}

		return {
			info,
			changeName
		}
	}
}
</script>
```