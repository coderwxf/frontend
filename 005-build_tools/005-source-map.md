在实际开发和上线的过程中，会对源代码进行编译丑化和压缩

以便于对应的代码可以更快更安全的运行在各个版本的浏览器上

但是编译压缩丑化后的代码，是以牺牲可读性为前提的

换句话来说编译丑化压缩后的代码会之前的源代码很有可能会完全不一样

为了方便我们调试编译后的代码，就生成了一种名为`source-map`的文件

通过该文件，浏览器可以从编译丑化压缩后的代码还原出构建之前的源代码

从而方便我们对于源代码的编译和调试

> source map文件是一种映射文件，用于将编译后的代码（通常是JavaScript或CSS）映射回源代码

```js
const path = require('path')

module.exports = {
  mode: 'production',
  // 使用devtool选项开启对应的source-map
  devtool: 'source-map',
  entry: '/src/main.js',
  output: {
    path: path.resolve(__dirname, 'build'),
    filename: 'bundle.js'
  }
}
```

这样在构建后的代码的最后一行会存在一条特殊的注释

```js
// 当浏览器识别到该注释后，就会自动将源代码通过构建后的代码和映射文件反向推导出来
// source-map注释的标准格式 //# sourceMappingURL= xxx
//# sourceMappingURL=common.bundle.js.map
```



## source-map结构

```js
// source-map 本质上是一个json对象 --- source-map文件是不支持注释的
{
  // 第三个版本的source-map文件
  // 最初source-map生成的文件大小是原始文件的10倍，第二版减少了约50%，第三版又减少了50%
	"version": 3,
    
  // 构建后的文件名
	"file": "bundle.js",
  
  // 这是一段使用base64 VLQ编码格式进行编码后得到的字符串
  // 用来记录源文件中的相关位置信息
	"mappings": "mBAEAA,QAAQC,KAAQ,GAAI,QCDlBD,QAAQC,IAAIC,K",
	
  // 构建后的文件是通过那些文件一起构建出来的
  "sources": [
		"webpack://js-demo/./src/main.js",
		"webpack://js-demo/./src/math/math.js"
	],
  
  // 原始文件对应的源代码
	"sourcesContent": [
		"import { sum } from './math/math'\n\nconsole.log(sum(10, 30))",
		"export function sum(num1, num2) {\n  console.log(num1 + num2)\n}\n"
	],
  
  // 源代码中使用到的变量列表
  // 和编译后的丑化后的变量名一一对应
	"names": [
		"console",
		"log",
		"num1"
	],
  
  // 用于指定源代码对应的根目录
  // 如果为空字符串 则表示不指定 - 使用绝对路径
	"sourceRoot": ""
}
```



## 可选source-map的值

webpack为我们提供了非常多的[选项](https://webpack.docschina.org/configuration/devtool/)(目前是26个)，来处理source-map

选择不同的值，生成的source-map会稍微有差异，打包的过程也会有性能的差异，可以根据不同的情况进行选择



### 不生成source-map

| 值    | 说明                                                         |
| ----- | ------------------------------------------------------------ |
| false | `不使用source-map`，也就是没有任何和source-map相关的内容     |
| none  | `production模式下的默认值`(什么值都不写) ，不生成source-map<br />`不能主动设置，只能作为production模式下的默认值存在` |
| eval  | development模式下的默认值，不生成source-map<br />但是它会`在eval执行的代码中，添加 //# sourceURL=`<br />它会被浏览器在执行时解析，并且在调试面板中生成对应的一些文件目录，方便我们调试代码<br />也就是`eval也可以推导出源文件，但是对应的源文件信息可能并不是那么的准确` |



### 生产source-map的值

#### source-map

生成一个完整独立的source-map文件，并且在bundle文件中有一个注释，指向source-map文件

开发工具会根据这个注释找到source-map文件)，并且解析

```js
//# sourceMappingURL=bundle.js.map
```

source-map选项还原出的源代码是完整的，并且会同时还原webpack运行时生成的一些源代码



#### eval-source-map

会生成sourcemap，但是source-map是以DataUrl（也就是base64编码后）添加到eval函数的后面

一般情况下，每引入一个模块文件就会生成一个对应的eval函数



#### inline-source-map

会生成sourcemap，但是source-map是以DataUrl添加到bundle文件的最后边



#### cheap-source-map

会生成sourcemap，但是会更加高效一些(cheap 便宜的 低开销)，因为它没有生成列映射(Column Mapping)

> `cheap-source-map`只能在development环境下使用，production环境下静默失效

![image.png](https://s2.loli.net/2022/12/24/mU5gJEZc8A2BSWr.png) 



#### cheap-module-source-map

会生成sourcemap，类似于cheap-source-map，但是对源自loader的sourcemap处理会更好

loader会对对应的文件进行编译解析成对应的Js文件后，再交给webpack进行打包

所以一个文件经过loader处理后，对应的内容和格式也会变的“面目全非”

如果使用`cheap-source-map`，那么代码只能被还原到经过loader处理后的代码

如果使用的是`cheap-module-source-map`，代码会还原到经过loader处理前的代码

> `cheap-module-source-map`只能在development环境下使用，production环境下静默失效

![image.png](https://s2.loli.net/2022/12/24/uxNhmWRFeJ4Q7lA.png) 



#### hidden-source-map

会生成sourcemap，但是不会对source-map文件进行引用

相当于删除了打包文件中对sourcemap的引用注释

```js
// 原本位于bundle.js末尾的这行代码会被移除
//# sourceMappingURL=bundle.js.map
```

所以默认情况下是没有开启对应的source-map

但是如果我们手动添加进来，那么sourcemap就会生效了



#### nosources-source-map

会生成sourcemap，但是生成的sourcemap只有错误信息的提示，不会生成源代码文件

也就是可以看到错误信息所在的文件名，列数和行数

但是点进去后，是无法准确看到对应源文件的

![image.png](https://s2.loli.net/2022/12/24/XFEN7LiCY6GPndU.png) 



## 多个值组合

事实上，webpack提供给我们的26个值，是可以进行多组合的

1. inline-|hidden-|eval: 三个值时三选一
2. nosources: 可选值
3. cheap: 可选值，并且可以跟随module的值 --- 即可选值为cheap或cheap-module

```shell
[inline-|hidden-|eval-][nosources-][cheap-[module-]]source-map
```

在开发环境，推荐使用`source-map`或`cheap-module-source-map`

在生产环境，推荐使用 `false` 或 `none`

