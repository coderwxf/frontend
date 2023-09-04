## 页面导航

页面导航指的是页面之间的互相跳转和跳转过程中的参数传递



在小程序中进行页面导航主要有两种方式:

+ `声明式导航`:    通过`<navigator>`组件进行页面导航
+ `编程式导航`:   通过调用小程序API 来实现页面之间的导航 --- 命令式导航



### 声明式导航

在`app.json`的list字段中进行配置了的页面被称之为tabBar页面，反之就被称之为非tabBar页面



#### 导航到 tabBar 页面

1. url 表示需要 跳转到的路径 --- 必须以 / 开头
2. open-type  表示跳转方式 --- 需要设置为switchTab

```html
<navigator url="/pages/logs/logs" open-type="switchTab">
	go to tab
</navigator>
```



#### 导航到非 tabBar 页面

1. url 表示需要 跳转到的路径 --- 必须以 / 开头

2. open-type  表示跳转方式 --- 需要设置为`navigate` 

   `navigate`也是open-type的默认值 如果省略就表示自动使用`navigate`

````html
<!-- 此时默认的open-type的值就是navigate -->
<navigator url="/pages/index/index">
	go to index
</navigator>
````



#### 后退导航

后退导航表示返回上一级页面或多级页面



在小程序中，当跳转到非tabBar页面时，会插入一条新的历史记录，因此可以执行回滚操作，返回上一个页面

而当跳转到tabBar页面时，小程序会重置历史记录栈，也就是清空历史记录，这意味着无法进行页面回退操作，在这种情况下，页面的跳转主要依靠tab切换，无法返回之前的非tabBar页面



1. `open-type`的值必须是`navigateBack`
2. `delta`表示后退的层级
   + `delta`的值必须是数字类型值 
     + 如果省略，则缺省值将会是1
     + 如果传入的是其它类型值，会优先尝试将其转换为数值类型值
       1. 转换成功 --- 使用转换后的值
       2. 转换失败 --- 使用缺省值 --- 也就是 默认值 1
   + 如果回滚值大于最大可回滚数，回滚值delta会被重置为最大可回滚数

```html
<navigator open-type="navigateBack" delta="{1}">
  返回
</navigator>
```



### 命令式导航

#### 导航到tabBar页面

1. `wx.switchTab` 用于跳转到tab页面
2. `wx.switchTab`是wx提供的方法，所以可以调用`success`、`fail` 和 `complete`方法
3. url表示 tab页面路径 必须以/开头 
4. 路径后边不能携带任何参数

```ts
wx.switchTab({
  url: '/pages/profile/profile',
})
```



#### 导航到非tabBar页面

1. `wx.navigateTo` 用于跳转到非tab页面
2. `wx.navigateTo是wx提供的方法，所以可以调用`success`、`fail` 和 `complete`方法`
3. url表示 tab页面路径 必须以/开头 
4. 路径后边可以携带任何参数

```ts
wx.navigateTo({
  url: '/pages/home/home',
})
```



#### 后退导航

1. `wx.navigateBack` 用于后退导航
2. `wx.navigateBack`是wx提供的方法，所以可以调用`success`、`fail` 和 `complete`方法`
3. delta 表示回退的页面数 规则和声明式编程中delta的规则一致

```ts
wx.navigateBack({
  delta: 3
})
```



## 参数传递

在非tab页面之间 可以通过query参数的形式进行参数的传递

`参数传递`

```html
<!-- 声明式编程 -->
<navigator url="/pages/home/home?name=klaus&age=23">click me</navigator>
```

```js
// 命令式编程
wx.navigateTo({
  url: '/pages/home/home?name=Klaus&age=23',
})
```

`参数获取`

```js
Page({
 onLoad(options) {
   // 如果传入了query参数
   // 对应的query参数会被转换为对象后 作为onLoad的参数被传入 
   console.log(options); // {name: "Klaus", age: "23"}
 }
})
```

`参数查看`

![image-20230903121550869](https://s2.loli.net/2023/09/03/iBAh2VT8LXpKlow.png) 



