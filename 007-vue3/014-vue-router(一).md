前端路由本质上维护了一个映射表，在该表中记录着不同的url地址对应页面所需要渲染的组件

在这个渲染流程中，我们只需要在第一次渲染的时候，向服务器请求静态资源的时候，将index.html下载下来

之后只要通过ajax去修改界面中组件即可，所以这种应用也被称之为SPA应用



## 路由模式

`前端路由的核心是 当URL发生改变的时候，不进行页面的整体刷新，而是只改变页面的部分内容`

目前常见的模式有两种: `hash模式` 和 `history模式`



#### hash模式

hash模式是通过监听`hashChange`事件来实现的

URL的hash也就是锚点(#), 本质上是改变window.location的href属性

我们可以通过直接赋值location.hash来改变href, 但是页面不发生刷新

hash的优势就是兼容性更好，在老版IE中都可以运行，但是缺陷是有一个#，显得不像一个真实的路径

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>URL的hash</title>
</head>
<body>
  <div id="app">
    <!-- 路由 -->
    <a href="#/home">Home</a>
    <a href="#/about">About</a>

    <!-- 占位符 -->
    <div id="content">Home</div>
  </div>

  <script>
    const contentElem = document.getElementById('content')

    // 监听路由改变函数 --- 路由改变的时候，页面是不会刷新的
    window.addEventListener('hashchange', () => {
      // location.hash --- 可以获取到hash值，获取到的值是带#的
      switch(location.hash) {
        case '#/home':
          contentElem.innerHTML = 'Home'
          break

        case '#/about':
          contentElem.innerHTML = 'About'
          break
      }
    })
  </script>
</body>
</html>
```



### history模式

history模式是调用history接口来实现的

使用history模式的页面，`必须使用服务器打开，不支持`file`协议开头的URL`

| 方法             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| replaceState方法 | 替换原来的路径，不会有新的浏览记录产生                       |
| pushState方法    | 使用新的路径，相当于新增了一条新的浏览记录                   |
| popstate事件     | 当除因history.pushState()或history.replaceState()方法触发的活动历史记录条目更改时，将触发popstate事件 |
| go方法           | 向前或向后改变路径, 参数为数字，可以跳转到任何的页面<br />如果参数是0，表示的是刷新当前页面<br />如果前进和回退的数值大于现有的历史记录的数值时，不会进行任何的操作，也不报错 |
| forward方法      | 向前改变路径，相当于go(1)                                    |
| back方法         | 向后改变路径，相当于go(-1)                                   |

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
</head>
<body>
  <div id="app">
    <!-- 路由 -->
    <a href="/home">Home</a>
    <a href="/about">About</a>

    <!-- 占位符 -->
    <div id="content">Home</div>
  </div>

  <script>
    const contentElem = document.getElementById('content')

    const aEls = document.getElementsByTagName('a')

    const changeRouter = () => {
      switch(location.pathname) {
        case '/home':
          contentElem.innerHTML = 'Home'
          break

        case '/about':
          contentElem.innerHTML = 'About'
          break
      }
    }

    for (const aEl of aEls) {
      aEl.addEventListener('click', e => {
        // 阻止默认行为
        e.preventDefault()

        // 设置历史记录，并修改url --- 不会发生跳转
        /*
        	有三个参数:
        	1. 一个对象，可以用于存储一些需要在多个页面中传递的数据，
        		 可以使用history.state来获取，初始值是null,
        		 在使用pushState或replaceState后会变为所传入的值

        	2. 新页面的title，目前浏览器都不支持，保留属性
        	3. 新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址
                   (对应的拼接规则和URL的拼接规则一致)
        */
        history.pushState({}, '', aEl.getAttribute('href'))
        changeRouter()
      })
    }

    // 点击回退键和前进键的时候, 也需要重新渲染对应的内容
    /*
      1. 当活动历史记录条目更改时，将触发popstate事件
      2. 事件名都是小写的
      3. 这个事件是在window上的
    */
    window.addEventListener('popstate', changeRouter)
  </script>
</body>
</html>
```



## Vue-router

Vue Router 是 Vue.js 的官方路由，与 Vue.js 核心深度集成，让用 Vue.js 构建单页应用(SPA)变得非常容易

vue-router是基于路由和组件的，路由用于设定访问路径, 将路径和组件映射起来

在vue-router的单页面应用中, 页面的路径的改变就是组件的切换

```shell
npm install vue-router
```



### 基本使用

`/routes/index.js`

```js
import { createRouter, createWebHistory } from 'vue-router'
import About from '../views/About'
import Home from '../views/Home'

const router = createRouter({
	// createWebHistory --- 创建history路由
	// createWebHashHistory --- 创建hash路由
	history: createWebHistory(),
	// 路由映射表
  // 当url发生改变的时候，vue-router会自动在路由表中查找对应组件
  // 并显示在合适的位置
	routes: [
    {
      path: '/', // 默认路径 重定向到 /home
      redirect: '/home'
    },
		{
			path: '/home',
			component: Home
		},
		{
			path: '/about',
			component: About
		}
	]
})

export default router
```



`main.js`

```js
import { createApp } from 'vue'
import App from './App.vue'

import router from './routes'

const app = createApp(App)

// vue-router本质上是一个vue的官方插件
// 所以在注册路由的时候使用use方法
app.use(router)

app.mount('#app')
```



`App.vue`

```vue
<template>
	<div>
		<!-- router-link 是 vue-router内置的用于进行组件切换的内置组件 -->
    <router-link to="/home">home</router-link>
    <router-link to="/about">about</router-link>

		<!--
			router-view是vue-router内置组件
			因为组件是局部内容更新，所以需要占位符告诉vue-router对应组件需要被渲染到哪里
		-->
		<router-view />
	</div>
</template>
```



## Router-link

| 属性               | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| to                 | 是一个字符串，或者是一个对象                                 |
| replace            | 设置 replace 属性的话，当点击时，会调用 router.replace()，而不是 router.push() |
| active-class       | 设置激活a元素后应用的class，默认是router-link-active         |
| exact-active-class | 链接精准激活时，应用于渲染的 `<a>` 的 class，默认是router-link-exact-active |



## 路由懒加载

当项目打包的时候，默认情况下，所有的逻辑代码都会被打包到app.js中

随着项目越来越大，app.js也会越来越大，会影响项目的首屏渲染速度

所以我们引入路由组件的时候，一般都是使用import函数进行引入，这样在打包的时候会进行分包处理

这样只有我们真正需要使用那些组件的时候，浏览器才会去加载那些组件

```js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
	history: createWebHistory(),
	routes: [
		{
			path: '/home',
			// 启用路由懒加载
			// webpackChunkName 是魔法注释 这类注释在打包的时候，会被webpack所读取，所以这是有意义的注释
			// webpackChunkName 用于指定 打包后的chunk名，默认为[hash].[hash].js
			// 指定后就会变成 [name].[hash].js
			component: () => import(/* webpackChunkName: home */'../views/Home')
		},
		{
			name: 'about', // 路由记录独一无二的名称
			path: '/about',
			component: () => import('../views/About'),
			// meta属性用于设置自定义数据
			meta: {
				name: 'Klaus',
				age: 23
			}
		}
	]
})

export default router
```



## 动态路由

有些路由会携带动态参数，例如`/user/1111, /user/2222`等，但是他们都需要展示user组件

那么就需要使用动态路由，所谓动态路由，就是在路径中使用一个动态字段来接收对应的参数

而这个动态字段就被称之为路径参数

```js
{
  path: '/user/:id', // 可以匹配 /user/1222 /user/aaa 但是无法匹配 /user
  component: () => import('../views/User.vue')
}
```

```vue
<template>
	<div>
		<!--
			 在template中可以使用$route来获取当前路由对象
			 注意: 是$route 不是 $router $router是当前路由表对象
		-->
		id: {{ $route.params.id }}
	</div>
</template>

<script setup>
	// 在options api中可以通过 this.$route 来获取当前的路由对象

	// 在composition api中 获取对应的路由对象，需要使用useRoute这个hook函数
	import { useRoute, onBeforeRouteUpdate } from 'vue-router'

	// useRoute hook 必须在setup语法糖顶层被调用和执行
	const route = useRoute()

	console.log(route.params.id)

  // 当动态参数改变，且当前组件被复用的时候 会回调的路由守卫函数
	// 参数to 表示 前往的那个路由的路由对象
	// 参数from 表示 当前的那个路由的路由对象
	onBeforeRouteUpdate((to, from) => {
		console.log(route.params.id)
	})
</script>
```



### not found

对于哪些没有匹配到的路由，我们通常会匹配到固定的某个页面， 比如not found页面

这个时候我们可编写一个动态路由用于匹配所有的页面

```js
import { createRouter, createWebHistory } from 'vue-router'

const router = createRouter({
	history: createWebHistory(),
	routes: [
		{
			path: '/home',
			component: () => import('../views/Home')
		},
		{
			path: '/about',
			component: () => import('../views/About'),
		},
		{
			path: '/user/:id',
			component: () => import('../views/User.vue')
		},
		// vue-router在匹配路由表的时候，会从上到下依次进行匹配
		// 如果匹配到了对应的路由，vue-router就会加载对应组件并停止匹配
		// 所以not found页面一般会放置到整个路由表的最后边
		{
			// 在路由最后的小括号中 是对应动态参数运行匹配的正则表达式
			// 动态参数 pathMatch 是自定义的变量名
			// 在正则匹配后边加上* 表示的时候 按照 / 进行分割
			// 例如 /aaa/bbb/ccc
			// 		 -> /:pathMatch(.*) => /aaa/bbb/ccc
			// 		 -> /:pathMatch(.*)* => ['aaa', 'bbb', 'ccc']
			path: '/:pathMatch(.*)*',
			component: () => import('../views/NotFound.vue')
		}
	]
})

export default router
```

![image.png](https://www.z4a.net/images/2022/08/25/image.png)


