当我们从一个文件中读取数据时，文件的二进制(字节)数据会源源不断的被读取到我们程序中，而这个一连串的字节，就是我们程序中的流

流是连续字节的一种表现形式和抽象概念， 流应该是可读的，也是可写的

所有的流都是EventEmitter的实例，所以所有的流都可以进行事件监听和触发事件



流的应用场景：

​	直接读写文件的方式，虽然简单，但是无法控制一些细节的操作

​	比如从什么位置开始读、读到什么位置、一次性读取多少个字节

​	读到某个位置后，暂停读取，某个时刻恢复继续读取等等

​	或者这个文件非常大，比如一个视频文件，一次性全部读取并不合适

​	http模块的Request和Response对象是基于流来实现的



## 流的分类

| 分类                                                         | 说明                                                         | 示例                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [writable](https://nodejs.org/dist/latest-v15.x/docs/api/stream.html#stream_class_stream_writable) | 写入流                                                       | [fs.createWriteStream](https://nodejs.org/dist/latest-v15.x/docs/api/fs.html#fs_fs_createwritestream_path_options) |
| [readable](https://nodejs.org/dist/latest-v15.x/docs/api/stream.html#stream_class_stream_readable) | 读取流                                                       | [fs.createReadStream](https://nodejs.org/dist/latest-v15.x/docs/api/fs.html#fs_fs_createreadstream_path_options) |
| [duplex](https://nodejs.org/dist/latest-v15.x/docs/api/stream.html#stream_class_stream_duplex) | 双向流                                                       | [net.Socket](https://nodejs.org/dist/latest-v15.x/docs/api/net.html#net_class_net_socket) |
| [transform](https://nodejs.org/dist/latest-v15.x/docs/api/stream.html#stream_class_stream_transform) | 转换流<br />例如在写入文件的时候，进行相应的压缩<br />读取的时候，进行相应的解压缩 | [zlib.createDeflate](https://nodejs.org/dist/latest-v15.x/docs/api/zlib.html#zlib_zlib_createdeflate_options) |



### Readable

```js
const fs = require('fs')

// 创建一个读取流
// 参数一 --- 需要读取的文件
// 参数二 --- 配置对象 对应的start和end是都包含的
//       --- 例如本例中 需要读取的字节为 [3, 18]
const stream = fs.createReadStream('./foo.txt', {
  start: 3,
  end: 18
})

// onData事件 === 当读取到的数据发生改变的时候，就会触发对应的事件
// 读取到字节后，会通过data回调的方式传递过来
// 默认情况下，读取到的也是buffer流，可以通过toString转换成对应的字符串
stream.on('data', data => console.log(data.toString()))
```

```js
const fs = require('fs')

const stream = fs.createReadStream('./foo.txt', {
  start: 3,
  end: 18,
  // 默认情况下，对应的[start, end]中的字节是一次性是读取64kb的内容
  // 设置了highWaterMark为3后，每次就会读取3个字节的数据
  // 并触发一次对应的onData事件
  highWaterMark: 3
})

stream.on('data', data => console.log(data.toString()))
```

```js
const fs = require('fs')

const stream = fs.createReadStream('./foo.txt', {
  start: 3,
  end: 18,
  highWaterMark: 3
})

stream.on('data', data => {
  console.log(data.toString())

  // 暂停字节流的读取
  stream.pause()

  // 过1s后，恢复字节流的读取
  setTimeout(() => stream.resume(), 1000)
})
```

```js
const fs = require('fs')

const stream = fs.createReadStream('./foo.txt')

stream.on('open', fd => console.log('文件被打开了', fd))
stream.on('data', data => console.log('数据的读取', data))
stream.on('end', () => console.log('文件读取完毕了'))
stream.on('close', () => console.log('文件读取完，并关闭了'))
```



### writable

```js
const fs = require('fs')

// 创建写入流
// 参数一 - 文件名
// 参数二 - 配置对象 - 可以配置encoding 和 flags 等相关参数
// 注意: 在stream中的属性名是 flags, 而在fs.writeFile中的属性名是 flag
const stream = fs.createWriteStream('./baz.txt', {
  flags: 'a+'
})

// 文件打开事件
stream.on('open', fd => console.log('文件被打开', fd))

// 写入流写入完毕
stream.on('finish', () => console.log('写入流写入完毕了'))

// 文件关闭事件 (写入流写入完毕，并将文件关闭后，才会回调该事件)
stream.on('close', () => console.log('文本被关闭了'))

// 写入文件
// \n 是 文本中的换行
// 参数一 - 需要写入的内容
// 参数二 - 写入后对应的回调函数
stream.write('Hello World\n')
stream.write('Hello Vue\n', err => console.log('写入成功'))

// write事件 并不会自动关闭对应的文件
// 需要手动进行关闭
stream.close()
```

```js
const fs = require('fs')

const stream = fs.createWriteStream('./baz.txt', {
  flags: 'a+'
})

stream.on('close', () => console.log('文本被关闭了'))

stream.write('Hello World\n')

// end方法 等价于 写入内容后 再关闭文件
// end = write + close
stream.end('Hello Vue\n')
```

```js
const fs = require('fs')

// start表示从那个字节开始进行进行写入
// start: 3 --- 表示从第四个字节开始编写
// 注意: 在mac中，使用start flags 既可以是a+也可以是r+
// 但是在windows中，只能使用r+结合start一起使用
// 如果flags对应的值是a+ 那么start会静默失效
const stream = fs.createWriteStream('./baz.txt', {
  flags: 'a+',
  start: 3
})

stream.on('close', () => console.log('文本被关闭了'))

stream.end('Hello Vue')
```



#### 文件复制

`一次读取并写入`

```js
const fs = require('fs')

fs.readFile('./foo.txt', (err, data) => {
  // writeFile的第三个参数 回调 并不能被省略
  fs.writeFile('./baz.txt', data, err => {})
})
```



`使用流来进行读取和写入`

```js
const fs = require('fs')

const readStream = fs.createReadStream('./foo.txt')
const writeStream = fs.createWriteStream('./baz.txt')

readStream.on('data', data =>  {
  writeStream.write(data)
})

readStream.on('close', () => writeStream.close())
```



`通过管道来在读取流和写入流之间建立连接`

```js
const fs = require('fs')

const readStream = fs.createReadStream('./foo.txt')
const writeStream = fs.createWriteStream('./baz.txt')

// 在读取流和写入流之间建立通道
// 执行的时候，会依次读取读取流中对应的数据并写入到写入流中
readStream.pipe(writeStream)
```
