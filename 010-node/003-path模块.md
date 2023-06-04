path模块用于对路径和文件进行处理，提供了很多好用的方法

我们知道在Mac OS、Linux和window上的路径时不一样的

如果我们在window上使用 \ 来作为分隔符开发了一个应用程序，要部署到Linux上面，就可能因为路径分隔符不一致而产生问题

所以为了屏蔽他们之间的差异，在开发中对于路径的操作我们可以使用 path 模块

简而言之，path模块可以帮助我们简化路径操作操作，并屏蔽不同操作系统中路径分隔符的差异



| 方法     | 说明                                          |
| -------- | --------------------------------------------- |
| dirname  | 获取文件的父文件夹                            |
| basename | 获取文件名                                    |
| extname  | 获取文件扩展名                                |
| join     | 路径拼接 可以返回相对路径，也可以返回绝对路径 |
| resolve  | 路径拼接 一定返回的是绝对路径                 |



```js
const path = require('path')

const pathname = path.resolve(__dirname, __filename)

console.log(pathname) // => /Users/klaus/Desktop/demo/foo/index.js
console.log(path.extname(pathname)) // => .js
console.log(path.basename(pathname)) // => index.js
console.log(path.dirname(pathname)) // => /Users/klaus/Desktop/demo/foo
```



`path.join`

如果我们希望将多个路径进行拼接，但是不同的操作系统可能使用的是不同的分隔符, 

这个时候我们可以使用path.join函数

```js
const path = require('path')

const basename1 = '/a/b'
const basename2 = './c/d'
const filename = '../foo.js'

// path.join方法的参数是路径序列
console.log(path.join(basename1, filename)) // => /a/foo.js
console.log(path.join(basename2, filename)) // => c/foo.js
console.log(path.join(basename1, basename2, filename)) // => /a/b/c/foo.js
console.log(path.join(basename2, basename1, filename)) // => c/d/a/foo.js
```



`path.resolve`

+ path.resolve() 方法会把一个路径或路径片段的序列解析为一个绝对路径
+ 给定的路径的序列是从右往左被处理的，后面每个 path 被依次解析，直到构造完成一个绝对路径
+ 如果在处理完所有给定path的段之后，还没有生成绝对路径，则使用当前工作目录
+ 生成的路径被规范化并删除尾部斜杠，零长度path段被忽略

```js
const path = require('path')

const basename1 = '/a/b'
const basename2 = './c/d'
const filename = '../foo.js'

console.log(path.resolve(basename1, filename)) // => /a/foo.js

// 如果拼接后的路径依旧是相对路径 会和当前命令行执行时候所在的__dirname一起拼接成绝对路径
console.log(path.resolve(basename2, filename)) // => /Users/klaus/Desktop/demo/foo/c/foo.js

// 路径拼接从右往左执行，一旦拼接的结果是绝对路径 即停止函数的执行
console.log(path.resolve(basename2, basename1, filename)) // => /a/foo.js

// 路径拼接中的空字符串会被忽略
console.log(path.resolve(basename2, '', filename)) // => /Users/klaus/Desktop/demo/foo/c/foo.js

// 不传递参数的时候 返回的是当前命令行执行时候所在的__dirname
console.log(path.resolve()) // => /Users/klaus/Desktop/demo/foo
```