## Number

在JavaScript中，`number`类型不区分整数和浮点数，它们都是数字类型。

数字类型可以进行加减乘除等运算



### 特殊数值

1. **无穷大（Infinity）**

   在JavaScript中，可以表示无穷大和负无穷大：

   ```javascript
   let positiveInfinity = Infinity;
   let negativeInfinity = -Infinity;
   ```

   例如：

   ```javascript
   let result = 1 / 0; // Infinity
   ```

   

2. **非数字（NaN）**

   `NaN`表示“不是一个数字”，通常在进行非法运算时会得到这个结果：

   ```javascript
   let notANumber = 3 * 'abc'; // NaN
   ```

   可以使用`isNaN`函数来判断一个值是否为`NaN`：

   ```javascript
   let isNotANumber = isNaN(notANumber); // true
   ```
   
   `NaN`无法进行任何运算，且和`NaN`的任何比较都会返回`false` 「 包括NaN本身 」



###  进制表示

JavaScript支持多种进制的表示方法：

1. **十进制**

   默认情况下，直接写数字表示十进制：

   ```javascript
   let decimal = 100;
   ```

2. **十六进制**

   以`0x`开头表示十六进制：

   ```javascript
   let hex = 0x64; // 100的十六进制表示
   ```

3. **八进制**

   以`0o`开头表示八进制：

   ```javascript
   let octal = 0o144; // 100的八进制表示
   ```

4. **二进制**

   以`0b`开头表示二进制：

   ```javascript
   let binary = 0b1100100; // 100的二进制表示
   ```

> 1. 进制前缀中的字母不区分大小写，`0x`、`0b` 和 `0o` 前缀中的字母可以是大写，也可以是小写
> 2. 在控制台输出其它进制时，会自动转换为十进制
> 3. 如果要输出其它进制只能通过`toString(n)`将其转换为其它格式字符串 「 其中n是进制值 」



#### 数字的最大值和最小值

在JavaScript中，可以使用`Number.MAX_VALUE`和`Number.MIN_VALUE`来获取数字类型的最大的正数和最小的正数：

```javascript
let maxValue = Number.MAX_VALUE;
let minValue = Number.MIN_VALUE;
```

