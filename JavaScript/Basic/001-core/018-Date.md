## 时间表示

1. 格林威治标准时间（GMT）
   + GMT是根据太阳在本初子午线上的日常运动来定义的并根据经度划分了对应的时区
2. 协调世界时（UTC）
   + 地球的自转速度并不是恒定的，它会受到多种因素的影响，从而导致微小的变化或“误差”，GMT会出现偏差。
   + 现代标准时间是通过原子钟计算的UTC时间

在计算机系统中，通常使用的是**UTC时间**，而不是传统的**GMT时间**



**创建当前时间的`Date`对象**

```javascript
let now = new Date();
console.log(now);
```

**通过字符串创建`Date`对象**

日期字符串一般使用 `YYYY-MM-DD HH:mm:ss.sss`的格式

其余格式 「 如 `YYYY/MM/DD` 」是否可以正确解析取决于浏览器实现 「 大多数现代浏览器确实可以解析这些格式，但它们不是标准的、推荐的格式 」

```javascript
let specificDate = new Date('2022-08-08');
console.log(specificDate);
```

**通过年月日等参数创建`Date`对象**

```javascript
let detailedDate = new Date(2033, 9, 10, 8, 7, 33);
console.log(detailedDate);
```

**通过时间戳创建`Date`对象**

```javascript
let timestampDate = new Date(1000);
console.log(timestampDate);
```



## 时间戳

时间戳是一个整数，表示从1970年1月1日00:00:00 UTC以来经过的毫秒数。

JavaScript中是十三位时间戳，单位是毫秒 「 毫秒级Unix时间戳 」

Go是十位时间戳，单位是秒 「Unix时间戳」



## 格式化

Date 对象提供了多种方法来获取和格式化日期和时间，例如：

- `getFullYear()`: 获取年份 「 `getYear()`已被废弃 」
- `getMonth()`: 获取月份（从0开始计数，即0表示1月，11表示12月）
- `getDate()`: 获取日期（一个月中的第几天，1-31）
- `getDay()`: 获取日期 （0表示星期日，6表示星期六）
- `getHours()`: 获取小时（0-23）
- `getMinutes()`: 获取分钟（0-59）
- `getSeconds()`: 获取秒（0-59）
- `getMilliseconds()`: 获取毫秒（0-999）
- `getTime()`: 获取时间戳



Date中有对应的get方法，就有对应的set方法 「 并没有`setDay`方法 」

当设置一个不存在的日期时，日期会自动调整为下一个有效日期。

例如，在 5 月份（31 天）的日期对象上调用 `setDate(32)`，日期会自动调整为 6 月 1 日



## 日期格式标准

**RFC 2822**

1. 主要用于电子邮件

2. `Fri, 21 Nov 1997 09:55:06 -0600` 

   `星期，日 月 年 时:分:秒 时区`



**ISO 8601**

1. 国际标准化组织（ISO）制定的日期和时间表示法

2. 用于交换数据的日期和时间格式, 常用于互联网

3. `2023-08-07T14:23:55Z`

   `年-月-日T时:分:秒时区` -- `Z`表示零时区



**输出中国特有日期格式**

```js
const date = new Date()

// Wed Aug 07 2024 20:33:17 GMT+0800 (中国标准时间)
console.log(date.toString())

// Wed Aug 07 2024
console.log(date.toDateString())

// 20:33:17 GMT+0800
console.log(date.toTimeString())
```



**输出IOS 8601格式**

```js
const date = new Date()

// 2024-08-07T12:33:17.768Z
console.log(date.toISOString())

// 2024-08-07T12:33:17.768Z
console.log(date.toJSON())
```



**输出RFC 2822格式**

```js
const date = new Date()

// Wed, 07 Aug 2024 12:40:45 GMT
console.log(date.toUTCString())
```



**输出本地格式字符串**

```js
const date = new Date()

// 2024/8/7 20:33:17
console.log(date.toLocaleString())

// 2024/8/7
console.log(date.toLocaleDateString())

// 20:33:17
console.log(date.toLocaleTimeString())
```



**获取时间戳**

```js
// 获取当前时间戳
Date.now()

// 将日期对象转换为时间戳
const date = new Date()
date.getTime()
date.valueOf()

// 日期字符串转时aN
const dateString = '2033-03-03'
// 解析成功返回时间戳，解析失败返回NaN
Date.parse(dateString)
```

`Date.parse()` 方法支持多种日期字符串格式，但并不保证所有格式都能正确解析。建议使用符合 ISO 8601 标准的日期字符串
