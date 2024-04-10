React开发所需依赖有三个 「 单一职责原则 」

+ react：提供react基础语法API，主要功能将React.createElement -> VDOM

+ react-dom：
  + 将VDOM -> 真实DOM
    + react-dom/client： VDOM -> 真实DOM 「 CSR 」
    + react-dom/server： VDOM -> 字符串 「 SSR 」
  + react-native: VDOM -> 移动端控件

+ Babel：
  + JSX -> React.createElement
  + JSX是React.createElement的语法糖写法，所以如果直接使用React.createElement编写界面，可以不使用babel



## babel 

一个基于AST的代码转换工具

主要功能

1. ESNext -> ES5
2. TypeScript -> JavaScript
3. JSX -> React.createElement

```html
<!-- text/babel表示script标签中的内容需要先被babel解析后再交给浏览器执行  -->
<script type="text/babel">
</script>
```

