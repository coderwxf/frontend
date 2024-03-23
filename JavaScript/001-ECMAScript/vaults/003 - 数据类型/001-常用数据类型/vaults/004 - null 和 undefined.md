null 和 undefined 的共同点

1. 都是基本数据类型
2. 都只有一个值，即null 和 undefined本身
3. 都代表没有值



null：

1. 代表空值，一般用于表示空对象
2. 该值一般是用户主动设置的
3. 在内存中存储的时候，null是一个特殊标志符(类似于布尔值)，所占空间很小



undefined:

1. 变量定义了但是没有赋值
2. undefined一般作为变量的默认值



## void

1. 使用方式 `void 表达式` 或 `void(表达式)`
2. void 恒 返回 undefined

```js
console.log(void console.log(1)) 
/*
	1
	undefined
*/
```

所以也有人 使用 `void 0` 来 替代 `undefined`



