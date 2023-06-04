## computed

在Composition API中，我们可以在 setup 函数中使用 computed 方法来编写一个计算属性

+ 接收一个getter函数，并为 getter 函数返回的值，返回一个不变的 ref 对象
+ 接收一个具有 get 和 set 的对象，返回一个可变的(可读写)ref 对象

```vue
<template>
	<div>
		<div>	{{ firstname }} + {{ lastname }} = {{ fullname }}	</div>
		<button @click="change">change</button>
	</div>
</template>

<script>
import { ref, computed } from 'vue'

export default {
	setup() {
		const firstname = ref('Klaus')
		const lastname = ref('Wang')

		// 只使用get方法的computed
		// computed方法 返回的也是一个ref实例对象 - computedRefImpl
		// const fullname = computed(() => firstname.value + ' ' + lastname.value)
		const fullname = computed({
			get() {
				return firstname.value + ' ' + lastname.value
			},

			set(v) {
				const names = v.split(' ')
				firstname.value = names[0]
				lastname.value = names[1]
			}
		})

		const change = () => fullname.value = 'Steven Li'

		return {
			firstname,
			lastname,
			fullname,
			change
		}
	}
}
</script>
```



## ref

我们在options api中可以通过$refs的方式去获取组件实例和原生元素

如果我们需要在composition api中实现相同的功能，只需要定义一个ref对象，绑定到元素或者组件的ref属性上即可

```vue
<template>
  <div>
    <div ref="divRef">foo</div>
    <Cpn ref="cpnRef" />
  </div>
</template>

<script>
import { ref, onMounted } from 'vue'
import Cpn from './components/Cpn'

export default {
	components: {
		Cpn
	},
  setup() {
    // 如果ref的参数为空的时候，divRef.value的值默认为undefined
    // 获取元素的时候，一般也可以将ref中的默认值设置为null
    let divRef = ref()
    let cpnRef = ref(null)

    // 元素需要在onMounted生命周期中才可以获取
    onMounted(() => {
      console.log(divRef)
      console.log(cpnRef)
    })

    return {
			divRef,
			cpnRef
    }
  }
}
</script>
```



## 生命周期钩子

在setup中使用生命周期钩子，直接导入的 onX 函数注册生命周期钩子

![](https://s3.bmp.ovh/imgs/2022/08/22/8b73b6b0c0c8391c.png)

在composition api中已经没有了beforeCreate和created这两个生命周期钩子，

在这两个生命周期钩子中需要完成的内容可以直接在setup函数中进行实现

```js
import { onMounted } from 'vue'

export default {
	setup() {
		// 生命周期钩子可以被多次调用和执行
		// 对应的回调会被存入到一个数组中被一次调用
		onMounted(() => console.log('onMounted1'))
		onMounted(() => console.log('onMounted2'))
		onMounted(() => console.log('onMounted3'))
	}
}
```



## provide/inject

`父组件`

```vue
<template>
	<div>
		<Cpn />
	</div>
</template>

<script>
import { provide, ref } from 'vue'
import Cpn from './components/Cpn'

export default {
	components: {
		Cpn
	},

	setup() {
		// 为了使provide提供的值依旧是响应式的
		// 对应的数据需要使用ref或reactive包裹
		const name = ref('Klaus')

		// provide函数有两个参数
		// 参数一  key
		// 参数二 value
		provide('name', name)
	}
}
</script>
```



`子组件`

```vue
<template>
	<div>
		{{ name }} - {{ age }}
	</div>
</template>

<script>
import { inject } from 'vue'

export default {
	setup() {
		// inject函数有两个参数
		// 参数一: key
		// 参数二: 默认值 可以不传
		// ps: 如果是composition api中使用inject函数获取的ref, 在模板中使用的时候会自动解包
		// 但是如果是在options api的inject选项获取的ref, 在模板中并不会自动解包
		const name = inject('name')
		const age = inject('age', 23)

		return {
			name,
			age
		}
	}
}
</script>
```



## 数据侦听

在Composition API中，我们可以使用watchEffect和watch来完成响应式数据的侦听

+ watchEffect:用于自动收集响应式数据的依赖
+ watch:需要手动指定侦听的数据源



### watch

watch的API完全等同于组件watch选项的Property:

+ watch需要侦听特定的数据源，并且执行其回调函数
+ 默认情况下它是惰性的，只有当被侦听的源发生变化时才会执行回调

```vue
<template>
	<div>
    <h2>{{ userInfo.firend.name }}</h2>
    <button @click="change">change</button>
	</div>
</template>

<script>
import { watch, ref } from 'vue'

export default {
	setup() {
    const userInfo = ref({
      name: 'Klaus',
      firend: {
        name: 'Alex'
      }
    })

    // 参数一: 需要监听的变量值
    // 参数二: 回调函数 用于接收新值和旧值
    //        如果是对象，新值和旧值依旧是一致的
    // 参数三: 配置对象
    //     deep 是否开启深度监听
    //     immediate 首次执行时 是否触发watch
    watch(userInfo, (newV, oldV) => {
      console.log(newV, oldV)
    }, {
      immediate: true
    })

    const change = () => userInfo.value.firend.name = 'Steven'

		return {
      userInfo,
      change
		}
	}
}
</script>
```

```js
// watch的第一个参数也可以是一个回调函数
// vue会自动收集第一个函数中的响应式数据作为对应依赖
// 当依赖发送改变的时候，自动执行函数，并将对应结果作为第二个参数的返回值返回

// 因为本例中是对数据进行了浅拷贝，所以newV和oldV获取到的是原始数据不是代理对象
watch(() => ({ ...userInfo.value }), (newV, oldV) => {
  console.log(newV, oldV)
}, {
  deep: true, // 如果参数一为回调函数的时候，默认deep是false，并不会开启深度监听
  immediate: true
})
```

```js
// watch可以监听多个值，多个值以数组形式进行传入
// 对应的新值和旧值 也会被以数组形式进行传入
watch([name, age], ([newName, newAge], [oldName, oldAge]) => {
  console.log(newName, newAge, oldName, oldAge)
})
```

```js
// watch方法返回了一个函数
// 通过该函数可以终止当前监听函数的执行

// watch所返回的新值和旧值是被自动解包后的值
// 所以对于对象而言获取到的是proxy值
// 对于普通数据类型获取到的是普通数据类型本身
const stopWatch = watch(age, v => {
  console.log(v)
  if (v >= 18) {
    stopWatch()
  }
})
```



### watchEffect

```js
// watchEffect参数值是一个函数
// 该函数默认会被执行一次，以将该函数收集为对应响应式数据的副作用函数
// 也就是收集对应的依赖，当对应响应式数据值发生改变的时候，会自动重新执行对应回调函数
// watchEffect函数会返回一个函数，通过该函数可以终止监听
const stopWatch = watchEffect(() => {
  console.log(count.value)

  if (count.value >= 10) {
    stopWatch()
  }
})
```

```js
// watchEffect的第一个参数是回调函数，第二个参数是一个配置对象
// 在配置对象中，我们可以改变副作用函数的执行时机
// 属性flush的默认值为pre，表示的是函数在创建完毕或者更新之前执行
//                      所以在此时无法操作dom
// flush的值修改为post后，表示的是函数在挂载完毕或者更新之前执行
//                      所以此时我们可以对dom进行操作
// flush还支持一个属性 sync，官方不推荐使用，了解即可
watchEffect(() => {
 // 确保dom元素存在后，再执行对应操作
 if（numElem.value）{
   console.log(numElem.value)
 }
}, {
  flush: 'post'
})
```



### watch vs watchEffect

1. watch默认是惰性的，第一次不会执行，而watchEffect默认不是惰性的，必须第一次执行以收集对应的依赖

2. watch在数据发生改变的时候可以获取对应的新值和旧值，而watchEffect只能拿到对应的新值

3. watch可以自定义需要监听的数据源，watchEffect是自动收集对应的数据源，只要数据源中的某一项发生了改变，那么watchEffect的整个回调就会被重新执行



## 示例

`组件`

```vue
<template>
	<div>
		<button @click="btnClick">btn</button>
	</div>
</template>

<script>
import useTitle from './hooks/useTitle'

export default {
	setup() {
		// 调用hook后返回一个新的ref对象
		// 以后如果需要修改title值的时候，直接修改titleRef的value属性值即可
		// 不需要在频繁的调用useTitle
		const { title } = useTitle('首页')

		const btnClick = () => {
			title.value = '新的标题'
		}

		return {
			btnClick
		}
	}
}
</script>
```



`hook`

```js
import { ref, watch } from 'vue'

// hook的名称一般使用use开头
// 所谓hook其实就是对应相同逻辑的代码抽离，所抽离出的一个个的小函数
// hook之所以为函数是因为  避免多次调用的时候出现变量冲突
export default function useTitle(title = '') {
	const titleRef = ref(title)

	watch(titleRef, newV => {
		document.title = newV
	}, {
		immediate: true
	})

	return {
    titleRef
  }
}
```



## script setup

`<script setup>` 是在单文件组件 (SFC) 中使用组合式 API 的`编译时语法糖`

当同时使用 SFC 与组合式 API 时则推荐该语法:

+ 更少的样板内容，更简洁的代码
+ 能够使用纯 Typescript 声明 prop 和抛出事件
+ 更好的运行时性能
+ 更好的 IDE 类型推断性能

默认情况下，SFC并不会默认开启`script setup`, 如果需要开启需要将setup属性添加到script标签上

script setup中的内容 最终依旧会被编译为 setup函数

组件在多次调用的时候，就会创建多个实例，而每创建一个新的实例，就会重新执行一遍setup函数以初始化独立的数据

所以script setup与普通的 `<script>`只在组件被首次引入的时候执行一次不同



`父组件`

```vue
<template>
	<div>
		<Cpn />
	</div>
</template>

<script setup>
// 当使用 <script setup> 的时候，任何在 <script setup> 声明的顶层的绑定 (包括变量，函数声明，以及 import 引入的内容) 都能在模板中直接使用
// 所以在<script setup>中，导入的组件可以直接被使用
import Cpn from './components/Cpn'
</script>
```



`子组件`

```vue
<template>
	<div>
    <!--
			defineProps中定义的props在mustache中可以直接使用
			不需要通过props.name来获取
		-->
		{{ name }}
	</div>
</template>

<script setup>
// defineProps 和 defineEmits 以及 defineExpose 都是script setup∂提供的编译时宏，可以不引入直接使用

// 使用defineProps来定义接收的props的类型和默认值
// 该编译时宏会返回一个只读的props对象，可以供我们使用父组件传递过来的props
const props = defineProps({
	name: {
		type: String,
		default: 'Klaus'
	}
})

// defineEmits可以替代原本的emits选项
// 该编译时宏会返回emits函数，通过该函数可以向父组件传递自定义数据
const emits = defineEmits(['cpnClick'])

const cpnMsg = 'hello cpn'

// 使用 <script setup> 的组件是默认关闭的
// 通过模板 ref 或者 $parent 获取到的组件的实例，不会暴露任何在 <script setup> 中声明的属性和方法
// 通过 defineExpose 编译器宏来显式指定在 <script setup> 组件中要暴露出去的 属性和方法
defineExpose({
	cpnMsg
})
</script>
```