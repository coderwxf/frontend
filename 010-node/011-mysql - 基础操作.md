数据库通俗来讲就是一个存储数据的仓库，数据库本质上就是一个软件、一个程序

通常我们将数据划分成两类: 关系型数据库和非关系型数据库

`关系型数据库 - MySQL、Oracle等`

+ 关系型数据库通常我们会创建很多个二维数据表
+ 数据表之间相互关联起来，形成一对一、一对多、多对多等关系
+ 可以利用SQL语句在多张表中查询我们所需的数据



`非关系型数据库 - MongoDB、Redis等`

+ 非关系型数据库的英文其实是Not only SQL，也简称为NoSQL
+ 相当而言非关系型数据库比较简单一些，存储数据也会更加自由
+ NoSQL是基于Key-Value的对应关系，并且查询的过程中不需要经过SQL解析



MySQL是一个关系型数据库，其实本质上就是一款软件、一个程序

+ 这个程序中管理着多个数据库
+ 每个数据库中可以有多张表
+ 每个表中可以有多条数据



### 安装

去[官网](https://dev.mysql.com/downloads/mysql/)下载并安装mysql

```shell
# 默认情况下mysql并没有安装到环境变量中
export PATH=$PATH:/usr/local/mysql/bin

# 查看版本
mysql --version

# 进行连接 --- 退出 输入exit
# 可以在-p后跟上密码(明文显示)
# 也可以在-p后直接回车(密码显示)
mysql -u <用户名> -p
```

| 数据库             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| infomation_schema  | 信息数据库，其中包括MySQL在维护的其他数据库、表、列、访问权限等信息 |
| performance_schema | 性能数据库，记录着MySQL Server数据库引擎在运行过程中的一些资源消耗相关的信息 |
| mysql              | 用于存储数据库管理者的用户信息、权限信息以及一些日志信息等   |
| sys                | 相当于是一个简易版的performance_schema<br />将性能数据库中的数据汇总成更容易 理解的形式 |



## SQL语句

我们希望操作数据库(特别是在程序中)，就需要有和数据库沟通的语言，这个语言就是SQL:

+ SQL是Structured Query Language，称之为结构化查询语言，简称SQL
+ 使用SQL编写出来的语句，就称之为SQL语句, SQL语句可以用于对数据库进行操作



SQL语句的常用规范:

+ 通常关键字使用大写的，比如CREATE、TABLE、SHOW等
  + 关键字大写，自己定义名称小写
  + 约定俗称的规范，不是强制性的
+ 一条语句结束后，需要以 ; 结尾
+ 如果遇到关键字作为表名或者字段名称，可以使用``包裹



### 分类

| 类别                                             | 说明                                                        |
| ------------------------------------------------ | ----------------------------------------------------------- |
| DDL(Data Definition Language) --- 数据定义语言   | 可以通过DDL语句对`数据库或者表`进行: 创建、删除、修改等操作 |
| DML(Data Manipulation Language) --- 数据操作语言 | 可以通过DML语句`对表中的数据`进行: 添加、删除、修改等操作   |
| DQL(Data Query Language) --- 数据查询语言        | 可以通过DQL从数据库中查询记录                               |
| DCL(Data Control Language) --- 数据控制语言      | 对数据库、表格的权限进行相关访问控制操作                    |



### DDL

`对数据库操作`

```mysql
# mysql中注释方式一
-- mysql中注释方式二
```

```mysql
# 查看当前所有的数据库
# 一般情况下，一个项目对应一个数据库
show databases;

# 创建数据库 - mysql中的名称推荐以下划线进行连接
create database <db_name>;

# 重复创建数据库会报错
# IF NOT EXISTS -- 如果不存在
# IF EXISTS -- 如果存在
# 下述SQL表示不存在数据库时再去创建对应的数据库
create database IF NOT EXISTS <db_name>;

# 使用数据库 - 默认情况下 创建完数据库后需要手动进行使用
use <db_name>;

# 查看当前正在使用的数据库
SELECT DATABASE();

# 删除数据库 --- 删除不存在的数据库会报错
drop database <db_name>;

# 如果数据库存在再去删除，不存在则静默失效
drop database if exists <db_name>;
```



对表操作

```mysql
# 创建一张表
CREATE TABLE IF NOT EXISTS t_users (
  # 字段和描述之间使用空格分割
  # 字段和字段之间使用逗号分割
	name VARCHAR(10),
	age INT
);

# 查看所有的表
SHOW TABLES;

# 查看某一张表的表结构
DESC t_users;

# 删除某一张表
DROP TABLE IF EXISTS t_users;
```



#### 数据结构

MySQL支持的数据类型有:数字类型，日期和时间类型，字符串(字符和字节)类型，空间类型(如POINT, POLYGON等)和 JSON数据类型



##### 数字类型

1. [INTEGER, INT, SMALLINT, TINYINT, MEDIUMINT, BIGINT](https://dev.mysql.com/doc/refman/8.0/en/integer-types.html)

![image.png](https://s2.loli.net/2022/12/01/TkHslJFDdEZP21v.png) 

2. [FLOAT, DOUBLE](https://dev.mysql.com/doc/refman/8.0/en/floating-point-types.html)
   + FLOAT是4个字节，DOUBLE是8个字节

3. [DECIMAL, NUMERIC](https://dev.mysql.com/doc/refman/8.0/en/fixed-point-types.html)
   + DECIMAL是NUMERIC的实现形式



##### 日期类型

| 类型      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| YEAR      | YYYY 格式 表示年份                                           |
| DATE      | DATE类型用于具有日期部分但没有时间部分的值<br />DATE以格式YYYY-MM-DD显示值 |
| DATETIME  | DATETIME类型用于包含日期和时间部分的值<br />DATETIME以格式'YYYY-MM-DD hh:mm:ss'显示值<br />支持的范围是1000-01-01 00:00:00到9999-12-31 23:59:59 |
| TIMESTAMP | TIMESTAMP数据类型被用于同时包含日期和时间部分的值<br />TIMESTAMP以格式'YYYY-MM-DD hh:mm:ss'显示值<br />它的范围是UTC的时间范围:'1970-01-01 00:00:01'到'2038-01-19 03:14:07‘ |

DATETIME或TIMESTAMP 存储的值 都可以精确到毫秒



##### 字符串类型

| 类型              | 说明                                                         |
| ----------------- | ------------------------------------------------------------ |
| CHAR              | CHAR类型在创建表时为固定长度，长度可以是0到255之间的任何值<br />CHAR会将尾随空格去掉 |
| VARCHAR           | VARCHAR类型的值是可变长度的字符串，长度可以指定为0到65535之间的值<br />VARCHAR会将尾随空格去掉 |
| BINARY和VARBINARY | 存储二进制字符串，存储的是字节字符串                         |
| BLOB              | BLOB用于存储大的二进制类型                                   |
| TEXT              | TEXT用于存储大的字符串类型                                   |



#### 表约束

| 关键字         | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| PRIMARY KEY    | 为了区分每一条记录的唯一性而存在的字段，被称之为`主键`<br />1. 不可重复，即必须是唯一的<br />2. 不能为空<br />主键可以由表中的多个字段共同组成`PRIMARY KEY(key_part, ...)`, 这类主键被称之为`联合主键`<br />建议:开发中主键字段应该是和业务无关的，尽量不要使用业务字段来作为主键 |
| UNIQUE         | 字段必须是唯一的<br />可以接收多个null值存在 - null 可以绕过唯一性检测 |
| NOT NULL       | 不能为空                                                     |
| DEFAULT        | 用于设置默认值                                               |
| AUTO_INCREMENT | 对应字段会从0开始  以1作为步长  进行自动增长<br />该字段不用显示设置 |

```MYSQL
# 在插入数据前，一般使用sql先将对应的表结构给创建好
CREATE TABLE IF NOT EXISTS t_users (
	id INT PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20) UNIQUE NOT NULL,
	level INT DEFAULT(0), # DEFAULT(0) === DEFAULT 0
	phone VARCHAR(20) UNIQUE, # UNIQUE在不设置NOT NULL的时候，默认值就是NULL
	create_time TIMESTAMP DEFAULT(CURRENT_TIMESTAMP), # 默认值是当前时间戳
  # 默认值是当前时间戳，当数据发生更新的时候，使用最新的时间戳作为对应的最新值
	update_time TIMESTAMP DEFAULT(CURRENT_TIMESTAMP) ON UPDATE CURRENT_TIMESTAMP
);
```

```mysql
# 修改表名
ALTER TABLE t_users RENAME TO users;

# 添加新列
ALTER TABLE users ADD create_time TIMESTAMP;

# 修改列名称
#   1. 新字段后 更上需要设置的对应数据类型和约束 
#      + 数据类型不可以省略
#      + 约束可以省略 省略表示不发生更新
#   2. 新字段名 可以和 旧字段名 同名 - 此时功能和 MODIFY 一致
ALTER TABLE users CHANGE create_time create_at TIMESTAMP;

# 修改列数据类型
ALTER TABLE users MODIFY create_at DATETIME;

# 删除列数据
ALTER TABLE users DROP create_at;
```



### DML

```mysql
# 插入数据
INSERT users (name, age, level, phone) VALUES ('Klaus', 23, 18, '13166522248');

# 更新数据 - 多个更新字段之间使用逗号分割
# 如果没有约束条件 那么默认所有的数据都会发生更新
UPDATE users SET level = 100, age = 18 WHERE name = 'Klaus';

# 删除数据
# 没有约束条件的时候 会删除所有的数据
DELETE FROM users where name = 'Klaus';
```



### DQL

```mysql
# 查询所有数据，并展示所有字段(filelds)INDEX
SELECT * FROM products;

# 查询某些特定的字段
SELECT brand, title FROM products;

# 起别名 - AS关键字可以省略
SELECT id AS prodId, title productName FROM products;
```



#### 查询条件

在开发中，我们希望根据条件来筛选我们的数据，这个时候我们要使用条件查询

```mysql
# 比较运算符: > < >= <= = !=
SELECT * FROM products WHERE price > 4000;

# 逻辑运算符 && and || or
SELECT * FROM products WHERE brand = '华为' || brand = '小米';

SELECT * FROM products WHERE price BETWEEN 2000 AND 4000;

SELECT * FROM products WHERE brand in ('华为', '小米');

# 模糊查询
# _ -> 一个字符
# % -> 零个或多个字符
SELECT * FROM products WHERE title LIKE '__M%';
```



#### 结果排序

当我们查询到结果的时候，我们希望讲结果按照某种方式进行排序，这个时候使用的是ORDER BY;

+ ASC - 升序 - ascend的简写
+ DESC - 降序 - descend的缩写

```mysql
SELECT *  FROM products
	WHERE price >= 4000
	ORDER BY score DESC;
```



#### 分页查询

当数据库中的数据非常多时，一次性查询到所有的结果进行显示是不太现实的

 在真实开发中，我们都会要求用户传入offset、limit或者page等字段, 目的是让我们可以在数据库中进行分页查询

```mysql
# limit - 查询20条数据
# offset - 偏移值 - 默认值为0 - 即不偏移
# offset 30 表示 偏移30条数据 数据从第31条数据开始获取
SELECT *  FROM products
	WHERE price >= 4000
	ORDER BY score DESC
	LIMIT 20 OFFSET 30;
```

```mysql
# LIMIT 30, 20 === LIMIT 20 OFFSET 30
SELECT *  FROM products
	WHERE price >= 4000
	ORDER BY score DESC
	LIMIT 30, 20;
```

