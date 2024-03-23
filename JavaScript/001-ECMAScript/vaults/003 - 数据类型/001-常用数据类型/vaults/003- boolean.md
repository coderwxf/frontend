boolean用于表示真和假

只有两个值 -- 真为true，假为false



## 其余类型值转布尔值

只有falsy值会转换为false，其余都是true



falsy值有五个:

1. 0
2. ''
3. NaN
4. undefined
5. null



## 转换方式

1. `Boolean(val)` --- 参数val不需要进行任何类型转换，直接进行判断

   ```js
   Boolean(3) // => true
   Boolean([]) // => true --- []不是falsy值
   Boolean(false) // => false
   ```

   

2. `!/!!`

   ```js
   !1 // => fasle --- `!`先转换为布尔类型值，再取反
   !!1 // => treu --- !! 等价于 Boolean方法
   ```

   

3. 条件判断语句

   ```js
   // 条件判断中，只有一个值的时候，会先将这个值转换为布尔类型值后在使用
   if (1) {
     console.log('条件永远成立')
   }
   ```

   
