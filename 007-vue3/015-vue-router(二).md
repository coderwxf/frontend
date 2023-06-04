## 路由嵌套

```js
routes: [
  {
    path: '/home',
    component: () => import('../views/Home'),
    // 用来定义子路由
    children: [
      {
        // 默认跳转路由 这里也可以被设置为 /home/recommend
        // 一般情况下，如果path属性的值是空字符串的时候
        // 推荐给当前路由对象设置一个name属性，以标识当前路由对象
        path: '',
        redirect: '/home/recommend'
      },
      {
        // 注意: 这里需要编写的是相对路径，vue-router会自动进行路径拼接
        // 即此处对应的路径是 /home/recommend
        // 因为对于所有的/home对应的子路由而言，其baseUrl都是/home
        // 所以在可以我们可以省略baseUrl
        path: 'recommend',
        component: () => import('../views/Home/Recommend.vue')
      },
      {
        path: 'ranking',
        component: () => import('../views/Home/Ranking.vue')
      }
    ]
  }
]
```



## 编程式导航

有时候我们希望通过代码来完成页面的跳转, 此时就可以使用编程式导航

```js
import { useRouter } from 'vue-router'

// 生成路由表对象，必须在script顶层生成
const router = useRouter()

// 路由跳转到/about对应的组件
// router.push('/about?name=klaus&age=23')

// 上述写法 等价于
router.push({
  path: '/about',
  query: {
    name: 'Klaus',
    age: 23
  }
})
```



| 方法           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| router.push    | 跳转到一个新页面<br />会向历史记录栈中压入一个新的页面       |
| router.replace | 跳转到一个新的页面<br />不会向历史记录栈中压入一个新的页面   |
| router.go(n)   | 前进或回退n个页面<br />n大于历史记录栈中可以前进或回退的个数时候，会前进到最新的页面或回退到第一个页面<br />`router.go(1)  === `<br />`router.go(-1) === router.back()` |
| router.forward | 前进一个页面，无法前进时，静默失效                           |
| router.back    | 后退一个页面， 无法后退时，静默失效                          |



## 动态管理路由

在某些情况下，我们需要根据用户不同的权限，来动态注册不同的路由

`添加`

```js
// 动态添加一级路由
router.addRoute({
	path: '/about',
	component: () => import('../views/About.vue')
})
```

```js
// 动态注册二级路由
// param1 - 父级路由对象名称(即路由的name属性值)
// param2 - 需要注册的子级路由
router.addRoute('home', {
	path: 'recommend',
	component: () => import('../views/Home/Recommend.vue')
})
```



`删除路由`

1. 方式一:  添加一个name相同的路由, 用新的组件替换原本的组件(不推荐)

   ```js
   router.addRoute ({ path: '/about', name: 'about', component: About})
   
   // 在路由中，路由的名称是唯一的
   // 所以当两个路由的名称一致的时候， 新的路由会覆盖旧的路由
   router. addRoute({ path: '/other', name: 'about', component: Home })
   ```

2. 方式二:  通过removeRoute方法，传入路由的名称

   ```js
   router.addRoute({ path: '/about', name: 'about', component: About })
   
   // 删除路由
   router.removeRoute('about')
   ```

3. 方式三: 通过addRoute方法的返回值回调

   ```js
   const removeRoute = router.addRoute(routeRecord)
   removeRoute() //册除路由

`其它方法`

+  router.hasRoute(): 检查路由是否存在

  ```js
  // 参数为路由名称
  console.log(router.hasRoute('home'))
  ```

  

+ router.getRoutes(): 获取一个包含所有路由记录的数组

  ```js
  // 获取到的结果是所有路由组成的别拍平的一维数组
  console.log(router.getRoutes())
  ```



## [路由守卫](https://router.vuejs.org/zh/guide/advanced/navigation-guards.html)

vue-router 提供的导航守卫主要用来通过跳转或取消的方式守卫导航

也就是说路由守卫本质上就是一些拦截函数，这些拦截函数，会在路由发生改变，组件重新渲染的各个阶段被回调

通过实现这样路由守卫函数，就可以在路由导航的各个阶段完成我们所需要的对应功能



一般路由守卫函数都有如下两个参数

| 参数 | 说明                    |
| ---- | ----------------------- |
| to   | 即将进入的路由Route对象 |
| from | 即将离开的路由Route对象 |



一般路由守卫函数返回值都可以是如下形式

| 方式                        | 说明                                                         |
| --------------------------- | ------------------------------------------------------------ |
| false                       | 取消导航                                                     |
| 不返回，也就是返回undefined | 进行默认导航                                                 |
| 返回一个路由地址            | 导航到所返回的路由地址<br /> 对应的路由地址即可以是string类型的路径，也可以使用一个route对象 |

```js
// 全局的前置守卫beforeEach是在导航之前被回调的函数
router.beforeEach(to => {
	const token = localStorage.getItem('token')

	if (to.path !== '/login' && !token) {
		return `/login?redirect=${encodeURIComponent(to.path)}`
	}
})
```

![image.png](https://s2.loli.net/2022/08/27/QNGjrc6CFXwHJsl.png) 



### 路由前置守卫

挂载在router对象上，匹配路由前会被触发的回调函数

在前置守卫里可以进行权限判断或者跳转操作

```js
// to - 需要前往的那个路由的路由对象
// from - 来自于那个路由对应的路由对象
router.beforeEach((to, from) => {
  console.log('路由前置守卫')
  console.log(to, from)
})
```



### 路由后置守卫

挂载在router对象上，匹配路由并渲染完对应组件时候会被触发的回调

在路由后置守卫中，可以执行埋点操作和日志分析

```js
router.afterEach((to, from) => {
  console.log('路由后置守卫')
  console.log(to, from)
})
```



### 路由解析守卫

```js
// 功能和beforeEach一致
// 唯一的区别在于 
//     beforeEach的时候, 实例还没有被创建
//     beforeResolve的时候, 实例已经被创建了
router.beforeResolve((to, from) => {
  console.log('beforeResolve')
})
```



### 路由独享守卫

功能和beforeEach类似，不过是针对于某一个单一的路由对象进行拦截的回调函数

```js
{
  path: '/',
  component: () => import('./views/Home.vue'),
  beforeEnter(to, from) {
    console.log(to)
    console.log(from)
    console.log('路由发生了拦截')
  }
}
```



### 组件内守卫

```js
beforeRouteEnter(to, from) {
  // 路由匹配到了，需要渲染对应组件之前被回调的守卫函数
  // 在调用该函数的过程中，因为没有创建对应的组件实例
  // 所以不能在该函数中使用this关键字
  // 在composition api中 并没有该函数对应的api
  // 所以在组合式API中，并不能使用该函数
}
```

```js
import { onBeforeRouteUpdate } from 'vue-router'

// 在当前路由改变，但是该组件被复用时调用
// 举例来说，对于一个带有动态参数的路径 `/users/:id`
// 在 `/users/1` 和 `/users/2` 之间跳转的时候
// 在options api中 该函数可以使用this 但是在组合api中，该函数无法使用对应的this
onBeforeRouteUpdate((to, from) => {
  console.log('Foo组件被复用了')
})
```

```js
// 离开该组件时候 会被回调的函数
// 在options api中 该函数可以使用this 但是在组合api中，该函数无法使用对应的this
onBeforeRouteLeave((to, from) => {
  console.log('离开了Foo组件')
})
```

