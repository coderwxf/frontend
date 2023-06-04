## wxss

小程序的样式写法有三种: 行内样式、页面样式、全局样式

优先级依次是: 行内样式 > 页面样式 > 全局样式

如果是写在`app.wxss`中的样式 即为全局样式

ps: 小程序不支持通配符选择器



在小程序中，对应的组件会被渲染成对应的shadow dom后再插入到dom树中

所以对于小程序而言，默认情况下所编写的样式都是局部样式



注意:  在wxss中，只能使用网络图片和base64格式的图片，不能使用本地图片

例如background-image中不能使用本地图片作为背景图



```css
/* 
	小程序也可以和css一样引入外部样式
	但是对应路径需要使用相对路径，不能使用绝对路径
*/
@import './index.css'

/*
	在小程序中，每个页面渲染后的最终结构中都会使用page对整个页面进行包裹
	在小程序中，所有的内置组件都可以看成是普通标签，可以通过标签选择器去设置对应的样式
*/ 
page {
  /* 对于变量的设置 一般在这个样式列表的最上边进行设置(约定俗称) */
  --search-padding: 10px 0;
  
  height: 100vh;
  padding: 0 30rpx;
  background-color: #fff;
  /* 
  	无论是替换元素还是page页面，小程序都是通过设置css样式的方式去设置对应的默认样式
  	所以在小程序中，如果设置了page的左右内边距，会导致page页面的宽度变宽
  	如果需要像浏览器那样出现内容居中，左右各15px空白的效果
  	需要修改page对应的box-sizing为border-box (box-sizing的默认值为默认值为content-box)
  */
  box-sizing: border-box;
}
```



### rpx

rpx(responsive pixel): 可以根据屏幕宽度进行自适应，规定所有的屏幕宽为750rpx

如在 iPhone6 上，屏幕宽度为375px，共有750个物理像素，则750rpx = 375px = 750物理像素，1rpx = 0.5px = 1物理像素

所以，推荐 开发微信小程序时设计师使用 iPhone6 作为视觉稿的标准

![image.png](https://s2.loli.net/2022/08/11/LoIA1b3cGyuPRHw.png) 



## wxml

开发中, 界面上展示的数据并不是写死的, 而是会根据服务器返回的数据，或者用户的操作来进行改变

如果使用原生JS或者jQuery的话, 我们需要通过操作DOM来进行界面的更新

小程序和Vue一样, 提供了插值语法: Mustache语法(双大括号)

1. 类似于HTML代码: 比如可以写成单标签，也可以写成双标签
2. 必须有严格的闭合: 没有闭合会导致编译错误
3. 大小写敏感: class和Class是不同的属性

```html
<!-- 基本使用 -->
<view>{{ msg }}</view>

<!-- 使用表达式 -->
<view>{{ firstName + '-' + lastName  }}</view>

<!-- 但是小程序的mustache语法并没有vue灵活 -->
<!-- 在小程序中，mustache中的js表达式会交给WXS进行解析 -->
<!-- 所以在小程序中，不可以在mustache语法中使用ES6+的语法 -->
<!-- 也不可以在小程序的mustache语法中使用非WXS中定义的方法 -->
<!-- <view>{{ `${firstName}-${lastName}` }}</view> -->
<!-- <view>{{ getFullName() }}</view> -->

<!-- 属性中使用mustache语法的时候,属性的引号不可以省略 -->
<view data-msg="{{ msg }}">使用自定义属性传递数据</view>
```



### 逻辑判断

某些时候, 我们需要根据条件来决定一些内容是否渲染:

+ 当条件为true时,  组件会渲染出来
+ 当条件为false时,  组件不会渲染出来

```html
<!-- 只要在界面中使用了逻辑层中的数据，就必须要使用mustache语法 -->
<view wx:if="{{ score >= 90 }}">优秀</view>
<view wx:elif="{{ score >=60 }}">良好</view>
<view wx:else>不及格</view>
```

 `wx:if`指令类似于`v-if`，它是惰性的，如果其对应的值为false的时候，其压根就不会被渲染出来



### hidden

所以和`v-if`对应`v-show`一样，小程序提供了hidden属性

hidden是所有的组件都默认拥有的属性

当hidden属性为true时, 组件会被隐藏

当hidden属性为false时, 组件会显示出来

hidden属性和 v-show 一样 是通过设置属性的display属性是否为none，来控制元素的显示和隐藏的

```html
<!--
  当一个属性只有属性名 没有属性值的时候
  那么该属性名对应的属性值默认为 true
-->
<view hidden>hidden属性基本使用</view>
```



### 循环判断

在实际开发中，服务器经常返回各种列表数据，我们不可能一一从列表中取出数据进行展示

我们需要通过for循环的方式，遍历所有的数据，一次性进行展示

在组件中，我们可以使用wx:for来遍历一个数组 (或字符串 或数字 或对象)

+ 默认情况下，遍历后在wxml中可以使用一个变量index，保存的是当前遍历数据的下标值
+ 数组中对应某项的数据，使用变量名item获取

```html
<view wx:for="{{ users }}" wx:key="*this">
  <text>key: {{ index }}</text>
  <text> - </text>
  <text>value: {{ item }}</text>
</view>
```

```html
<!-- 和vue不一样的是，在小程序中迭代数字，对应的值是从0开始的 -->
<view wx:for="{{ 10 }}" wx:key="*this">
  {{ index }} - {{ item }}
</view>
```

```html
<view wx:for="Klaus" wx:key="*this">
  {{ index }} - {{ item }}
</view>
```

```html
<view wx:for="{{ userInfo }}" wx:key="item" >
  <!-- 
    小程序也是可以迭代对象的
    在迭代过程中，index即为对应的属性名，value即为对应的属性值
  -->
  {{ index }} - {{ item }}
</view>
```



### block

`<block/>` 并不是一个组件，它仅仅是一个包装元素，不会在页面中做任何渲染，只接受控制属性

block的功能类似于vue的template标签

1. 将需要进行遍历或者判断的内容进行包裹
2. 将遍历和判断的属性放在block组件中，不影响普通属性的阅读，提高代码的可读性

```html
<block wx:for="Klaus" wx:key="*this">
  {{ index }} - {{ item }}
</block>
```



### item/index的名称

默认情况下，item – index的名字是固定的

但是某些情况下，我们可能想使用其他名称，比如在需要多层循环的情况下

```html
<!-- wx:for-item="user" -- 指定迭代后对应变量元素为user，不再是默认的item  -->
<!-- wx:for-index -- 指定迭代后对应的索引元素为i， 不再是默认的index -->
<view wx:for="Klaus" wx:key="*this" wx:for-item="user" wx:for-index="i">
  {{ i }} - {{ user }}
</view>
```



### key

小程序内部在组件更新的时候也使用了虚拟DOM

所以当某一层有很多相同的节点时，也就是列表节点时，如果我们希望插入、删除一个新的节点，可以更好的复用节点，推荐添加一个key来提供性能

| 合法值             | 说明                                                         |
| ------------------ | ------------------------------------------------------------ |
| 字符串             | 代表在 for 循环的 array 中 item 的某个 property<br />该 property 的值需要是列表中唯一的字符串或数字，且不能动态改变 |
| 保留关键字 `*this` | `*this` 代表在 for 循环中的 item 本身，这种表示需要 item 本身是一个唯一的字符串或者数字 |



## wxs

WXS(WeiXin Script) 是小程序的一套脚本语言，结合 WXML，可以构建出页面的结构

小程序采用的是双线程模型，渲染界面使用的是一个webview，执行对应的js逻辑使用的是另一个webview

在设计的时候，小程序的mustache语法中是不可以调用自定义方法的

因为这设计到两个webview之间的频繁通信和数据交互，这必然会大大减低小程序的执行性能

为此小程序推出了WXS(WeiXin Script)，WXS的使用方式和`ES5`的语法是基本完全一致的

和小程序执行JS不同的是，小程序的WXS是在渲染UI的那个webview中进行执行的

这样因为存在于同一个webview中，所以也就不存在跨线程之间的数据通信

其执行效率也会比在小程序中执行JS的效率要高一点点

但是因为webview是单线程的，如果在webview中执行复杂的JS逻辑必然会阻塞页面的渲染

所以不推荐在小程序的WXS中实现那么复杂的业务逻辑



### WXS的特点

1. WXS 不依赖于运行时的基础库版本，可以在所有版本的小程序中运行

   WXS是直接在渲染UI的webview中执行的，是直接使用jsCore进行执行

   不需要通过JSBridge在两个webview之间进行通信，必然也就不依赖于小程序的基础库版本

2. WXS 的运行环境和其他 JavaScript 代码是隔离的，WXS 中不能调用其他 JavaScript 文件中定义的函数，也不能调用小程序 提供的API
3. 由于运行环境的差异，在 iOS 设备上小程序内的 WXS 会比 JavaScript 代码快 2 ~ 20 倍。在 android 设备 上二者运行效率 无差异



每一个 .wxs 文件和 `<wxs>` 标签都是一个单独的模块

1. 每个模块都有自己独立的作用域。即在一个模块里面定义的变量与函数，默认为私有的，对其他模块不可见
2. 一个模块要想对外暴露其内部的私有变量与函数，只能通过 module.exports 实现



### 基本使用

`单独的wxs标签中`

```html
<!-- 
  因为一个wxml可以存在多个wxs
  为了避免导出方法命名冲突，需要使用module属性给对应的wxs起一个模块名
-->
<wxs module="utils">
// 在WXS中只能使用ES5的语法，不支持使用ES6及以上的语法
function getFullName(firstname, lastname) {
  return firstname + '-' + lastname
}

// 因为每一个wxs都是一个单独的模块
// 所以对应的方法需要被导出，因为WXS不支持ES6+语法
// 所以WXS使用CJS的方式导出对应的模块内容
module.exports = {
  getFullName: getFullName // 因为不支持ES6+语法，所以不可以使用对象增强
}
</wxs>

<!-- 在调用对应方法的时候，必须添加上对应的模块名 -->
<view>{{ utils.getFullName(firstName, lastName) }}</view>
```



`抽离到一个单独的WXS文件中`

`/utils/util.wxs`

```js
function getFullName(firstname, lastname) {
  return firstname + '-' + lastname
}

module.exports = {
  getFullName: getFullName 
}
```



`index.wxml`

```html
<!-- 可以通过src属性来引入对应的wxs文件 -->
<wxs module="utils" src="/utils/util.wxs"></wxs>

<view>{{ utils.getFullName(firstName, lastName) }}</view>
```



### 示例

`传入一个数，转换为以万或亿为单位的数，并保留三位小数`

例如: 输入36456， 输出 3.6万

```js
function formatCount(count) {
  count = parseFloat(count) || 0

  if (count > 100000000 ) {
   return (count / 100000000).toFixed(1) + '亿'
  } else if (count > 10000 ) {
    return (count / 10000).toFixed(1) + '万'
  } else {
    return count
  }
}

module.exports = {
  formatCount: formatCount 
}
```



`传入一个时间，格式化对应的时间`

例如: 输入 100s， 输出 01 : 40

```js
// 因为padStart和padEnd是ES6+语法
// 所以padStart和padEnd语法在WXS中无法使用
// 需要自己实现
function padStart(num, length, padChar) {
  if ((num + '').length >= length) {
    return num
  }

  var len = length - (num + '').length
  var padStr = ''

  for (var i = 0; i < len; i++) {
    padStr += padChar
  }

  return padStr + num
}

function formatTime(time) {
  time = parseInt(time)

  // 因为在WXS中，所以只能通过var来定义对应的变量
  var minute = parseInt(time / 60)
  var second = time % 60

  if (minute < 10) {
    minute = padStart(minute, 2, '0')
  }

  if (second < 10) {
    second = padStart(second, 2, '0')
  }

  return minute + ':' + second
}

module.exports = {
  formatTime: formatTime 
}
```

