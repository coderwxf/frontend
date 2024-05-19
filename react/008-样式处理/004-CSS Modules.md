`style.module.css`

```css
.box {
  width: 300px;
  padding: 15px;
  border: 1px solid gray;
}

.box h1,
.box h2 {
  margin: 0;
  padding: 0;
  color: skyblue
}

.title {
  font-size: 20px;
}

.subTitle {
  font-size: 18px;
}

/*
  通过:global函数设置全局样式 --- 对应样式名不会转换
*/
:global(.global) {
  background: lightgrey;
}
```



`App.jsx`

```jsx
import React from 'react'
// 如果样式文件以module.css结尾，则该css样式文件开启了css modules
// 会默认导出一个样式对象
import style from './style.module.css'

export default function App() {
  // global是全局样式, 所以类名不会被转换，直接使用即可
  return <div className={`${style.box} global`}>
    <h1 className={style.title}>Hello React</h1>
    <h2 className={style.subTitle}>Hello World</h2>
  </div>
};
```



## 编译结果

```html
<div class="style_box__2f0Aw global">
  <h1 class="style_title__kuIgK">Hello React</h1>
  <h2 class="style_subTitle__oA8QP">Hello World</h2>
</div>
```

```css
/* 
	样式会被转换为 .模块名_类名__hash值 的形式
	1. 从而保证类名的唯一性
  2. 转换后的样式名遵循CSS BEM规范
*/
.style_box__2f0Aw {
  width: 300px;
  padding: 15px;
  border: 1px solid gray;
}

.style_box__2f0Aw h1,
.style_box__2f0Aw h2 {
  margin: 0;
  padding: 0;
  color: skyblue
}

.style_title__kuIgK {
  font-size: 20px;
}

.style_subTitle__oA8QP {
  font-size: 18px;
}

/* 全局样式，样式名不进行任何处理 */
.global {
  background: lightgrey;
}
```

所以通过css modules 设置的样式是有组件作用域的，并不是全局样式



## 样式组合

在 CSS Modules 中，一个选择器可以继承另一个选择器的规则，这称为”组合”

```js
/* 编译后为 style_color__7pWpK */
.color {
  color: skyblue
}

/* 编译后为 style_title__kuIgK */
.title {
  font-size: 20px;
  composes: color; 
}

/* 编译后为 style_subTitle__oA8QP */
.subTitle {
  composes: color;
  font-size: 18px;
}
```

```html
<!-- 样式会被组合在一起，composes入的样式永远是后插入的那个 -->
<h1 class="style_title__kuIgK style_color__7pWpK">Hello React</h1>
<h2 class="style_subTitle__oA8QP style_color__7pWpK">Hello World</h2>
```

