## 应用配置信息写到环境变量

我们可以将对应的配置文件放置到项目的根目录下建立一个名为`.env`的文件

.env文件是一个用来存储环境变量的文件.env文件可以让您在不同的环境中（如开发环境和生产环境）使用不同的环境变量

再通过[dotenv](https://www.npmjs.com/package/dotenv)可以将对应的配置加载到`process.env`中

`.env`

```shell
SERVER_PORT=8000
```



`/src/config/server.js`

```js
const env = require('dotenv')

// config默认加载项目根目录下的.env文件
// 可以通过path配置项来加载对应不同的文件，以在不同的环境下加载不同的环境变量
// 如.env.dev 或 .env.prod等
env.config({
  path: '.env.dev'
})

// 配置中转
module.exports = {
  SERVER_PORT
} = process.env
```



## 目录结构划分

```shell
.
├── node_modules
├── package.json
├── pnpm-lock.yaml
└── src
    ├── app # 单独进行app的操作 如注册路由和中间件等
    │   └── index.js
    ├── config #  自动加载配置脚本
    │   └── server.js
    ├── main.js # 项目入口文件
    └── router # 路由
        ├── index.js # 路由导出和路由注册文件
        └── user.js
├── .env.dev # 项目的环境配置变量
```



## app的抽取

`app/index.js --- app需要挂载多个中间件，所以对于这部分逻辑进行单独抽取`

```js
const Koa = require('koa')
const bodyParser = require('koa-bodyparser')
const registerRouter = require('../router')

const app = new Koa()

// 添加body解析
app.use(bodyParser())

// 添加路由
registerRouter(app)

// 导出app实例
module.exports = app
```



`main.js`

```js
// 在main.js中只需要启动对应的服务即可
const app = require('./app')
const { SERVER_PORT } = require('./config/server')

app.listen(SERVER_PORT, () => {
  console.log('server is running~')
})
```



## MVC

如果我们所有对于请求的操作全部在控制器中，会导致代码变得越来越臃肿，不利于维护和扩展

```js
router.get('/list', ctx => {
  // 1. 获取到客户端传入的参数
  // 2. 对对应的参数进行处理
  // 3. 进行数据库的CRUD
  // 4. 根据不同的查询结果向客户端返回不同的JSON数据

 	// 上述这部分逻辑在后续的日常开发中会变得越来越复杂和臃肿，所以需要进行拆分
})

```



1. 路由和对应的业务处理函数(Controller)需要进行拆分并一一对应
2. 数据库操作可能比较复杂，也需要单独进行抽取



`View `

```js
const KoaRouter = require('@koa/router')
const userController = require('/controller/user')

const router = new KoaRouter({
  prefix: '/user'
})

// 路径和控制器的映射
router.get('/list', userController.create)

module.exports = router
```



`Model -- 一般放置在/services目录下`

```js
class UserService {
  create() {
    // 进行数据库的CRUD，并返回对应的结果
  }
}

module.exports = new UserService()
```



`Controller -- 一般放置在/controller目录下`

```js
const userService = require('/services/user')

class UserController {
  create() {
    // 1. 获取到客户端传入的参数
    // 2. 对对应的参数进行处理

    userService.create()

    // 4. 根据不同的查询结果向客户端返回不同的JSON数据
  }
}

module.exports = new UserController()
```



## 数据库连接

```js
const mysql = require('mysql2')

// 建立连接池
const connectionPool = mysql.createPool({
  host: 'localhost',
  port: 3306,
  database: 'coderhub',
  user: 'root',
  password: 'xxxxx',
  connectionLimit: 5
})

// 测试连接是否成功
connectionPool.getConnection((err, connection) => {
  if (err) {
    console.error('连接池建立失败')
    return
  }

  connection.connect(err => {
    if (err) {
      console.error('和数据库建立连接失败')
    }
  })
})

// 调用promise方法，获取promise形式的连接并返回
module.exports = connectionPool.promise()
```



`一个简单的示例`

创建一个新建用户的接口

`view`

```js
const KoaRouter = require('@koa/router')
const userController = require('../../controller/user')

const router = new KoaRouter({
  prefix: '/user'
})

// 路由和控制器之间的映射
router.get('/create', userController.create)

module.exports = router
```



`model`

```js
const connection = require('./database')

class UserService {
  // 这里不做边界处理
  async create({name, password}) {
    // 定义预处理语句
    const statement = `insert into user(name, password) value(?, ?)`
		// 执行预处理语句
    return await connection.execute(statement, [name, password])
  }
}

module.exports = new UserService()
```



`controller`

```js
const userService = require('../services/user')

class UserController {
  async create(ctx) {
    const user = ctx.request.body

    const res = await userService.create(user)

    // 如果不是查询，那么返回结果数组中，没有data数据，只有fields数据
    console.log(res)

    ctx.body = {
      msg: 'create user success'
    }
  }
}

module.exports = new UserController()
```



## 错误处理

`config/error-code.js`

```js
const USER_OR_PASSWORD_ERROR = 'user_or_password_error'
const USER_IS_EXIST = 'user_is_exist'

module.exports = {
  USER_OR_PASSWORD_ERROR,
  USER_IS_EXIST
}
```



`error/handle-error.js`

```js
const app = require('../app')
const handleUserError = require('./user')

app.on('error', (...args) => {
  if (args.errMsg.startWith('USER_')) {
    handleUserError(...args)
  }
})
```



`error/user.js`

```js
const { USER_OR_PASSWORD_ERROR, USER_IS_EXIST } = require('../config/error-code')

module.exports = (errMsg, ctx) => {
  let msg = ''
  let code = ''

  // 规定1xxx形式的错误码为用户相关错误
  switch(errMsg) {
    case USER_OR_PASSWORD_ERROR:
      msg = '用户名或密码错误'
      code = -1001
      break
    case USER_IS_EXIST:
      msg = '用户名已存在'
      code = -1002
      break
  }

  ctx.body = {
    msg,
    code
  }
}
```



## 执行示例

发送请求去注册一个新用户



`匹配路由`

```js
const KoaRouter = require('@koa/router')
const userController = require('../controller/user')
const verifyUserHandler = require('../middleware/user')

const router = new KoaRouter({
  prefix: '/user'
})

// 先进行错误处理
// 如果不存在对应的错误，才会触发第二个中间件
router.get('/create', verifyUserHandler, userController.create)

module.exports = router
```



`verifyUserHandler --- 错误处理`

```js
const { USER_OR_PASSWORD_ERROR, USER_IS_EXIST } = require('../config/error-code')
const userServices = require('../services/user')

module.exports = async (ctx, next) => {
  const { name, password } = ctx.request.body

  if (!name || !password) {
    return ctx.app.emit('error', USER_OR_PASSWORD_ERROR, ctx)
  }

  const users = await userServices.queryUserByName(name)

  // 查询到的表数据会被转换为对象数组的形式返回
  // 即使查询到的结果只有一条，其返回的数据类型也是数组
  if (users.length) {
    return ctx.app.emit('error', USER_IS_EXIST, ctx)
  }


  // 执行下一个中间件
  // 因为下一个中间件也是异步的，所以需要await
  await next()
}
```



`userController.create --- 对应逻辑处理`

```js
const userService = require('../services/user')

class UserController {
  async create(ctx) {
    const user = ctx.request.body

    await userService.create(user)

    ctx.body = {
      msg: 'create user success'
    }
  }
}

module.exports = new UserController()
```



`userService --- 网络请求部分`

```js
const connection = require('./database')

class UserService {
  async create({name, password}) {
    const statement = `insert into user(name, password) value(?, ?)`

    return await connection.execute(statement, [name, password])
  }

  async queryUserByName(name) {
    // 因为name对应的值在对应的sql语句中应该是一个字符串，所以在sql语句中需要使用单引号进行包裹
    // 否则会被识别为字段
    const statement =  `select * from user where name = '${name}'`

    const [data] = await connection.query(statement)

    return data
  }
}

module.exports = new UserService()
```



## 数据加密

`MD5`

散列函数(HASH函数)是一种将任意长度的输入数据映射为固定长度的输出数据的过程。4

这个输出数据就叫做散列值，或哈希值。

散列函数的作用:

1. 用于数据的完整性校验

   在数据传输过程中，可以对数据进行散列，然后在传输完成后，对接收到的数据再次进行散列，比较散列值是否相同，以确保数据在传输过程中没有被篡改

2. 用于密码存储

   将用户的密码进行散列，然后将散列值存储到数据库中，以提高安全性。在用户登录时，可以将用户输入的密码进行散列，然后与数据库中存储的散列值进行比对，以确定用户输入的密码是否正确

其中最为著名的散列函数是`MD5`, MD5的输出是一个128位的哈希值，由于其较长的长度，可以提供更高的安全性

由于MD5算法的设计有一些缺陷，因此在某些情况下，可能会存在暴力破解的风险

因此，在安全要求较高的应用中，可能需要使用更为安全的散列函数，如SHA-1或SHA-2



`utils/md5-password.js`

```js
const crypto = require('crypto')

// plainText - 明文 cipherText - 密文
function encrptByMd5(plainText = '') {
  // 创建md5加密方法
  const md5 = crypto.createHash('md5')

  // 使用update方法对明文进行加密 - 该方法返回的是一个hash对象
  const cipherText = md5.update(plainText)

  // 将hash对象 转换为对应的16进制表示的字符串
  return cipherText.digest('hex')
}

module.exports = {
  encrptByMd5
}
```



`使用`

```js
async function encrptPwd(ctx, next) {
  // 使用加密后的密文 覆盖 原来请求体中的明文
  ctx.request.body.password = encrptByMd5(ctx.request.body.password)
  await next()
}
```

```js
// 中间件的好处就是 可以将一个路由的多个操作进行拆分
router.get('/create', verifyUser, encrptPwd, userController.create)
```