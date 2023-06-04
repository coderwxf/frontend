## 列表元素

### 有序列表

ol (ordered list) --- 有序列表，直接子元素只能是li

 li (list item) --- 列表中的每一项

### 无序列表

ul (unordered list) --- 无序列表，直接子元素只能是li

 li (list item) --- 列表中的每一项

### 自定义列表

dl (definition list) --- 定义列表，直接子元素只能是dt、dd

dt (definition term) --- term是项的意思, 列表中每一项的项目名

dd (definition description) 

​	--- 列表中每一项的具体描述，是对 dt 的描述、解释、补充

​	--- 一个dt后面一般紧跟着1个或者多个dd

```html
<dl>
  <dt>阶段一:</dt>
  <dd>HTML</dd>
  <dd>CSS</dd>
  <dd>JavaScript</dd>

  <dt>阶段二:</dt>
  <dd>Vue</dd>
  <dd>小程序</dd>
  <dd>React</dd>
</dl>
```

[![qb5nKg.png](https://s1.ax1x.com/2022/04/04/qb5nKg.png)](https://imgtu.com/i/qb5nKg) 



### 清除列表样式

```css
/* 
	对于自定义列表，需要清除的样式是margin和padding
	对于有序列表和无序列表，需要清除的样式除了margin和padding, 还有前面默认的小图标
*/
ul, li {
  margin: 0;
  padding: 0;
  /* list-style是 list-style-type | list-style-postition | list-style-image的简写 */
  /* 不过因为我们一般只会清除默认的小点，所以直接使用list-style: none即可 */
  /* list-style: none; 的本质是list-style-type: none; */
 	/* list-style和list-style-* 都是可继承属性 */
  list-style: none;
}
```



## 表格元素

表格元素table是一个块级元素

table元素中只有行和单元格的概念，并没有列的概念，所以在HTML中的表格可以不是方方正正的

| 元素            | 说明                  |
| --------------- | --------------------- |
| table           | 表格元素              |
| tr (table row)  | 表格中的行            |
| td (table data) | 行中的单元格          |
| th              | 表格的表头单元格      |
| thead           | 表格的表头            |
| tbody           | 表格的主体            |
| tfoot           | 表格的页脚 --- 不常用 |
| caption         | 表格的标题 --- 不常用 |



![LhcPzz.png](https://s6.jpg.cm/2022/04/04/LhcPzz.png) 

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>

  <style>
    table {
      /* text-align 作为文本相关属性，会被继承 */
      text-align: center;
      /*
        border-collapse 用于设置表格单元格的边框是否合并
        border-collapse 只能设置在table标签上
        默认值为 separate 分开展示
        可选值为 collapse 合并展示
      */
      border-collapse: collapse;
    }

    td,
    th {
      border: 1px solid #333;
      padding: 5px 10px;
    }
  </style>
</head>
<body>
  <table>
    <caption>热门股票</caption>

    <thead>
      <tr>
        <th>排名</th>
        <th>股票名称</th>
        <th>股票代码</th>
        <th>股票价格</th>
        <th>股票涨跌</th>
      </tr>
    </thead>

    <tbody>
      <tr>
        <td>2</td>
        <td>腾讯控股</td>
        <td>600519</td>
        <td>1800</td>
        <td>5%</td>
      </tr>

      <tr>
        <td>3</td>
        <td>贵州茅台</td>
        <td>00700</td>
        <td>400</td>
        <td>3%</td>
      </tr>

      <tr>
        <td>4</td>
        <td>五粮液</td>
        <td>000858</td>
        <td>160</td>
        <td>8%</td>
      </tr>

      <tr>
        <td>5</td>
        <td>东方财富</td>
        <td>300059</td>
        <td>25</td>
        <td>4%</td>
      </tr>
    </tbody>

    <tfoot>
      <td>其它</td>
      <td>其它</td>
      <td>其它</td>
      <td>其它</td>
      <td>其它</td>
    </tfoot>
  </table>
</body>
</html>
```



### 单元格合并

在某些特殊的情况下, 每个单元格占据的大小可能并不是固定的

一个单元格可能会跨多行或者多列来使用

这个时候我们就要使用**单元格合并**来完成

| 分类     | 属性          | 描述                                                    |
| -------- | ------------- | ------------------------------------------------------- |
| 跨列合并 | 使用`colspan` | 在最左边的单元格写上colspan属性, 并且省略掉合并的td     |
| 跨行合并 | 使用`rowspan` | 在最上面的单元格协商rowspan属性, 并且省略掉后面tr中的td |



#### 跨列合并 

![Lhcnw4.png](https://s6.jpg.cm/2022/04/04/Lhcnw4.png) 

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    table {
      border-collapse: collapse;
      text-align: center;
    }

    td {
      border: 1px solid red;
      padding: 5px 10px;
      color: red;
      width: 50px;
      height: 50px;
    }
  </style>
</head>
<body>
  <table>
    <tr>
      <td colspan="2">colspan</td>
    </tr>
    <tr>
      <td></td>
      <td></td>
    </tr>
  </table>
</body>
</html>
```



#### 跨行合并

![Lhc5xX.png](https://s6.jpg.cm/2022/04/04/Lhc5xX.png) 

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    table {
      border-collapse: collapse;
      text-align: center;
    }

    td {
      border: 1px solid red;
      padding: 5px 10px;
      color: red;
      width: 50px;
      height: 50px;
    }
  </style>
</head>
<body>
  <table>
    <tr>
      <td rowspan="2">rowspan</td>
      <td></td>
    </tr>
    <tr>
      <td></td>
    </tr>
  </table>
</body>
</html>
```



## 表单元素

HTML表单元素是和用户交互的重要方式之一，我们可以通过表单来收集用户输入的相关信息

| 元素           | 说明                                               |
| -------------- | -------------------------------------------------- |
| form           | 表单, 一般情况下，其他表单相关元素都是它的后代元素 |
| input          | 单行文本输入框、单选框、复选框、按钮等元素         |
| textarea       | 多行文本框                                         |
| select，option | 下拉选择框                                         |
| button         | 按钮                                               |
| label          | 表单元素的标题                                     |



### [input](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input)

表单元素使用最多的是**input**元素

`input`元素是`行内替换元素`

| 属性值      | 说明                                                         |
| ----------- | ------------------------------------------------------------ |
| type        | 用以决定input的类型，默认值为text<br />input的type属性有很多的可选值，具体可以查看[MDN文档](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/Input) |
| readonly    | 只读， 可以选中，但是无法输入                                |
| disabled    | 禁用                                                         |
| placeholder | 默认的提示文本                                               |
| checked     | 默认被选中<br />只有当type为radio或checkbox时可用            |
| autofocus   | 当页面加载时，自动聚焦，存在多个autofocus的时候，只会有第一个起作用 |
| name        | 字段名称<br />在提交数据给服务器时，可用于区分数据类型       |
| value       | 值                                                           |



### 布尔属性

在HTML中存在一些布尔属性，例如常见的布尔属性有`disabled`、`checked`、	`readonly`、`multiple`、`autofocus`、`selected`等

布尔属性可以没有属性值，写上属性名就代表使用这个属性

1. 不设置，就意味着，值为false，不使用
2. 如果有值，可以是任意值(值甚至可以是false，一般设置的值就是属性名本身)，那么就表示其值为true，也就是使用该值

![LhcKVX.png](https://s6.jpg.cm/2022/04/04/LhcKVX.png) 



### 表单按钮

1. 普通按钮

   ```html
   <input type="button" value="普通按钮">
   
   <!-- 等价于 -->
   <button type="button" >普通按钮</button>
   <!-- button的type属性的默认值为button -->
   
   <!-- 所以上述按钮等价于 -->
   <button>普通按钮</button>
   ```

   

2. 重置按钮

   ```html
   <!--
   	对于重置按钮，默认情况下，只能重置位于同一个表单的其它表单元素
   -->
   <input type="reset" value="重置按钮">
   <!-- 等价于 -->
   <button type="reset" >重置按钮</button>
   ```

   

3. 提交按钮

   ```html
   <!--
   	对于提交按钮，会收集同一个表单中的表单元素提交到对应的地址，并对表单元素进行重置操作
   	这个对应的地址指代的是form的action属性的值
   -->
   <input type="submit" value="提交">
   <!--
   	等价于
   	在表单中, button元素的type的默认值为submit
   	所以在下边代码中 type属性可以省略
   -->
   <button type="submit" >提交</button>
   ```



### label

`label`元素一般跟表单元素一起结合使用，用于在表单元素的标题被点击的时候，自动聚焦对应的表单元素

```html
<!-- 通过label的for属性 和 input元素的id属性 进行相互关联 -->
<label for="name">
  姓名 <input type="text" id="name">
</label>

<!--
只要表单元素的title包含在label元素内即可
对应的表单元素是可以不被包含在label元素内的
-->
<label for="age"> 年龄: </label>
<input id="age" type="number">
```



### radio

```html
<!--
	只有radio的name属性的值一致，那么这些radio才会在同一个组内
	在同一个组内的多个radio的值 是会互斥的
	也就是说只有name值相同的radio才具备单选功能

	name属性的值 --- 传递给服务器的key -- 字段名
	value属性的值 --- 传递给服务器的value --- 对应字段的值
-->
<input type="radio" name="gender" value="male" checked>男
<input type="radio" name="gender" value="female">女
```



### checkbox

```html
<!-- 和radio一样，我们可以使用name将checkbox进行分组 -->
<!-- 可以通过checked属性 来设置单选按钮或多选按钮是否默认选中 -->
<input type="checkbox" name="hobby" value="football" checked>足球
<input type="checkbox" name="hobby" value="basketball">篮球
```



### select

`select`元素用于表示下拉列表，option是select的子元素，一个option代表一个选项

```html
<!-- 默认的下拉列表是单选的 -->
<select name="hobby">
  <!-- 
		可以通过selected属性来设置下拉列表中的那些项是否默认被选中 
		对于单选的下拉列表 默认选择的就是第一个option的value值
  -->
  <option value="basektball" seleced>篮球</option>
  <option value="football">足球</option>
  <option value="badminton">羽毛球</option>
</select>
```

```html
<!--
	multiple表示当前下拉列表可以多选
	如果一个下拉列表是多选的，那么默认情况下所有的值都会被显示出来
	用户可以结合shift键来进行多选

	size属性可以用来设置 展示的option的个数
-->
<select name="hobby" multiple size="2">
  <option value="basektball">篮球</option>
  <option value="football">足球</option>
  <option value="badminton">羽毛球</option>
</select>
```



### textarea

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <style>
    textarea {
      /*
        resize 用于设置textarea能否被拖拽改变尺寸

        禁止缩放: resize: none
        水平缩放: resize: horizontal
        垂直缩放: resize: vertical
        水平垂直缩放: resize: both --- 默认值
      */
      resize: none;
    }
  </style>
</head>
<body>
  <!--
    cols --- 一行可以输入多少个字符
    rows --- 可以输入多少列
  -->
  <textarea  cols="30" rows="10"></textarea>
</body>
</html>
```



### form

form通常作为表单元素的父元素:

form可以将整个表单作为一个整体来进行操作 (例如重置操作和提交操作)

| 属性   | 说明                                                   |
| ------ | ------------------------------------------------------ |
| action | 用于提交表单数据的请求URL                              |
| method | 请求方法(只支持get和post请求方式)，默认是get           |
| target | 使用什么方式打开URL<br />`规则和a元素的target属性一致` |

```html
<form action="https://www.google.com/search" target="_blank">
  <input type="text" name="q">
  <button>google</button>
</form>
```
