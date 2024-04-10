1. 在 Vite 中，当你导入一个 `.css` 文件时，该文件的内容将会被插入到页面的 `<style>` 标签中

2. Vite 默认支持使用 `@import` 来内联 CSS 文件，并且能识别路径别名
   + 内置实现依赖于`postcss-import`
   + 通过`postcss-import`可以在 CSS 文件中引入其它的 CSS 文件，并在 PostCSS 处理过程中将这些外部的样式合并到当前文件中

3. 支持使用 `Sass`、`Less`、`Stylus` 等预处理器
   + 只需安装对应的预处理器依赖，不需额外的 Vite 插件
   + 不推荐使用预处理器，vite推荐使用现代原生CSS特性
   + 可以通过 `<style lang="xxx">` 指定使用哪种预处理器
   + css module 和预处理器是可以一起使用的 ---  例如`xxx.module.scss`

4. 如果你的项目有一个有效的 PostCSS 配置(比如 `postcss.config.js`)，它将会自动应用到所有导入的 CSS 文件中

5. 在 Vite 中，`.module.css` 结尾的文件被视为 CSS 模块文件
   + 当你导入这些文件时，你会得到一个模块对象，你可以通过这个对象应用样式
   + 可以通过vite配置项`css.modules.localsConvention`来开启 camelCase 格式变量名转换
   + 内置实现依赖于[[postcss-modules](https://github.com/css-modules/postcss-modules)]



## 禁用 CSS 注入

禁用css注入的意思就是编译时不将样式插入到style标签中

```js
import './foo.css' // 样式将会注入页面

// 禁止注入后，样式文件默认导出的是 美化格式后生成的样式字符串
import otherStyles from './bar.css?inline' // 样式不会注入页面
```



## Lightning CSS 

`Lightning CSS`是`Vercel`开发的`rust版本的postcss`

可以通过在配置文件中添加 [`css.transformer: 'lightningcss'`](https://cn.vitejs.dev/config/shared-options.html#css-transformer) 并安装可选的 [`lightningcss`](https://www.npmjs.com/package/lightningcss) 依赖项来选择使用它

开启后，配置通过`css.lightningcss`传递，通过`css.lightningcss.cssModules`开启css modules

默认情况下，Vite 使用 esbuild 来压缩 CSS。通过 [`build.cssMinify: 'lightningcss'`](https://cn.vitejs.dev/config/build-options.html#build-cssminify) , 将css压缩工作交给lightning css

Lightning CSS不支持CSS 预处理器