## 将查询结果转换为对象

假设执行如下sql：

```mysql
SELECT * FROM products LEFT JOIN brands ON products.id = brands.id WHERE price > 5000;
```

可以得到如下结果

![image.png](https://s2.loli.net/2022/12/07/3l6sUKkNB1evjSP.png) 

该数据被加载到代码中的时候，会被自动转换为对应的JS对象

```js
// 类似于如下对象
[
  {
  	id: 2,
    brand: '华为',
    title: '华为P20 Pro',
    price: 4488,
    score: 8.3,
    voteCnt: 103,
    url: 'http://detail.zol.com.cn/cell_phone/index1207038.shtml',
    pid: 1207038,
    brand_id: 99,
    id(1): 2,
    name: '小米',
    websit: 'www.mi.com',
    worldRank: 10
	},
  {
    // ....
  },
  // ,,,
]
```

此时所有的属性都会作为对象的第一层属性进行挂载

并不方便我们对其进行操作和维护

```mysql
 SELECT brand, title, price, score, 
 -- JSON_OBJECT(key, value, key, value, ....) as 对象名
 JSON_OBJECT('brandId', brands.id, 'name', brands.name, 'website', brands.website, 'rank', brands.worldRank) as brand
 FROM products 
 LEFT JOIN brands ON products.id = brands.id 
 WHERE brands.id IS NOT NULL ;
```

![image.png](https://s2.loli.net/2022/12/07/3NsOBlRbGV2mfTv.png) 

上述结果在被代码加载的时候，会被转换为类似于如下的对象

```js
[
  {
    brand: '华为',
    title: '华为P20 Pro',
    price: 4488,
    score: 8.3,
    brand(1): {
    	name: '小米', 
    	rank: 10, 
    	brandId: 2, 
    	website: 'www.mi.com'
  	}
	},
  {
    // ....
  },
  // ,,,
]
```



## 多对多转换为数组

```mysql
SELECT * FROM students stu
JOIN stu_course sc ON stu.id = sc.id
JOIN courses cos ON sc.course_id = cos.id;
```

![image.png](https://s2.loli.net/2022/12/07/f7nloRLCtP3H2js.png) 

上述代码会被转换为

```js
[
  {
    id: 1,
    name: 'why',
    age: 18,
    // ....
    name(1): '英语'，
    price: 100
  },
  {
    id: 1,
    name: 'why',
    age: 18,
    // ....
    name(1): '数学'，
    price: 120
  },
  // ....
]
```



但实际我们更期待可以得到如下形式的对象

```js
[
  {
    id: 1,
    name: 'why',
    age: 18,
    // ....
   	courses: [
      {
         name(1): '英语'，
   			 price: 100
      },
      {
         name(1): '数学'，
    		 price: 888
      }
    ]
  }
  // ....
]
```



此时我们可以将对应的sql语句进行如下修改:

```mysql
SELECT stu.id, stu.name, cos.name AS courseNmae, price,
-- JSON_ARRAYAGG是一个聚合函数，可以将分组后的数据转换为数组
-- JSON_OBJECTAGG是一个聚合函数，可以将分组后的数据转换为对象
-- JSON_ARRAYAGG(JSON_OBJECT(cos.name, cos.price)) 查询到对象的数组
-- AS courses 对查询到的结果取别名
JSON_ARRAYAGG(JSON_OBJECT(cos.name, cos.price)) AS courses
FROM students stu
JOIN stu_course sc ON stu.id = sc.id
JOIN courses cos ON sc.course_id = cos.id
GROUP BY stu.id;
```



## 在node中操作mysql

如果需要在node中操作对应的mysql数据库，那么就需要安装对应的第三方库

这类帮助我们操作数据库的第三方库也被称之为数据库驱动

在node中可以帮助我们链接数据库的有以下两个库

| 库     | 说明                                                         |
| ------ | ------------------------------------------------------------ |
| mysql  | 最早的Node连接MySQL的数据库驱动, 最早的mysql驱动 --- 已经不在维护 |
| mysql2 | 在mysql的基础之上，进行了很多的优化、改进 --- 现在使用较多   |

相比较mysql，mysql2拥有以下几个优势：

1. 更快/更好的性能
2. Prepared Statement(预编译语句)
3. 支持promise

```shell
npm install mysql2
```

```js
const mysql = require('mysql2')

// 创建连接对象
const connection = mysql.createConnection({
  host: 'localhost', // 数据库地址
  port: 3306, // 端口 - mysql 默认3306
  user: 'root', // 用户名
  password: 'xxxxx', // 密码
  database: 'demo_db' // 需要链接那个数据库
})

// 通过连接发起查询
// 这里的query指代的是结构化查询语句中的这个查询，不是增删改查中的查询
// 所以这里的query可以进行包括DCL,DDL,DML,DDL在内的所有操作
connection.query('select * from students', (err, data, fields) => {
  if (!err) {
    // data - 查询到的数据结果
    console.log(data)
    // fields - 查询的数据结果 对应的字段
    console.log(fields)
  }
})
```



## 预编译语句

sql语句的执行和JS一样，需要进行编译解析，优化后再进行执行

所以预编译语句就是那些只需要编译一次就可以重复执行的语句



预编译语句的好处

1. 优化性能

   + 默认情况下，sql给mysql客户端的时候，mysql需要解析编译优化对应的sql语句后再执行
   + 如果这个sql语句是被重复执行的，那么对应的编译优化过程会被重复执行
   + 而预编译语句在交给mysql后，对应的编译优化后的结果会被mysql存储起来
   + 如果重复执行该预编译语句，mysql会自动执行执行对应的编译后的代码，就不需要重复优化和编译对应的sql语句

2. 防止sql注入

   + 默认的sql语句很容易被sql注入

   ```mysql
   # 如果mysql执行以下这个语句
   # 客户端传入 用户名 Klaus 密码 'pwd' or 1 = 1
   # 此时mysql会认为 用户名为Klaus且密码为pwd  或者 1 = 1 的时候 执行该语句
   # 此时对应的语句永久为真，这就是sql注入
   # ps 区别 name = 'Klaus' and password = 'pwd' or 1 = 1 和 name = 'Klaus' and (password = 'pwd' or 1 = 1)
   select * FROM students WHERE name = 'Klaus' and password = 'pwd' or 1 = 1;
   ```

   ```mysql
   # 预编译语句如下
   # 此时客户端传入 用户名 Klaus 密码 'pwd' or 1 = 1
   # mysql在解析的时候，发现只有两个？ 也就是只需要两个参数
   # 此时它会将传入的结果解析为 name = 'Klaus' password = ('pwd or 1= 1')
   # 此时密码错误，可以有效的避免sql注入
   select * FROM students WHERE name = ? and password = ?;
   ```

   

### 基本使用

```js
// query方法用于查询普通的sql语句
// execute方法用于执行预编译sql语句
/*
  execute方法的参数：
  参数一 - 对应的sql语句
  参数二 - 参数值构成的数组
  参数三 - 回调函数，参数和query方法对应回调的参数一致
*/
connection.execute('select * from products where brand = ? and price > ?', ['华为', 3000], (err, data, fields) => {
  if (!err) {
    // data - 查询到的数据结果
    console.log(data)
    // fields - 查询的数据结果 对应的字段
    console.log(fields)
  }
})
```



## 连接池

默认情况下，mysql中一个链接只能处理一个查询请求

这也就意味着如果有多个请求同时存在，那么同一时间mysql只能处理一个请求，对于其余请求而言，只能处于等待状态

```js
// 为每一个请求单独创建一个链接用于处理对应的操作
const mysql = require('mysql2')

const connection = mysql.createConnection({
  host: 'localhost',
  port: 3306,
  user: 'root',
  password: 'xxxxx', 
  database: 'demo_db' 
})


connection.query('select * from students', (err, data, fields) => {
  if (!err) {
    console.log(data)
  }
})

// 在请求结果查询完毕后，需要销毁对应的链接，否则该链接将一个处于占用状态
connection.destory()
```



事实上，mysql2给我们提供了连接池(connection pools)

连接池可以在需要的时候自动创建连接，并且创建的连接不会被销毁，会放到连接池中，后续可以继续使用

```js
const mysql = require('mysql2')

const connection = mysql.createPool({
  host: 'localhost',
  port: 3306,
  user: 'root',
  password: 'xxx',
  database: 'demo_db',
  connectionLimit: 5 // 连接池最大链接数 - 默认值是100
})

// 之后的查询操作和不是有连接池是一致的
connection.execute('select * from products where brand = ? and price > ?', ['华为', 3000], (err, data, fields) => {
  if (!err) {
    console.log(data)
    console.log(fields)
  }
})
```



## 使用promise方式建立链接

```js
// 如果使用promise方式建立连接
// 在成功的回调中，data和fields会组合成一个数组一起作为结果返回
connection.promise().execute('select * from products where brand = ? and price > ?', ['华为', 3000]).then(([data, fields]) => {
  console.log(data)
  console.log(fields)
}).catch(err => console.error(err))
```


