目前，前端最流行的开发方式是组件化，而CSS的设计本身就不是为组件化而生的，所以在目前组件化的框架中都在需要一种合适的CSS解决方案

在组件化开发环境下的CSS，应该满足如下需求:

1. 可以编写局部css:  css具备自己的具备作用域，不会随意污染其他组件内的元素
2. 可以编写动态的css:  可以获取当前组件的一些状态，根据状态的变化生成不同的css样式
3. 支持所有的css特性:伪类、动画、媒体查询等
4. 编写起来简洁方便、最好符合一贯的css风格特点
5. 等等 。。。。



Vue在CSS上虽然不能称之为完美，但是已经足够简洁、自然、方便了，至少统一的样式风格不会出现多个开发人员、多个项目 采用不一样的样式风格

相比而言，React官方并没有给出在React中统一的样式风格

由此，从普通的css，到css modules，再到css in js，有几十种不同的解决方案，上百个不同的库

大家一致在寻找最好的或者说最适合自己的CSS方案，但是到目前为止也没有统一的方案



## 内联样式

内联样式是官方推荐的一种css样式的写法:

1. style 接受一个采用小驼峰命名属性的 JavaScript 对象，而不支持 CSS 字符串形式写法
2. 并且可以引用state中的状态来设置相关的样式

`优点`

1. 内联样式, 样式之间不会有冲突
2. 可以动态获取当前state中的状态

`缺点`

1. 写法上都需要使用驼峰标识

2. 某些样式没有提示
3. 大量的样式, 代码混乱
4. 某些样式无法编写(比如伪类/伪元素)

所以官方依然是希望内联合适和普通的css来结合编写

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {
  constructor(props) {
    super(props)

    this.state = {
      color: 'red'
    }
  }


  render() {
    return (
      <div style={{ colot: this.state.color, backgroundColor: 'skyblue' }}>App</div>
    )
  }
}

export default App
```



## 普通的CSS

普通的css我们通常会编写到一个单独的文件，之后再进行引入

这样的编写方式和普通的网页开发中编写方式是一致的

但是使用这种方式编写我们的css有一个致命缺点，即没有自己的样式作用域

默认引入的样式都是全局样式

```jsx
import React, { PureComponent } from 'react'
import './style.css'

export class App extends PureComponent {
  constructor(props) {
    super(props)

    this.state = {
      color: 'red'
    }
  }


  render() {
    return (
      <div>App</div>
    )
  }
}

export default App
```





## css modules

css modules并不是React特有的解决方案，而是所有使用了类似于webpack配置的环境下都可以使用的

如果在其他项目中使用它，那么我们需要自己来进行配置，比如配置webpack.config.js中的modules: true等

如果使用React脚手架，其内部已经内置了css modules的配置

.css/.less/.scss 等样式文件都需要修改成 .module.css/.module.less/.module.scss 等 之后就可以引用并且进行使用了

```jsx
import React, { PureComponent } from 'react'
// 引入css modules - 实际会将对应的css文件编译为一个JS对象
// 当我们将一个样式文件的后缀名修改为 .module.css的时候，对应样式文件就变成了css样式模块
import AppStyle from './style.module.css'

export class App extends PureComponent {
  constructor(props) {
    super(props)

    this.state = {
      color: 'red'
    }
  }


  render() {
    return (
      <div>
				{/*
					css modules 会将css文件编译为js对象，所以需要向属性那样使用
					同理 如果使用了一个样式模块中不存在的样式，对应值就是undefined
					实际表现为对应元素样式不生效, 页面并不会报错
				*/}
				<h2 className={AppStyle.title}>title</h2>
				{/*
					对应的样式会被编译w为 style_content__O3F7P
					也就是 [样式文件名]_[样式名]__[hash值]
					从而避免出现样式冲突
				*/}
				<p className={AppStyle.content}>content</p>
			</div>
    )
  }
}

export default App
```



但是CSS modules依旧存在自己的缺点

1. 引用的类名，不能直接使用连接符(.home-title)，需要使用中括号语法，因为连字符在JavaScript中是不识别的
2. 所有的className都必须使用{style.className} 的形式来编写
3. 不方便动态来修改某些样式，依然需要使用内联样式的方式



## css in js

“CSS-in-JS” 是指一种模式，其中 CSS 由 JavaScript 生成而不是在外部文件中定义

注意此功能并不是 React 的一部分，而是由第三方库提供



React的思想中认为逻辑本身和UI是无法分离的，所以才会有了JSX的语法

事实上CSS-in-JS的模式就是一种将样式(CSS)也写入到JavaScript中的方式，并且可以方便的使用JavaScript的状态

所以React有被人称之为 All in JS



CSS-in-JS通过JavaScript来为CSS赋予一些能力，包括类似于CSS预处理器一样的样式嵌套、函数定义、逻辑复用、动态修 改状态等等

所以，目前可以说CSS-in-JS是React编写CSS最为受欢迎的一种解决方案

目前最为常用的css-in-js库是 `styled-components`

```shell
npm install styled-components
```



`组件`

 ```jsx
 import React, { PureComponent } from 'react'
 // 使用css in js后，对应的样式直接编写在js文件中即可
 import AppWrapper from './style.js'

 export class App extends PureComponent {
   render() {
     return (
       <AppWrapper>
 				<h2 className="title">title</h2>
 				<p className="content">content</p>
 			</AppWrapper>
     )
   }
 }

 export default App

 ```

`样式`

```js
import styled from 'styled-components'

// styled.div本质上是一个函数  需要通过标签模板字符串进行调用 并返回一个新的有对应样式的组件
// ps: 默认情况下对应样式是不会高亮的，需要高亮可以安装vscode-styled-components插件
export default styled.div`
	/* 对应样式会被编译为 .jaGVDq */
	/* 即一个唯一的hash值 */
	/* 所以使用styled-components编写对应的样式会存在自己的样式作用域 */
	/* 组件和组件之间的样式是不会冲突的 */
	/*
		但因为编译后的样式类似于 .jaGVDq .content
		所以在实际使用过程中，对于后代选择器仍然可能出现样式冲突的情况
  */
	background-color: #f5f5f5;

	/* .jaGVDq .title */
	.title {
		color: red;

		&:hover {
			background-color: gray;
		}
	}

	/* .jaGVDq .content */
	.content {
		color: skyblue
	}
`
```



### 样式组件

```js
import styled from 'styled-components'

export default styled.div`
	background-color: #f5f5f5;

	.content {
		color: skyblue
	}
`

// 如果某一块的样式比较多，可以将对应的样式进行单独抽离
// 形成一个独立的样式组件
export const TitleWrapper = styled.h2`
	color: red;

	&:hover {
		background-color: gray;
	}
`
```



### 引入外部变量

```jsx
import React, { PureComponent } from 'react'
import AppWrapper from './style.js'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			color: 'skyblue'
		}
	}


  render() {
    return (
			// AppWrapper本质上是一个组件
			// 所以对应的样式直接以props的形式进行传入即可
      <AppWrapper color={ this.state.color }>
				<h2 className="title">title</h2>
				<p className="content">content</p>
			</AppWrapper>
    )
  }
}

export default App
```

```js
import styled from 'styled-components'

export default styled.div`
	.title {
		/*
			如果直接使用props.color 在js中 会沿着作用域链去查找对应的props
			所以直接使用props 在css in js中是不合适的
			所以在styled-components中引入外部变量的时候，需要传入一个回调函数
			该回调函数的参数为外部传入的props，返回值是所需要设置的对应样式值
		*/
		color: ${ props => props.color };

		&:hover {
			background-color: gray;
		}
	}

	.content {
		color: gray
	}
`
```



### 默认值

```jsx
import styled from 'styled-components'

// 因为styled-components 本质上就是css in js
// 所以我们也可以通过如下方式来使用styled-components
export default styled.div`
	.title {
		// 解构语法
		color: ${ ({ color }) => color };

		&:hover {
		  // 解构的时候 设置对应的默认值
			background-color: ${ ({ bgColor = 'yellow' }) => bgColor };
		}
	}

	.content {
	 	// 空值合并操作符
		color: ${ ({ contentColor }) => contentColor ?? 'orange' }
	}
`
```



有的时候，我们希望在外部没有传入对应props的时候，可以存在对应的默认值

但是使用上述写法，每使用一次就需要单独设置一次对应的默认值，这必然是十分麻烦的

所以styled-components提供了attrs方法，专门用于设置默认值

```jsx
import styled from 'styled-components'

// attrs方法会返回对应的含有样式的样式组件
// 所以在这里可以链式调用
// 在attrs中可以传入一个回调函数，用于设置对应的默认值
// 回调函数在被调用的时候会将对应的props传递过来
export default styled.div.attrs(props => ({
	contentColor: props.contentColor ?? 'purple'
}))`
	.title {
		color: ${ props => props.color };

		&:hover {
			background-color: gray;
		}
	}

	.content {
		color: ${ props => props.contentColor }
	}
`
```



### 引入全局样式

`/style/theme.js --- 全局的主题样式文件`

```js
export const primaryColor = '#409eff'
export const warnColor = '#e6a23c'
export const successColor = '#67c23a'
```

`样式组件`

```js
import styled from 'styled-components'
import {
	primaryColor,
	successColor
} from '../style/theme'

export default styled.div`
	.title {
		color: ${ successColor };

		&:hover {
			background-color: gray;
		}
	}

	.content {
		color: ${ primaryColor }
	}
`
```



### provider

`app.jsx`

```jsx
import ReactDOM from 'react-dom/client'
import App from './App'
import { StrictMode } from 'react'
import { ThemeProvider } from 'styled-components';

ReactDOM.createRoot(document.querySelector('#root')).render(
	<StrictMode>
		{/*
			ThemeProvider 是 styled-components中 导出的 context
			通过theme属性来设置对应的全局变量值
		*/}
		<ThemeProvider theme={{ color: 'skyblue' }}>
			<App />
		</ThemeProvider>
	</StrictMode>
)
```

```jsx
import styled from 'styled-components'

export default styled.div`
	.title {
		/* ThemeContext中提供的全局样式值会被传入到 props.theme中 */
		color: ${ props => props.theme.color };
	}

	.content {
		color: red
	}
`
```



### 样式继承

```jsx
const OriginButton = styled.button`
	border-radius: 5px;
`

export const PrimaryButton = styled(OriginButton)`
	color: #fff;
	background-color: #409eff;
`
```



## 动态添加class

`写法一 --- 使用三元运算符`

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			isActive: true,
			isCurrent: true
		}
	}

	render() {
		const { isActive, isCurrent } = this.state

		return (
			<div>
				<div className={ `${isActive ? 'active' : '' } ${ isCurrent ? 'current' : '' }` }>active</div>
			</div>
		)
	}
}

export default App
```



`写法二 --- 使用join方法`

```jsx
import React, { PureComponent } from 'react'

export class App extends PureComponent {
	constructor(props) {
		super(props)

		this.state = {
			isActive: true,
			isCurrent: true
		}
	}

	render() {
		const { isActive, isCurrent } = this.state

		const activeClass = []

		if (isActive) {
			activeClass.push('active')
		}

		if (isCurrent) {
			activeClass.push('current')
		}

		return (
			<div>
				<div className={ activeClass.join(' ') }>active</div>
			</div>
		)
	}
}

export default App
```



`写法三 --- 使用第三方库 classnames`

```shell
npm install classnames
```

```jsx
classNames('foo', 'bar'); // => 'foo bar'
classNames('foo', { bar: true }); // => 'foo bar'
classNames({ 'foo-bar': true }); // => 'foo-bar'
classNames({ 'foo-bar': false }); // => ''
classNames({ foo: true }, { bar: true }); // => 'foo bar'
classNames({ foo: true, bar: true }); // => 'foo bar'

// classnames 支持多种编写方式 混合使用
classNames('foo', { bar: true, duck: false }, 'baz', { quux: true }); // => 'foo bar baz quux'

// 只要是falsy值的结果 全部都会被忽略
classNames(null, false, 'bar', undefined, 0, 1, { baz: null }, ''); // => 'bar 1'

// classnames 支持 数组写法 和 计算属性名
className={ classnames([{ [activeClass]: isActive }, { current: isCurrent }])
```