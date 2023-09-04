## 模板语法

### 数据绑定

`index.js`

```js
Page({
  // data的值传入对象即可，并不需要是函数的返回值
  // 在创建实例的时候 小程序会确保每个页面和组件的data都是相互独立的
  data: {
    msg: 'Hello World',
    imgSrc: 'https://picsum.photos/100/100',
    // toFixed(n) 保留 n位 小数点
    // n的默认值是0 
    // toFixed(n)的返回值是string
    randomNum: Math.random().toFixed(2)
  }
})
```

`index.html`

```html
<!-- 
	小程序页面 存在一个根组件 Page
	换句话说 下面的所有组件 在渲染后都将成为page的子组件
-->

<!-- 在小程序中使用mustache(双大括号)语法来动态绑定数据 -->
<view>{{ msg }}</view>

<!-- 在小程序中 绑定属性 使用的依旧是mustache语法， 不需要使用指令 -->
<image src="{{imgSrc}}" mode="widthFix"/>

<!-- 在小程序的mustache语法中 可以使用任何合法的JavaScript表达式 -->
<!-- 例如 三元运算符、 变量 及 算术运算 等 -->
<!-- 但不可以在内容区调用函数 -->
<view>{{ randomNum * 100 }}</view>
```



可以在`调试器 --- AppData`中 查看所有在小程序当前页面中定义的状态

![image-20230819155347173](https://s2.loli.net/2023/08/19/6N1ZcnyKpuD3zVs.png) 



### 事件

事件是渲染层向逻辑层进行通信的一种方式

+ 用户在渲染层产生行为，并触发对应事件
+ 渲染层将事件传递给逻辑层，并执行对应的业务逻辑处理



#### 常用事件

| 事件类型 | 绑定方式                    | 事件描述               |
| -------- | --------------------------- | ---------------------- |
| `tap`    | `bindtap`或`bind:tap`       | 小程序的点击事件       |
| `input`  | `bindinput`或`bind:input`   | 文本框输入事件         |
| `change` | `bindchange`或`bind:change` | checkbox等状态改变事件 |

> `bindxxx` 和 `bind:xxx` 这两种写法是等价的
>
> 可以将`bindxxx` 看成是 `bind:xxx` 的语法糖写法



#### 事件对象

当事件的回调函数被触发时，会接收事件对象`event`作为回调函数的参数被传入

| 属性           | 类型    | 说明                                                         |
| -------------- | ------- | ------------------------------------------------------------ |
| type           | String  | 事件类型                                                     |
| timeStamp      | Integer | 页面打开到事件被触发所经过的毫秒数                           |
| target         | Object  | 触发事件的元素，即事件最初发生的地方                         |
| currentTarget  | Object  | 当前正在处理事件的元素，即事件当前所在的阶段的元素           |
| detail         | Object  | 传入的额外信息                                               |
| touches        | Array   | 触发事件时，停留在屏幕上的触摸点相关信息数组<br />也就是触发事件时，有多少根手指停留在屏幕上 |
| changedTouches | Array   | 触发事件时，变化的触发点信息数组                             |



##### target vs currentTarget

![image-20230819163941228](https://s2.loli.net/2023/08/19/wNCFE1Qkuaofi8v.png) 

在小程序中 也存在事件冒泡和事件捕获

+ 当我们点击按钮后，会触发按钮的点击事件
+ 随后会随着冒泡传递给外层的view组件，并触发`view`组件的点击事件

我们在`view`组件上绑定了`bindtap`事件

+ `target`是最初触发事件的那个元素，在这里就是`button组件`
+ `currentTarget`是当前正在处理事件的元素，在这里就是`view组件`



##### touches vs changedTouches

![image.png](https://s2.loli.net/2023/08/20/5w7esAmqEtLvzou.png) 

当刚开始点击的时候，只有一个手指，所以此时touches和changedTouches中对应的值是一致的

但是当一个手指按住不动，在添加第二个和第三个手指的时候，因为新增加了两个手指

所以touches中记录的是所有的手指个数，所以是长度为3的数组

而changedTouches中记录的是改变的手指个数，所以是长度为2的数组



同理，在点击结束手指离开的时候

在touchstart事件中，因为此时点击在屏幕上的手指为一个，且是从零个手指变成一个手指的点击

所以此时touches和changedTouches中的值是一致的，是长度为1的数组

但是在touchend事件中，因为此时点击屏幕的手指由一个转变为了零个手指，

所以此时touches数组为长度为0的数组，而changedTouches为长度为1的数组 

![image.png](https://s2.loli.net/2023/08/20/o5Jt9PMlS6dxWEm.png) 



#### 参数传递

##### 通过 data-* 传递参数

`html`

```html
<!-- 
  小程序中参数传参需要通过data-*属性
  
  data-step的值之所以使用mustache语法
  是因为传入的是number类型值，而不是string类型值

	小程序中事件名是全小写的
-->
<button 
  type="primary" 
  size="mini" 
  bindtap="handleTap"
  data-step="{{2}}"
>
  {{ count }}
</button>
```

`js`

```js
Page({
  data: {
   count: 0
  },

  // 事件处理函数和data同级
  handleTap(e) {
    // 小程序更新状态需要使用setData方法
    // 参数是一个对象，会和this.data进行Object.assign操作
    this.setData({
      // 通过this.data 获取当前组件状态
      // 通过e.currentTarget.dataset 获取传入的组件
      // 推荐使用currentTarget获取参数而不是target
      // 是因为target不一定是当前组件，但是currentTarget一定是当前组件
      count: this.data.count + e.currentTarget.dataset.step
    })
  }
})
```



##### 通过 mark:* 传递参数

`html`

```html
<!-- 
  小程序也可以通过mark进行方法参数传递
-->
<button 
  type="primary" 
  size="mini" 
  bindtap="handleTap"
  mark:step="{{2}}"
>
  {{ count }}
</button>
```

`js`

```js
Page({
  data: {
   count: 0
  },

  handleTap(e) {
    this.setData({
      // 通过mark传递的参数会被挂载到e.mark上
      count: this.data.count + e.mark.step
    })
  }
})
```



##### mark:* vs data-*

`html`

```html
<view 
  bindtap="handleTap"
  mark:info="view"
  data-info="view"
>
  <button 
  type="primary" 
  size="mini" 
  mark:step="{{2}}"
  data-step="{{2}}"
>
  click me
</button>
</view>
```

`js`

```js
Page({
  handleTap(e) {
    // dataset 只会获取当前组件上对应的 data-* 属性
    console.log(e.currentTarget.dataset); // {info: "view"}
    // mark 会获取 整个冒泡阶段中 所有组件的 mark:* 属性
    console.log(e.mark); // {step: 2, info: "view"}
  }
})
```



#### 双向数据绑定

`html`

```html
<!-- 
	1. 属性值的引号是不可以省略的
	2. 默认情况下，小程序中的input组件是没有任何样式的，所有的样式都需要手动进行设置
  3. input的值 通过value属性进行设置
	4. 在小程序中没有v-model之类的语法糖写法，只有完整写法
-->
<input value="{{inputValue}}" bindinput="handleInput" />
```

`js`

```js
Page({
  data: {
    inputValue: 'inital value'
  },

  handleInput(e) {
    // input输入框的值 通过 e.detail 获取
    this.setData({
      inputValue: e.detail.value
    })
  }
})
```



### 条件渲染

```html
<view wx:if="{{ sex === 'male' }}">男</view>
<view wx:elif="{{ sex === 'female' }}">女</view>
<view wx:else>保密</view>
```

```html
<!-- 
  block类似于vue中的template元素
  可以避免渲染不必要的元素，从而提升页面性能
-->
<block wx:if="{{ flag }}">
  <view>name</view> 
  <view>age</view>
</block>

<!-- 
  hidden属性 类似于vue的 v-show
  hidden属性 通过display属性 控制元素的显示和隐藏
  hidden属性 不可以和wx:elif 和 wx:else一起结合使用
	hidden的属性值为true时 隐藏。 属性值为false的时候 显示 --- 和wx:if的判断正好相反
-->
<view hidden="{{ !flag }}">hidden</view>

<!-- 
  对于属性值是布尔类型的元素属性
  如果只传递属性 并不传递属性值
  那么默认的属性值将会是true
-->
<view hidden>hidden</view>
<view hidden="{{ false }}">hidden</view>
```

> 在`切换不频繁或者切换条件比较复杂`的情况下， 推荐使用`wx:if`
>
> 在`需要频繁进行元素切换的情况下`，推荐使用`hidden`属性



### 列表渲染

```html
<!-- 
  1. wx:for 所在的元素 会被多次创建并渲染
  2. wx:for的值 直接是 需要迭代的元素
  3. wx:for底层使用了类似于虚拟DOM的形式进行迭代和更新 
     所以需要通过wx:key给予循环的每一个项一个key值
     + wx:key的值 是一个字符串 不需要通过mustache语法进行指定
     + 值可以是迭代项的属性名 如 id 
     + 值也可以是*this 表示迭代项本身 此时迭代项需要是字符串或数值类型值
-->
<view wx:for="{{ users }}" wx:key="*this">
  {{ index }} -- {{ item }}
</view>

<view> ---------------------- </view>

<!-- 
	1. wx:for 中 每一个数据项的默认名是item 可以通过wx:for-item进行更改
  2. wx:for 中 每一个索引的默认名是index 可以通过wx:for-index进行更改
-->
<view 
  wx:for="{{ users }}" 
  wx:key="*this" 
  wx:for-index="idx" 
  wx:for-item="itemName"
>
  {{ idx }} -- {{ itemName }}
</view>

<view> ---------------------- </view>

<!-- 
  1. wx:for 可以用于迭代数组、普通对象、数值 
		 并不可以用来迭代其余可迭代对象(如Set，Map等)
  2. 如果迭代的是普通对象 
     + index --- 属性名(key)
     + value --- 属性值(value)
-->
<view wx:for="{{ user }}" wx:key="index">
  {{ index }} -- {{ item }}
</view>

<view> ---------------------- </view>

<!-- 
    1.index --- 索引值 (从0到9)
    2.value --- 数值 (从0到9)
    ps: 小程序中迭代数值是从0开始的
        vue中迭代数值是从1开始的
-->
<view wx:for="{{ 10 }}" wx:key="index">
  {{ index }} -- {{ item }}
</view>

<!-- 
  wx:for 可以迭代字符串  
  index从0开始进行计算 
-->
<view wx:for="KlAUS" wx:key="index">
  <text>{{ index  }} --- {{ item }}</text>
</view>
```



## WXSS

WXSS(Weixin Style Sheets) 是小程序特有的样式语言，用于美化WXML的组件样式，功能类似于网页开发中的CSS

WXSS拥有绝大部分的CSS语法功能，并在此基础上扩展了rpx尺寸单位



### rpx

rpx ( responsive pixel ) 是小程序中用于解决屏幕适配问题的尺寸单位

rpx类似于root pixel 自动设置完成的rem

rpx可以看成将屏幕在宽度上等分为750分，换句话来说无论屏幕大小如何改变，其对应的总宽度一直是750rpx

+ 在较大的设备上，1rpx对应的宽度就相对较小
+ 在较大的设备上，1rpx对应的宽度就相对较大

在实际渲染的时候，微信小程序会将rpx尺寸转换为实际的px尺寸后再进行实际的渲染



在iPhone6上，屏幕的宽度为375px，DPR=2。

换句话说在iPhone6上，屏幕有750个物理像素，正好1个物理像素 等于 1rpx

因此，建议在开发微信小程序时，使用iPhone6作为小程序的视觉设计稿标准

```shell
# 在iPhone6上
1rpx === 0.5px === 1 physical pixel
1px === 2rpx === 2 physical pixel
```

![image-20230821231531936](https://s2.loli.net/2023/08/21/Ga7mNUJHtlwBpnS.png) 

 

### 样式作用域

在小程序中 `app.wxss`是全局作用域，而`<page>.wxss`是局部作用域

当局部样式和全局样式冲突的情况下：

1. 权重大的覆盖权重小的样式
2. 如果权重相同，按照就近原则，局部样式覆盖全局样式

```css
/*  
  1. 在小程序中 可以通过@import语法 导入外联的样式表
  2. 在小程序中 绝对路径 以项目根路径为基准路径
  3. @import语句必须以分号结尾 表示样式导入语句的结束
*/
@import '/foo.wxss';

view {
  color: blue
}
```

