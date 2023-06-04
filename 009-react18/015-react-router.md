React Router在最近两年版本更新的较快，并且在最新的React Router6.x版本中发生了较大的变化

本例中，使用React Router使用的是6.x的版本

```shell
# 安装
# 注意: 这里安装和使用的是react-router-dom 不是 react-router
# react-router 只提供了路由的核心API，但是并不提供具体平台的操作方法
# 这么做是因为可以基于不同的平台开发对应的路由扩展
# 例如在浏览器端使用 react-router-dom
# 在react-native端使用 react-router-native
npm install react-router-dom
```



react-router最主要的API是给我们提供的一些组件

| 组件          | 说明                                                        |
| ------------- | ----------------------------------------------------------- |
| BrowserRouter | 使用history模式<br />例如: 'http://www.example.com/profile' |
| HashRouter    | 使用hash模式<br />例如: 'http://www.example.com/#/profile'  |

```jsx
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import { BrowserRouter } from 'react-router-dom'


const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
		{/* APP及其子组件使用history模式 */}
		<BrowserRouter>
	    <App />
		</BrowserRouter>
  </React.StrictMode>
);
```



`路由映射配置`

```jsx
import React, { PureComponent } from 'react'
import { Routes, Route, Link } from 'react-router-dom'
import Home from './pages/Home'
import Profile from './pages/Profile'

export class App extends PureComponent {
	render() {
		return (
			<div>
				<header>
					<h1>App</h1>
					{/* 定义路由跳转组件 */}
          {/*
          	通常路径的跳转是使用Link组件，最终会被渲染成a元素
          	to属性: Link中最重要的属性，用于设置跳转到的路径
          	replace: 是否使用replaceState方式来添加历史记录栈
          */}
					<Link to="/home">Home</Link>
					<Link to="/profile">Profile</Link>
				</header>

				<section>
					{/* 定义路由表 */}
					<Routes>
            {/*
            	path属性: 用于设置匹配到的路径
            	element属性: 设置匹配到路径后，渲染的组件实例
            						   注意是组件实例，不是组件类
            */}
						<Route path="/" element={<Home />} />
						<Route path="/home" element={<Home />} />
						<Route path="/profile" element={<Profile />} />
					</Routes>
				</section>
			</div>
		)
	}
}

export default App
```



## NavLink

 NavLink是在Link基础之上增加了一些样式属性

默认情况下，匹配中的NavLink组件。会给渲染出来的a元素添加上一个名为active的class属性

| 属性名    | 说明                                         |
| --------- | -------------------------------------------- |
| style     | 传入函数，函数接受一个对象，包含isActive属性 |
| className | 传入函数，函数接受一个对象，包含isActive属性 |

```jsx
export class App extends PureComponent {
	render() {
		return (
			<div>
				<header>
					<h1>App</h1>
					{/*
						如果此时路由是 http://www.example.com/home
						那么<NavLink to="/home">Home</NavLink>实际会被渲染为
						<a class="active">Home</a>
						我们可以通过active属性来设置对应的样式
					*/}
					<NavLink to="/home">Home</NavLink>
					<NavLink to="/profile">Profile</NavLink>
				</header>

				<section>
					<Routes>
						<Route path="/" element={<Home />} />
						<Route path="/home" element={<Home />} />
						<Route path="/profile" element={<Profile />} />
					</Routes>
				</section>
			</div>
		)
	}
}
```

```jsx
export class App extends PureComponent {
	render() {
		return (
			<div>
				<header>
					<h1>App</h1>
					{/*
						style 需要传入一个函数
						该函数参数是一个配置对象，其中isActive属性决定当前路由是否被匹配上
						返回值为需要添加到当前组件上的样式对象
					*/}
					<NavLink to="/home" style={({ isActive }) => ({color: isActive ? 'red'  : ''})}>Home</NavLink>
					<NavLink to="/profile" style={({ isActive }) => ({color: isActive ? 'red'  : ''})}>Profile</NavLink>
				</header>

				<section>
					<Routes>
						<Route path="/" element={<Home />} />
						<Route path="/home" element={<Home />} />
						<Route path="/profile" element={<Profile />} />
					</Routes>
				</section>
			</div>
		)
	}
}
```

```jsx
export class App extends PureComponent {
	render() {
		return (
			<div>
				<header>
					<h1>App</h1>
					{/*
						有的时候，我们并不希望被选中的组件默认添加的样式是active
						此时我们可以使用className来进行自定义操作
						className的值也需要是一个函数，函数参数为配置对象，返回值是所需要设置的样式值
					*/}
					<NavLink to="/home" className={ ({ isActive }) => isActive ? 'nav-active'  : '' }>Home</NavLink>
					<NavLink to="/profile" className={ ({ isActive }) => isActive ? 'nav-active'  : '' }>Profile</NavLink>
				</header>

				<section>
					<Routes>
						<Route path="/" element={<Home />} />
						<Route path="/home" element={<Home />} />
						<Route path="/profile" element={<Profile />} />
					</Routes>
				</section>
			</div>
		)
	}
}
```

但无论是NavLink还是Link最终都只能被渲染为a元素，实际开发中，很多时候，我们并不希望对应的元素是a元素，而可能是其它元素

比如button元素等，所以NavLink和Link在实际开发中并不常用



## Navigate

Navigate用于路由的重定向，当这个组件出现时，就会执行跳转到对应的to路径中

```jsx
<Routes>
  {/*
		当路径是/或/home的时候，显示Home组件
		但是这样本质上会创建两次Home组件
	*/}
  <Route path="/" element={<Home />} />
  <Route path="/home" element={<Home />} />
  <Route path="/profile" element={<Profile />} />
</Routes>
```

```jsx
<Routes>
  {/*
		一旦渲染Navigate组件，Navigate组件就会将路由修改为to属性所设置的对应值
		而当对应路由值发生改变的时候，页面就会渲染对应的匹配组件，从而实现了重定向的功能
	*/}
  <Route path="/" element={<Navigate to="/home" />} />
  <Route path="/home" element={<Home />} />
  <Route path="/profile" element={<Profile />} />
</Routes>
```



## NotFound

```jsx
<Routes>
  <Route path="/" element={<Navigate to="/home" />} />
  <Route path="/home" element={<Home />} />
  <Route path="/profile" element={<Profile />} />
  <Route path="/Login" element={<Login />} />
  {/*
		 *是一个通配符，所以没有被之前路由所捕获的路由都会被该路由捕获
		 所以*通配符一般被用于设置404页面
	*/}
  <Route path="*" element={<NotFound />} />
</Routes>
```



## 路由嵌套

```jsx
<Routes>
  <Route path="/" element={<Navigate to="/home" />} />
  <Route path="/home" element={<Home />}>
    {/*
			 Route组件中可以配置子Route 也就是所谓的子路由
			 <Route path="/home" element={<Navigate to="/home/recommend" />} />
			 表示的是如果路由是/home的时候，直接跳转到/home/recommend
			 因为如果不配置 当路由为/home的时候，二级路由并不知道需要被渲染为那个组件
		*/}
    <Route path="/home" element={<Navigate to="/home/recommend" />} />
    <Route path="/home/recommend" element={<Recommend />} />
    <Route path="/home/list" element={<List />} />
  </Route>
  <Route path="/profile" element={<Profile />} />
  <Route path="/Login" element={<Login />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```

```jsx
import React, { PureComponent } from 'react'
import { Link, Outlet } from 'react-router-dom'

export class Home extends PureComponent {
	render() {
		return (
			<div>
				<Link to="/home/recommend">recommend</Link>
				<Link to="/home/list">list</Link>

				{/* <Outlet>组件用于在父路由元素中作为子路由的占位元素 */}
				<Outlet />
			</div>
		)
	}
}

export default Home
```



## 手动实现路由跳转

`withRouter.jsx`

```jsx
import { useNavigate } from 'react-router-dom'

export default function withRouter(WrapperedCpn) {
	return function(props) {
		// useNavigate是 react-router-dom提供的编程式路由导航API
		// 而HOC只能在函数式组件中使用，且只能在函数式组件的首层作用域中使用
		// 所以如果类组件希望使用react-router6.x的时候，需要自己使用高阶组件进行封装
		const navigate = useNavigate()

		// 因为navigate是路由的一部分，未来可能会扩展别的配置
		// 所以注入的是router对象，而不是直接注入navigate函数
		return <WrapperedCpn { ...props } router = {{ navigate }} />
	}
}
```

```jsx
import React, { PureComponent } from 'react'
import { Outlet } from 'react-router-dom'
import withRouter from '../hoc/with-router'

export class Home extends PureComponent {
	render() {
		return (
			<div>
				<button onClick={ () => this.props.router.navigate('/home/recommend') }>recommend</button>
				<button onClick={ () => this.props.router.navigate('/home/list') }>list</button>

				<Outlet />
			</div>
		)
	}
}

export default withRouter(Home)
```



## 参数传递

```jsx
import { useLocation, useNavigate, useParams, useSearchParams } from 'react-router-dom'

export default function withRouter(WrapperedCpn) {
	return function(props) {
		// 编程式导航
		const navigate = useNavigate()

		// 当前路由对象信息
		const location = useLocation()

		// 当前动态路由信息
		const params = useParams()

		// 获取当前的查询字符串对象
		// useSearchParams是一个标准的hook函数，其返回值是一个数组
		// 数组的第一个元素 是我们所需要的URLSearchParams实例对象
		// 数组的第二个元素 是用于设置第一个元素的对应值
		const [searchParams] = useSearchParams()

		// searchParams是一个Entry对象，而Entry对象都是可迭代对象
		// 可以使用如下方式 将Entry对象转换为原生对象
		const query = Object.fromEntries(searchParams)

		return <WrapperedCpn { ...props } router = {{ navigate, location, params, query }} />
	}
}
```

```jsx
<Routes>
  {/* 在React-Routerv6.x版本中目前是不支持在路由中编写对应的正则匹配的 */}
  <Route path="/user/:id" element={ <User /> } />
  <Route path="*" element={<NotFound />} />
</Routes>
```

```jsx
import React, { PureComponent } from 'react'
import withRouter from '../hoc/with-router'

export class User extends PureComponent {
	render() {
		return (
			<div>
				<div>id: { this.props.router.params.id }</div>
				<div>name: { this.props.router.query.name }</div>
				<div>age: { this.props.router.query.age }</div>
			</div>
		)
	}
}

export default withRouter(User)
```



## 配置文件

目前我们所有的路由定义都是直接使用Route组件，并且添加属性来完成的

但是这样的方式会让路由变得非常混乱，我们希望将所有的路由配置放到一个地方进行集中管理

在早期的时候，Router并且没有提供相关的API，我们需要借助于react-router-config完成

在Router6.x中，为我们提供了useRoutes API可以完成相关的配置



假设存在如下路由配置

```jsx
<Routes>
  <Route path="/" element={<Navigate to="/home" />} />
  <Route path="/home" element={<Home />}>
    <Route path="/home" element={<Navigate to="/home/recommend"/>}/>
    <Route path="/home/recommend" element={<Recommend />} />
    <Route path="/home/list" element={<List />} />
  </Route>
  <Route path="/profile" element={<Profile />} />
  <Route path="*" element={<NotFound />} />
</Routes>
```



修改为配置后为

```js
import Home from '../pages/Home'
import Profile from '../pages/Profile'
import Recommend from '../pages/Recommend'
import List from '../pages/List'
import NotFound from '../pages/NotFound'
import { Navigate } from 'react-router-dom'

const routes = [
	{
		path:  '/',
		element: <Navigate to="/home" />
	},
	{
		path: '/home',
		element: <Home />,

		children: [
			{
				path: '/home',
				element: <Navigate to="/home/recommend" />
			},
			{
        // 这里的path也可以写成recommend
				path: '/home/recommend',
				element: <Recommend />
			},
			{
				path: '/home/list',
				element: <List />
			}
		]
	},
	{
		path: '/profile',
		element: <Profile />
	},
	{
		path: '*',
		element: <NotFound />
	}
]


export default routes
```

```jsx
import React from 'react'
import { Link, useNavigate, useRoutes } from 'react-router-dom'
import routes from './router'
import "./style.css"

export function App(props) {
  const navigate = useNavigate()

  function navigateTo(path) {
    navigate(path)
  }

  return (
    <div className='app'>
      <div className='header'>
        <span>header</span>
        <div className='nav'>
          <Link to="/home">首页</Link>
          <Link to="/about">关于</Link>
          <Link to="/login">登录</Link>
          <button onClick={e => navigateTo("/category")}>分类</button>
          <span onClick={e => navigateTo("/order")}>订单</span>

          <Link to="/user?name=why&age=18">用户</Link>
        </div>
        <hr />
      </div>
      <div className='content'>
        {/*
        	useRoutes 是一个hook函数，所以其只能在函数式组件中使用
        	useRoutes(routes)在被调用后，会返回一个Route组件
        	对应的路由结构和之前使用Route定义的路由结构是一致的
        */}
        {useRoutes(routes)}
      </div>
      <div className='footer'>
        <hr />
        Footer
      </div>
    </div>
  )
}

export default App
```



## 路由懒加载

上述的路由配置中所有的路由都是同步加载的，这就意味着所有的组件都会被打包到一个js文件中

必然随着项目变得庞大起来，js文件也会变得越来越大，所以我们在加载路由组件的时候，往往需要使用懒加载

```jsx
import { lazy } from 'react'
import { Navigate } from 'react-router-dom'

// React提供了lazy函数，来让我们实现异步加载对应组件
const Home = lazy(() => import('../pages/Home'))
const Profile = lazy(() => import('../pages/Profile'))
const Recommend = lazy(() => import('../pages/Recommend'))
const List = lazy(() => import('../pages/List'))
const NotFound = lazy(() => import('../pages/NotFound'))

const routes = [
	{
		path:  '/',
		element: <Navigate to="/home" />
	},
	{
		path: '/home',
		element: <Home />,

		children: [
			{
				path: '/home',
				element: <Navigate to="/home/recommend" />
			},
			{
				path: 'recommend',
				element: <Recommend />
			},
			{
				path: 'list',
				element: <List />
			}
		]
	},
	{
		path: '/profile',
		element: <Profile />
	},
	{
		path: '*',
		element: <NotFound />
	}
]


export default routes
```



```jsx
import React, { Suspense } from 'react';
import ReactDOM from 'react-dom/client';
import { BrowserRouter } from 'react-router-dom'
import App from './App';

const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
		<BrowserRouter>
		  {/*
				如果使用了路由懒加载
				那么就必须使用Suspense包裹App组件
				用于设定对应组件脚本还未被完全下载和渲染完成前，界面需要显示的内容
			*/}
	    <Suspense fallback={<h2>loading ...</h2>}>
				<App />
			</Suspense>
		</BrowserRouter>
  </React.StrictMode>
);
```