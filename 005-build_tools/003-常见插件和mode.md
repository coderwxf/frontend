Webpack的另一个核心是Plugin

Loader是用于特定的模块类型进行转换，即用来进行文件转换

Plugin可以用于执行更加广泛的任务，比如打包优化、资源管理、环境变量注入等

也就是说在webpack中，所有loader之外的功能，都可以由plugin来进行实现



## cleanWebpackPlugin

每次修改了一些配置，重新打包时，都需要手动删除dist文件夹

我们可以借助于一个插件来帮助我们完成，这个插件就是CleanWebpackPlugin

```shell
npm install clean-webpack-plugin -D
```

`配置`

```js
const { CleanWebpackPlugin } = require('clean-webpack-plugin')

plugins: [
  new CleanWebpackPlugin()
]
```



在最新的webpack配置中，在output下新增了属性clean

当设置该属性为true的时候，也可以完成`clean-webpack-plugin`对应的功能

也就是说最新的webpack配置中，可以不用再安装`clean-webpack-plugin`这个插件

因为对应的功能，最新的webpack已经集成

```js
output: {
  filename: 'bundle.js',
    path: path.resolve(__dirname, 'build'),
      clean: true
}
```



## HtmlWebpackPlugin

我们的HTML文件是编写在根目录下的，而最终打包的dist文件夹中是没有index.html文件的

我们的HTML文件是编写在根目录下的，而最终打包的dist文件夹中是没有index.html文件的

所以我们也需要对index.html进行打包处理

对HTML进行打包处理我们可以使用另外一个插件:HtmlWebpackPlugin

```shell
npm install html-webpack-plugin -D
```



使用`htmlWebpackPlugin`进行打包的时候，在打包后的文件夹中会存在`index.html`, 并且会自动引入打包后形成的`index.js`

这个index.html文件，默认情况下是根据ejs的一个模板来生成的

我们也可以自己手动指定对应的模板文件，并使用EJS的填充语法(即`<%= 变量 %>`)来为我们的模板填充对应的数据

`template.html`

```ejs
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title><%= htmlWebpackPlugin.options.title %></title>
  </head>
  <body>
    <div id="app"></div>
    <h1><%= htmlWebpackPlugin.options.name %></h1>
    <script src="./build/bundle.js"></script>
  </body>
</html>
```



`webpack.config.js`

```js
const HtmlWebpackPlugin = require('html-webpack-plugin')

plugins: [
  new HtmlWebpackPlugin({
    title: '自定义title',
    name: 'Klaus',
    template: './index.html'
  })
]
```



## DefinePlugin

DefinePlugin允许在编译时创建配置的全局常量，是一个webpack内置的插件(不需要单独安装)

`webpack.config.js`

```js
const { DefinePlugin } = require('webpack')

plugins: [
  // 通过DefinePlugin插件 可以在编译的时候 注入全局变量
  new DefinePlugin({
    // DefinePlugin对应的value值是一个字符串类型的js表达式，会被交给eval函数进行解析
    // 所以如果对应的值是字面量类型值的时候，需要在双引号内单独再次使用单引号进行包裹
    NAME: "'Klaus'",
    AGE: "'23'"
  })
]
```

```js
// 默认情况下，DefinePlugin会在全局注入一个名为'process.env.NODE_ENV'的全局变量
// 如果是在开发环境下 对应的值为 production
// 如果是在生产环境下 对应的值为 development
console.log(process.env.NODE_ENV)
```



## mode

Mode配置选项，可以告知webpack当前处于什么开发环境

webpack会根据对应的模式，进行一系列的内置优化

mode的可选值有 `none | development | production`

默认值为`production`

![image.png](https://s2.loli.net/2022/07/19/VEsCcDQInFiaYwK.png) 

