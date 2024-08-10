## Math

`Math`对象是一个内置对象，不是构造函数，它包含常见的数学属性和方法



**Math 属性**：

- `Math.PI`：圆周率。



**Math 方法**：

- `Math.floor(x)`：向下取整。
- `Math.ceil(x)`：向上取整。
- `Math.round(x)`：四舍五入并取整，返回结果依旧是个数值类型。
- `Math.random()`：生成0到1之间的随机数，不包含1。
- Math.pow(x, y)`：计算x的y次幂。`
- Math.abs(x)`：取绝对值。`
- `三角函数：`Math.sin(x)`, `Math.cos(x)`, `Math.tan(x)`等。



例如: 生成5到50之间的随机数（包含50）的算法 「 **大- 小，向下取整后 再加小** 」

```js
function getRandomNumber(min, max) {
  return Math.floor(Math.random() * (max + 1 - min)) + min;
}

let randomNumber = getRandomNumber(5, 50);
```



