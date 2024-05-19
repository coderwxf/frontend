JSS 全称叫 JavaScript Style Sheets

JSS 是一种编写CSS的规范，本质也是 CSS IN JS技术的一种，其并不属于react技术栈范围

react中用于实现JSS的库为 [react-jss](https://cssinjs.org/react-jss/?v=v10.10.1)

早期react-jss支持在类组件中使用， 但自V10开始，react-jss已经不再支持在类组件中使用，只能用于函数组件中



## 基本使用

`style.js`

```js
import { createUseStyles } from 'react-jss';

// createUseStyles方法 接收样式对象 并生成一个hook函数
export const useStyles = createUseStyles({
  title: {
    // 样式名支持 小驼峰和中划线 两种写法 --- 推荐使用小驼峰写法
    fontSize: '20px', // 属性值需要时字符串
    color: 'skyblue',

    // 设置h2的子元素对应样式
    // & 和 样式名 之间要使用空格隔开
    // 因为title会被编译为独一无二的类名，所以子元素的类名可以写死
    '& .decoration': {
      'text-decoration': 'line-through'
    }
  }
})
```



`App.jsx`

```jsx
import { useStyles } from './style'

export default function App() {
  // useStyles是一个hook函数，调用后返回样式对象
  const { title } = useStyles()

  return (
    <>
      {/* className='decoration' 类名是写死的，所以并不会被编译 */}
      <h2 className={title}>Hello <span className='decoration'>World</span></h2>
    </>
  )
}
```



`编译结果`

```html
<h2 class="title-0-2-1">Hello <span class="decoration">World</span></h2>
```



## props和默认值

`app.jsx`

```jsx
import { useStyles } from './style'

export default function App() {
  // 通过hook函数传递对应props
  const { title } = useStyles({
    color: 'skyblue'
  })

  return (
    <>
      <h2 className={title}>Hello World</h2>
    </>
  )
}
```



`style.js`

使用props方式一

```jsx
import { createUseStyles } from 'react-jss';

export const useStyles = createUseStyles({
  title: {
    fontSize: '20px',
    // 设置默认值
    // 如果useStyles没有传递参数，props对应的值是undefined
    // 所以设置默认值的时候需要使用可选链 props?.color
    color: props => props?.color ?? 'lightgrey',
  }
})
```

`编译结果`

```html
<!-- props返回的样式会在 title样式后被插入 -->
<h2 class="title-0-2-1 title-d0-0-2-2">Hello World</h2>
```

```css
/* title-d0-0-2-2 中 props函数返回的样式 */
.title-d0-0-2-2 {
  color: skyblue;
}

/* title-0-2-1 是 title */
.title-0-2-1 {
  font-size: 20px;
}
```



使用props方式二

```jsx
import { createUseStyles } from 'react-jss';

export const useStyles = createUseStyles({
  title: props => ({
    fontSize: '20px',
    color: props?.color ?? 'lightgrey',
  })
})
```

`编译结果`

```html
<h2 class="title-0-2-1 title-d0-0-2-2">Hello World</h2>
```

```css
/* title-d0-0-2-2 是 props函数返回的结果  */
.title-d0-0-2-2 {
  color: skyblue;
  font-size: 20px;
}

/* title-0-2-1 是 title */
.title-0-2-1 {
}
```



## 支持类组件

从 react-jss 第10版本之后，不支持在类组件中使用，只能用于函数组件中

如果要在类组件中使用 v10版本的 react-jss，则需要使用HOC进行中转

`style.js`

```js
import { createUseStyles } from 'react-jss';

export const useStyles = createUseStyles({
  title: {
    fontSize: '20px',
    color: props => props?.color ?? 'skyblue',
  }
})
```



`App.jsx`

```jsx
import { PureComponent } from 'react'
import { useStyles } from './style'

function withStyles(Cpn) {
  return function(props) {
    const styles = useStyles(props)
    return <Cpn {...styles} />
  }
}

class App extends PureComponent {
  render() {
    return <h2 className={this.props.title}>Hello Wolrd</h2>
  }
}

export default withStyles(App)
```

