Vue Router 是 Vue.js 官方提供的一个路由管理库，它与 Vue.js 核心深度集成，让用 Vue.js 构建单页应用变得轻而易举

```shell
npm install vue-router@4
```

 

## 路由

路由（Route）是指将请求的URL映射到相应的处理程序或资源的过程，简单来说路由就是负责解析URL，然后决定应该展示哪个页面或者响应哪个API请求



### 后端路由

早期的网站开发中，服务器通常会直接生成渲染好对应的HTML页面，并将它返回给客户端进行展示, 这种操作被称为后端路由

为了处理多个页面，服务器会通过正则对URL进行匹配，并将请求交给一个Controller进行处理

Controller会进行各种处理，最终生成HTML或数据，并将它返回给前端展示



相比于前端路由，后端路由对SEO更加友好，因为渲染好的页面不需要单独加载任何js和css，可以直接交给浏览器展示

然而，后端路由的缺点也是显而易见的。HTML代码、数据以及对应的逻辑通常会混在一起，这样编写和维护都是非常糟糕的事情



### 前端路由

前端渲染是指在前端浏览器端进行页面渲染，而不是在后端服务器端进行页面渲染

随着Ajax的出现，前后端分离的开发模式逐渐流行起来。后端只提供API来返回数据，前端通过Ajax获取数据，并通过JavaScript将数据渲染到页面中

前端渲染的过程中，浏览器会请求静态资源（如HTML、CSS、JS等），然后通过JavaScript对这些资源进行渲染，生成最终的页面展示给用户。每次请求都会从静态资源服务器请求文件，后端只需要提供API来返回数据。

这样做的最大优点是前后端责任的清晰，后端专注于数据上，前端专注于交互和可视化上。并且当移动端出现后，后端不需要进行任何处理，依然使用之前的一套API即可

单页面富应用（SPA）的最主要特点是在前后端分离的基础上加了一层前端路由。前端路由的核心是改变URL，但是页面不进行整体的刷新

SPA通过前端路由来实现不同页面之间的切换，提供更好的用户体验。前端路由可以通过JavaScript来实现，也可以使用现成的框架如React Router、Vue Router等来简化开发。



## 初体验

如果使用Vue和Vue Router，我们可以很容易地通过组件来构建整个应用

用Vue Router后，我们只需要将组件映射到对应的路由上，让Vue Router知道在哪里渲染这些组件即可

这样，我们就可以通过路由来控制页面的展示和交互，提供更好的用户体验

```html
<!-- 使用 -->
<div id="app">
  <h1>Hello App!</h1>
  <p>
    <!--使用 router-link 组件进行导航, 通过传递 `to` 来指定链接 -->
    <!--`<router-link>` 将呈现一个带有正确 `href` 属性的 `<a>` 标签-->
    <router-link to="/">Go to Home</router-link>
    <router-link to="/about">Go to About</router-link>
  </p>
  
  <!-- 路由出口: 路由匹配到的组件将渲染在这里 -->
  <router-view></router-view>
</div>
```



```ts
// 定义

// 1. 定义路由组件. 也可以从其他文件导入
const Home = { template: '<div>Home</div>' }
const About = { template: '<div>About</div>' }

// 2. 定义路由: 每个路由都需要映射到一个组件
const routes = [
  { path: '/', component: Home },
  { path: '/about', component: About },
]

// 3. 创建路由实例
const router = VueRouter.createRouter({
  // 4. 定义路由实现方式
  history: VueRouter.createWebHashHistory(),
  routes
})

// 5. 创建并挂载根实例
const app = Vue.createApp({})
// 挂载路由实例
app.use(router)

app.mount('#app')
```

通过调用 `app.use(router)`，我们会触发第一次导航且可以在任意组件中以 `this.$router` 的形式访问它，并且以 `this.$route` 的形式访问当前路由，如果要在 `setup` 函数中访问路由，需要调用 `useRouter` 或 `useRoute` 函数