目前在React中，还流行 CSS-IN-JS 的模式：也就是把CSS像JS一样进行编写

其中比较常用的插件就是 [`styled-components`](https://styled-components.com/docs/basics#getting-started)

想要有语法提示，可以安装vscode插件：[`vscode-styled-components`](https://marketplace.visualstudio.com/items?itemName=styled-components.vscode-styled-components)



## 基本使用

`style.js`

```js
// style-components 设置的样式文件是JS文件
import styled from 'styled-components'

// 定义样式变量
const titleFontColor = 'skyblue'

// 通过模板字符串函数调用 返回一个样式组件
export const Title = styled.div`
  /* 写法和原生CSS完全一致 */
  color: ${titleFontColor}; /* 使用变量 */
  font-size: 20px;

  span {
    text-decoration: line-through;
  }
`
```

`App.jsx`

```jsx
// 导入样式组件
import { Title } from './style'

export default function App() {
  return (
    <>
      {/*
        Title是样式组件，本质就是加了样式的div元素, 所以使用方式和div元素一致
      */}
      <Title>Hello <span>World</span></Title>
    </>
  )
}
```



## props

`style.js`

```js
import styled from 'styled-components'

export const Title = styled.div`
	/* 
    使用通过样式组件传入的属性值
    如果没有设置对应props，则函数返回值为undefined --- 对应样式属性会被自动忽略 
  */
  color: ${props => props.color};
  font-size: 20px;
`
```

`App.jsx`

```jsx
import { Title } from './style'

export default function App() {
  return (
    <>
      <Title color='lightgray'>Hello World</Title>
    </>
  )
}
```



## 默认值

`style.js`

```jsx
import styled from 'styled-components'

// 通过attrs方法对props进行拦截
// attrs返回的方法会和传入的props执行Object.assign方法
// 因此可以使用attrs来设置样式props的默认值
export const Title = styled.div.attrs(props => ({
  color: props.color ?? 'sandybrown'
}))`
  color: ${props => props.color};
  /* 样式属性值不会自动添加单位，需要手动添加 */
  font-size: ${props => props.size}px;
`
```

`App.jsx`

```jsx
import { Title } from './style'

export default function App() {
  return (
    <>
      <Title size={20}>Hello World</Title>
    </>
  )
}
```



`编译结果`

```html
<div size="20" color="sandybrown" class="sc-aYaIB btvRng">Hello World</div>
```

1. 传递给组件的样式props会作为html元素的自定义属性被显示传递
2. 和css modules一样，会为元素添加独一无二的样式名
3. 一般来说，`style-components`注入的样式是`sc-[hash] [hash]`