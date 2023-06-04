计算机中所有的内容: 文字、数字、图片、音频、视频最终都会使用二进制来表示

JavaScript可以直接去处理非常直观的数据: 比如字符串，我们通常展示给用户的也是这些内容

而在服务器端，我们可能会需要去操作对应的二进制数据

比如读取图片，音频或视频数据，比如在Node中通过TCP建立长连接 (TCP传输的是字节流，我们需要将数据转成字节再进行传入，并且需要知道传输字节的大小(客户端需要根据字节的大小来判断读取多少内容))

所以Node为了可以方便开发者完成更多功能，提供给了我们一个类Buffer，并且它是全局的

`注意buffer本质上是一个全局类，不是模块，不需要引入后再进行使用`



在计算机中，很少的情况我们会直接操作一位二进制，因为一位二进制存储的数据是非常有限的

所以通常会将8位合在一起作为一个单元，这个单元称之为一个字节(byte)

所以Buffer本质上是一个字节的伪数组对象，伪数组对象中的每一项对应一个字节的大小



### 创建buffer

```js
// 默认情况下 vscode中对应js文件的默认宿主环境为浏览器
// 所以为了可以在vscode中编写node的时候获取更为友好的提示信息
// 可以在js文件的最上边 使用一些require关键字
// const fs = require('fs')

// 早期写法 -- 现在已不被推荐
// Buffer(需要编码的内容，编码方式) --- 默认编码方式是utf-8 
// const buffer = new Buffer('Hello')

// 通过字符串创建对应的buffer --- 推荐
// Buffer.from(需要编码的内容，编码方式) --- 默认编码方式是utf-8 
const buffer = Buffer.from('Hello')

// 一个英文字符串对应一个字节
// 为了显示方式 将其转换为对应的十六进制后再进行输出
console.log(buffer) // <Buffer 48 65 6c 6c 6f>

// 对于中文 在UTF8中 一个中文对应三个字节
console.log(Buffer.from('你好')) // <Buffer e4 bd a0 e5 a5 bd>
```

```js
const buffer = Buffer.from('Hello')

// toString 可以将 buffer转换为对应的字符串
// buffer.toString(解码方式) -- 默认的解码方式是utf-8
console.log(buffer.toString()) // => Hello

// 对于非字符串类型 或 一个字节 调用toString
// 对应的参数应该为 数值 表示的是n进制 默认值为10
console.log(buffer[0].toString(2)) // => 1001000
```



### Buffer.alloc

```js
// 向操作系统 申请 分配 8个字节的内存空间
const buffer = Buffer.alloc(8)

// 默认情况下，对应的内存空间中的值 全都是 0x00
console.log(buffer) // => <Buffer 00 00 00 00 00 00 00 00>

// 因为buffer本质上就是字节伪数组对象
// 所以我们可以对buffer中的每一项进行个性化操作

// charCodeAt和charAt参数index的默认值为0，即不传值的情况下表示的是第一个字符
// str.charCodeAt(index) - 获取字符串中对应索引位置所对应的unicode编码
// str.charAt(index) - 获取字符串中对应索引位置所对应的字符
buffer[0] = 'h'.charCodeAt() 
buffer[1] = 100 // 传入十进制 结果会被转换为对应的十六进制后，再进行存储
buffer[2] =  0x66 // 传入十六进制 直接存储

console.log(buffer) // => <Buffer 68 64 66 00 00 00 00 00>
console.log(buffer.toString()) // => hdf
```



事实上我们创建Buffer时，并不会频繁的向操作系统申请内存，它会默认先申请一个8 * 1024个字节大小的内存，也就是8kb的字节池

1. 如果需要申请的字节数 大于 4kb  ---> 直接向操作系统请求 分配一块内存空间
2. 如果请求的字节数小于 4kb
   + 如果字节数够存放对应的内容  ---> 在之前存储的基础上，将内容存入字节池中
   + 如果字节数不够存放对应的内容 ---> 再请求一块8kb的地址空间加入字节池，并将对应的内容加入到字节池中
