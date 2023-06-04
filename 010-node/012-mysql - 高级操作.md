## 聚合函数

我们所查询到的结果被称之为集合，如果我们需要对查询到的集合进行二次操作而使用到的函数 就被称为聚合函数

```mysql
# 常见的聚合函数
# AVG - 求平均值
# MAX - 最大值
# MIN - 最小值
# SUM - 求和
# COUNT - 计算值不为null的条数

# ROUND(num, count)
# 如果num为小数，且小数点后的位数大于count的时候，进行四舍五入操作
SELECT ROUND(AVG(price), 2) FROM products;
```



### group by

默认情况下，聚合函数将查询到的结果都划分成了一组

如果我们希望将结果划分成多组，此时就可以使用group by

GROUP BY通常和聚合函数一起使用, 表示我们先对数据进行分组，再对每一组数据，进行聚合函数的计算

```mysql
SELECT brand, AVG(price) FROM products GROUP BY brand;
```



如果我们希望给Group By查询到的结果添加一些约束，也就是对分组查询后的结果进行自定义筛选，那么我们可以使用: HAVING

```mysql
SELECT
	brand, AVG(price)
FROM products
GROUP BY brand
HAVING AVG(price) > 4000;
```



## 多表查询

如果所有数据都存放到一张表中，会导致表存在大量的冗余数据

如存放品牌的时候，需要存放品牌名，简介，市值等

如果有多个商品都是一样的品牌的时候，对应的品牌名，简介，市值等相关信息就需要被多次存储

所以为了减少这些冗余数据，往往会将这些数据进行单独抽离，形成一张独立的表



### 外键

如果一个表中的某一个键，对应的值是另一个表中的主键的时候，这个键就被诚之为外键

```mysql
# 如果是新增表的时候
FOREIGN KEY (brand_id) REFERENCES brands(id)
```

```mysql
# 如果是对已有表结构进行修改
ALTER TABLE `products` ADD `brand_id` INT;
ALTER TABLE `products` ADD FOREIGN KEY (brand_id) REFERENCES brands(id);
```



`外键存在时更新和删除数据`

默认情况下，存在外键的对应字段是不能被删除和更新的，因为他们之间存在两张表之间对应字段的相互引用

如果我们需要修改对应的值，我们需要修改on delete或者on update的时候对应的配置值

| 配置      | 说明                                                         |
| --------- | ------------------------------------------------------------ |
| RESTRICT  | 当更新或删除某个记录时，会检查该记录是否有关联的外键记录，有的话会报错的，不允许更新或<br />默认值<br />mysql规范中的值 |
| NO ACTION | 行为和RESTRICT一致<br />整个sql规范中的值                    |
| CASCADE   | 当更新或删除某个记录时，会检查该记录是否有关联的外键记录<br />更新: 那么会更新对应的记录<br />删除: 那么关联的记录会被一起删除掉 |
| SET NULL  | 当更新或删除某个记录时，会检查该记录是否有关联的外键记录，有的话，将对应的值设置为NULL |

```mysql
-- CREATE TABLE `products` (
--   `id` int NOT NULL AUTO_INCREMENT,
--   `brand` varchar(20) DEFAULT NULL,
--   `title` varchar(100) NOT NULL,
--   `price` double NOT NULL,
--   `score` decimal(2,1) DEFAULT NULL,
--   `voteCnt` int DEFAULT NULL,
--   `url` varchar(100) DEFAULT NULL,
--   `pid` int DEFAULT NULL,
--   `brand_id` int DEFAULT NULL,
--   PRIMARY KEY (`id`),
--   KEY `brand_id` (`brand_id`),
--   CONSTRAINT `products_ibfk_1` FOREIGN KEY (`brand_id`) REFERENCES `brands` (`id`)
-- ) ENGINE=InnoDB AUTO_INCREMENT=109 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci

# 查看 生成 对应表的sql语句
SHOW CREATE TABLE products;

# 查看 表的设计结果
DESC products;

# 查看表的生成sql语句 查找到对应的外键名 并删除外键
ALTER TABLE products DROP FOREIGN KEY products_ibfk_1;

# 重新定义外键 并更新对应的外键行为
ALTER TABLE products
	ADD FOREIGN KEY (brand_id)
	REFERENCES brands(id)
	ON UPDATE CASCADE
	ON DELETE CASCADE;
```



### 多表查询

默认情况下，查询到的结果是笛卡尔积，也被称之为直积，表示为 X*Y

也就是说第一张表中每一个条数据，都会和第二张表中的每一条数据结合一次

这样查询出来的结果存在很多的没有意义的数据，所以在实际多表查询的时候

都需要通过where子句加上对应的筛选条件，以过滤掉那些没有意义的数据

```mysql
SELECT * FROM `products`, `brandS` WHERE products.brand_id = brands.id;
```



事实上我们想要的效果并不是这样的，而且表中的某些特定的数据，这个时候我们可以使用 SQL JOIN 操作

![image.png](https://s2.loli.net/2022/12/04/WDQ7hlk5SP9IC61.png)



`左连接`

```mysql
-- 以左表为主进行查询 左表数据全部呈现 右表数据如果不存在就全部显示为null
-- LEFT JOIN 就是左连接 左连接是外连接 全写是 LEFT OUTER JOIN
-- ON 后边跟着的是连接条件
SELECT * FROM products LEFT JOIN brands ON products.brand_id = brands.id;

-- 如果只要左边的和右边有交集的数据 -- 只需要添加过滤条件即可
SELECT * FROM products LEFT JOIN brands ON products.brand_id = brands.id WHERE brands.id is NOT NULL;
```



`右连接`

```mysql
-- 以右边表为主
-- 右连接也是外连接  完整写法 RIGHT OUTER JOIN
SELECT * FROM products RIGHT JOIN brands ON products.brand_id = brands.id;

SELECT * FROM products RIGHT JOIN brands ON products.brand_id = brands.id WHERE products.id is NOT NULL;
```



`内连接`

```mysql
-- 默认情况下， 左连接，右连接 和 全连接 都是外连接
-- 内连接 查出的是 左表存在 且 右表存在的数据
-- 内连接 又被 称之为 交叉连接
-- 内连接，代表的是在两张表连接时就会约束数据之间的关系，来决定之后查询的结果
-- 内连接 完整写法 INNER JOIN 或者 CROSS JOIN 简写为 JOIN
SELECT * FROM products INNER JOIN brands ON products.brand_id = brands.id;

-- 默认筛选方式进行表连接 查询到的数据结果和 内连接是一致的
-- where条件，代表的是先计算出笛卡尔乘积，在笛卡尔乘积的数据基础之上进行where条件的帅选
SELECT * FROM `products`, `brandS` WHERE products.brand_id = brands.id;
```



`全连接`

SQL规范中全连接是使用FULL JOIN，但是MySQL中并没有对它的支持，我们需要使用 UNION 来实现

```mysql
# Mysql中的全连接并不支持
# 所以如果需要实现类似的效果 需要将两个查询到的结果进行合并
SELECT * FROM products LEFT JOIN brands ON products.brand_id = brands.id
UNION
SELECT * FROM products RIGHT JOIN brands ON products.brand_id = brands.id;
```



### 多对多查询

多对多（many-to-many）关系指的是两个表中的数据都可以和另外一个表的数据存在多对多的关系。

例如，在学校的数据库中，学生和课程之间可能存在多对多关系

因为一个学生可以选修多门课程，一门课程也可以被多名学生选修。

在数据库中表示多对多关系的一种方法是使用中间表。

中间表包含两个外键，每个外键分别对应两个表中的主键，并用来表示两个表之间的关系。

```mysql
# 创建学生表
CREATE TABLE IF NOT EXISTS `students`(
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(20) NOT NULL,
    age INT
);

# 创建课程表
CREATE TABLE IF NOT EXISTS `courses`(
id INT PRIMARY KEY AUTO_INCREMENT, name VARCHAR(20) NOT NULL,
price DOUBLE NOT NULL
);

# 中间表
CREATE TABLE IF NOT EXISTS `stu_course` (
	id INT PRIMARY KEY AUTO_INCREMENT,
	stu_id INT NOT NULL,
	course_id INT NOT null,
	FOREIGN KEY(stu_id) REFERENCES students(id)ON UPDATE CASCADE ON DELETE CASCADE,
	FOREIGN KEY(course_id) REFERENCES courses(id) ON UPDATE CASCADE ON DELETE CASCADE
)
```

```mysql
# 查询所有的学生选择的所有课程
SELECT
	# stu_name 是 stu.name的别名
	stu.name stu_name, stu.age stu_age, courses.name course_name, courses.price course_price 
# stu是student的别名
FROM students stu
JOIN stu_course stucos on stucos.stu_id = stu.id
JOIN courses on courses.id = stucos.course_id;
```

```mysql
# 哪些学生是没有选课的
# 这里需要使用左连接，默认是内连接
SELECT
	stu.name stu_name, stu.age stu_age, courses.name course_name, courses.price course_price 
FROM students stu
LEFT JOIN stu_course stucos on stucos.stu_id = stu.id
LEFT JOIN courses on courses.id = stucos.course_id
WHERE courses.name IS NULL;

# 查询哪些课程没有被学生选择
SELECT
	courses.name course_name, courses.price course_price 
FROM students stu
RIGHT JOIN stu_course stucos on stucos.stu_id = stu.id
RIGHT JOIN courses on courses.id = stucos.course_id
WHERE stu.name IS NULL;
```

