在开发中，我们想要给一个组件的显示和消失添加某种过渡动画，可以很好的增加用户体验

我们可以通过原生的CSS来实现这些过渡动画，但是React社区为我们提供了react-transition-group用来完成过渡动画

React曾为开发者提供过动画插件 react-addons-css-transition-group，后由社区维护，形成了现在的 react-transition- group

react-transition-group本身非常小，不会为我们应用程序增加过多的负担

这个库可以帮助我们方便的实现组件的 入场 和 离场 动画，使用时需要进行额外的安装

```shell
npm install react-transition-group
```



react-transition-group主要包含四个组件

| 组件             | 说明                                                         |
| ---------------- | ------------------------------------------------------------ |
| Transition       | 该组件是一个和平台无关的组件(不一定要结合CSS)<br />在前端开发中，我们一般是结合CSS来完成样式，所以比较常用的是CSSTransition |
| CSSTransition    | 在前端开发中，通常使用CSSTransition来完成过渡动画效果        |
| SwitchTransition | 两个组件显示和隐藏切换时，使用该组件                         |
| TransitionGroup  | 将多个动画组件包裹在其中，一般用于列表中元素的动画           |



## [CSSTransition](http://reactcommunity.org/react-transition-group/css-transition)

CSSTransition是基于Transition组件构建的

CSSTransition执行过程中，有三个状态:appear、enter、exit

它们有三种状态，需要定义对应的CSS样式

+ 开始状态:对于的类是-appear、-enter、-exit
+ 执行动画:对应的类是-appear-active、-enter-active、-exit-active
+ 执行结束:对应的类是-appear-done、-enter-done、-exit-done



| 属性          | 说明                                                         |
| ------------- | ------------------------------------------------------------ |
| in            | 触发进入或者退出状态位<br />当in为true时，触发进入状态，会添加-enter、-enter-acitve的class开始执行动画，<br />当动画执行结束后，会移除两个class， 并且添加-enter-done的class<br />当in为false时，触发退出状态，会添加-exit、-exit-active的class开始执行动画，<br />当动画执行结束后，会移除两个class，并 且添加-enter-done的class |
| classNames    | 决定了在编写css时，对应的class名称<br />当className的值为card时，对应样式类似于card-enter、card-enter-active、card-enter-done |
| timeout       | 过渡动画的时间<br />必传项<br />css中设置的是实际动画的执行时间<br />而timeout属性决定的是react-transition-group在什么时间点，<br />为我们添加对应的class样式名<br />在实际使用中推荐将css中动画的执行时间和timeout属性值设置成一致 |
| appear        | 是否在初次进入添加动画                                       |
| unmountOnExit | 默认情况下，对应组件并不会被卸载，只是自己通过css属性来隐藏<br />将unmountOnExit的值设置为true的时候，退出后将会卸载对应组件 |

```jsx
import React, { PureComponent } from 'react'
import { CSSTransition } from 'react-transition-group'
import './style.css'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			isShow: true
		}
	}

	render() {
		return (
			<div>
				<button onClick={() => this.changeState()}>click me</button>
				<CSSTransition
					in={ this.state.isShow }
					timeout={ 2000 }
					classNames='fade'
          appear
				>
					<div>Hello Wolrd</div>
				</CSSTransition>
			</div>
		)
	}

	changeState() {
		this.setState({
			isShow: !this.state.isShow
		})
	}
}

export default App
```



如果在使用上述示例的时候，开启了React.StrictMode

而默认情况下CSSTransition内部使用了findDOMNode方法

而findDOMNode方法在React严格模式下是不被允许的

因此会看控制台看到相应的错误信息

```jsx
import React, { PureComponent, createRef } from 'react'
import { CSSTransition } from 'react-transition-group'
import './style.css'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.transitionRef = createRef()

		this.state = {
			isShow: true
		}
	}

	render() {
		return (
			<div>
				<button onClick={() => this.changeState()}>click me</button>
        {/*
        	CSSTransition支持一个名为nodeRef的属性
        	nodeRef的值一般是执行动画的根元素的原生DOM节点
        	当没有设置nodeRef的时候，CSSTransition内部会使用findDOMNode来获取执行动画根节点所对应的原生DOM节点，所以在使用React严格模式的时候，会出现错误警告
        	当设置了nodeRef对应值的时候，CSSTransition组件就会使用我们所传入的节点作为执行动画的根节点，而不再主动调用findDOMNode方法来获取对应的根节点
        */}
				<CSSTransition
          appear
					timeout={ 2000 }
					classNames='fade'
					in={ this.state.isShow }
					nodeRef={ this.transitionRef }
				>
					<div ref={ this.transitionRef }>Hello Wolrd</div>
				</CSSTransition>
			</div>
		)
	}

	changeState() {
		this.setState({
			isShow: !this.state.isShow
		})
	}
}

export default App
```

```css
/* 入场动画 */

/* 入场进入前 和 页面初次渲染动画入场前 */
.fade-enter,
.fade-appear {
	opacity: 0;
}

/* 入场动画执行中 和 页面初次渲染动画执行中 */
.fade-enter-active,
.fade-appear-active {
	opacity: 1;
	transition: opacity 2s ease;
}

/* 入场动画执行完毕 和 页面初次渲染动画离场时 */
/* 入场动画执行完毕后的样式就是默认样式，所以省略 */
/* .fade-enter-done,
	 .fade-appear-done {
	opacity: 1;
} */

/* 出场动画 */

/* 出错动画执行前 */
/* 和默认样式一致，省略 */
/* .fade-exit {
	opacity: 1;
} */

/* 出场动画执行中 */
.fade-exit-active {
	opacity: 0;
	transition: opacity 2s ease;
}

/* 出场动画执行后 */
.fade-exit-done {
	opacity: 0;
}
```



CSSTransition对应的钩子函数:主要为了检测动画的执行过程，来完成一些JavaScript的操作

| 方法       | 作用                       |
| ---------- | -------------------------- |
| onEnter    | 在进入动画之前被触发       |
| onEntering | 在应用进入动画时被触发     |
| onEntered  | 在应用进入动画结束后被触发 |
| onExit     | 在离开动画之前被触发       |
| onExiting  | 在应用离开动画时被触发     |
| onExited   | 在应用离开动画结束后被触发 |



## SwitchTransition

SwitchTransition可以完成两个组件之间或者一个组件的两个状态直接的炫酷动画

这个动画在vue中被称之为 vue transition modes

react-transition-group中使用SwitchTransition来实现该动画

SwitchTransition中主要有一个属性:mode，有两个值

+ in-out:表示新组件先进入，旧组件再移除
+ out-in:表示旧组件先移除，新组件再进入

```jsx
import React, { PureComponent } from 'react'
import { CSSTransition, SwitchTransition } from 'react-transition-group'
import './style.css'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			isShow: true
		}
	}

	render() {
		const { isShow } = this.state

		return (
			<div>
				<button onClick={() => this.changeState()}>click me</button>
        {/* mode用于控制动画出入顺序，默认就是out-in */}
				<SwitchTransition mode="out-in">
					{/*
						SwitchTransition组件里面要有CSSTransition或者Transition组件，不能直接包裹你想要切换的组件
						CSSTransition如果被SwitchTransition组件所包裹，需要使用key属性来标识各个状态，以便于在各个状态之间进行切换
					*/}
					<CSSTransition
						key={ isShow ? 'show' : 'hidden' }
						timeout={ 1000 }
						classNames='fade'
						appear
					>
						<div>Hello Wolrd</div>
					</CSSTransition>
				</SwitchTransition>
			</div>
		)
	}

	changeState() {
		this.setState({
			isShow: !this.state.isShow
		})
	}
}

export default App
```

```css
.fade-enter,
.fade-appear {
	transform: translateX(100px);
	opacity: 0;
}

.fade-enter-active,
.fade-appear-active {
	transform: translateX(0);
	opacity: 1;
	transition: all 1s ease;
}

.fade-exit-active {
	opacity: 0;
	transform: translateX(-100px);
	transition: all 1s ease;
}

.fade-exit-done {
	opacity: 0;
}
```



## TransitionGroup

当我们有一组动画时，需要将这些CSSTransition放入到一个TransitionGroup中来完成动画

```jsx
import React, { PureComponent } from 'react'
import { TransitionGroup, CSSTransition } from 'react-transition-group'
import './style.css'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			friends: [
				{
					id: 0,
					name: 'Klaus'
				},
				{
					id: 1,
					name: 'Alex'
				},
				{
					id: 2,
					name: 'Alice'
				},
				{
					id: 3,
					name: 'Jhon'
				}
			]
		}
	}

	render() {
		const { friends } = this.state

		return (
			<div>
				{/*
					默认情况下，TransitionGroup会被渲染为div元素
					如果不希望渲染为div元素的时候，可以通过component属性来进行指定
			  */}
				<TransitionGroup component='ul'>
					{
						friends.map((friend, index) => (
							// CSSTransition的key必须唯一，且不能是index
							// 因为移除列表元素的时候，对应元素的index可能会发生改变
							// 从而导致离场动画执行混乱
							<CSSTransition
							timeout={ 1000 }
							classNames='fade'
							key={ friend.id }
						>
							<li>
								<span>{ friend.name }</span>
								<button onClick={() => this.removeFriend(index)}>remove</button>
							</li>
						</CSSTransition>
						))
					}
				</TransitionGroup>
				<button onClick={() => this.addFriend()}>add</button>

			</div>
		)
	}

	addFriend() {
		const friends = [...this.state.friends]

		friends.push({
			id: friends.length,
			name: 'Steven'
		})

		this.setState({
			friends
		})
	}

	removeFriend(index) {
		const friends = [...this.state.friends]
		friends.splice(index, 1)

		this.setState({
			friends
		})
	}
}

export default App
```

```css
.fade-enter {
	transform: translateX(100px);
	opacity: 0;
}

.fade-enter-active {
	transform: translateX(0);
	opacity: 1;
	transition: all 1s ease;
}

.fade-exit-active {
	opacity: 0;
	transform: translateX(-100px);
	transition: all 1s ease;
}

.fade-exit-done {
	opacity: 0;
}
```