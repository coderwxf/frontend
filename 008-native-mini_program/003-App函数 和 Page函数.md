## 注册小程序

在小程序中，最为重要的是APP实例，在该实例中，可以显示各个Page，而在每一个Page中，又可以存在多个组件

每个小程序都需要在 app.js 中调用 [App 函数](https://developers.weixin.qq.com/miniprogram/dev/reference/api/App.html) 注册小程序实例

我们可以在App函数中进行如下操作:

1. 判断小程序的进入场景
2. 监听生命周期函数，在生命周期中执行对应的业务逻辑，比如在某个生命周期函数中进行登录操作或者请求网络数据
3. 因为App()实例只有一个，并且是全局共享的(单例对象)，所以我们可以将一些共享数据放在这里



### 判断进入场景

小程序的[打开场景](https://developers.weixin.qq.com/miniprogram/dev/reference/scene-list.html)较多, 常见的打开场景:群聊会话中打开、小程序列表中打开、微信扫一扫打开、另一个小程序打开等

```js
// app.js 是整个小程序的入口文件
// 调用App方法可以生成 app实例
// app实例是单例 所以全局有且仅有一个
App({
  // onLaunch方法会在小程序初始化完成后被执行
  // 参数options是对应的事件对象
  onLaunch(options) {
    // 在options中有一个属性，记录着小程序的进入场景
    console.log(options.scene)
  },

  // 关闭小程序后，对应的小程序并不会立即销毁
  // 而是会在后台存活一段时间，以方便用户在短期内可以快速打开小程序
  // 首次打开小程序或者销毁后重新打开小程序 被称为 冷启动
  // 从后台切换到前台 显示的小程序 被称之为 热启动
  // 一般情况下， 当一个小程序在后台被挂起操作30分钟的时候，就会被销毁
  // 所以在第一次加载小程序的时候，会执行onLaunch回调和onShow回调
  // 而第二次执行小程序的时候，就只会执行onShow回调，onLaunch回调并不会被执行
  onShow(options) {
    // 和onLaunch回调一样，onShow回调也会将对应的事件对象传递过来
    // 在该事件对象中，scene属性 记录着 小程序的进入场景
    console.log(options.scene)
  },

  // 当小程序进入后台隐藏的时候，会执行这个回调函数
  onHide() {
    console.log('onHide')
  }
})
```



### 定义全局数据

因为App实例是单例，所以在App上定义的数据本质上其实是全局数据

`app.js`

```js
App({
  // 在App配置对象中 除了官方提供的生命周期函数外
  // 还可以定义任何我们所需要的数据和方法

  // 所以 约定俗称 我们进行将需要共享的数据放置到globalData中
  // 在App实例中共享的数据不是响应式
  // 如果在某次修改了App中共享的全局数据，在其它引用App实例共享的全局数据的地方无法知晓，并根据最新数据刷新界面
  // 所以一般在App中共享的数据 都是那些简单的不会频繁发生改动的数据
  globalData: {
    userInfo: {
      name: 'Klaus',
      age: 23
    }
  }
})
```



`index.js`

```js
// 当我们加载一个页面的时候需要调用Page方法初始化一个Page实例
Page({
  data: {
    userInfo: {}
  },

  // 页面被加载的时候，会调用onLoad方法
  onLoad() {
    // getApp是一个全局方法，通过调用该方法可以获取到App实例
    const app = getApp()

    this.setData({
      userInfo: app.globalData.userInfo
    })
  }
})
```



### 生命周期函数

```js
App({
  globalData: {
    userInfo: ''
  },

  onLaunch() {
    // 以同步的方式 向小程序的storage中设置数据
    // 小程序的storage和浏览器的storage不同
    // 小程序的storage可以存储任意类型的数据，包括对象和数组
    wx.setStorageSync('userInfo', {
      name: 'Klaus',
      age: 23
    })
  },

  onShow() {
    // 以同步的方式去小程序的storage中读取数据
    const userInfo = wx.getStorageSync('userInfo');

    // 因为globaData中的数据并不是响应式的
    // 所以在此处并不需要使用setData方法去设置数据
    this.globalData.userInfo = userInfo
  }
})
```



## 注册页面

程序中的每个页面, 都有一个对应的js文件, 其中调用[Page函数](https://developers.weixin.qq.com/miniprogram/dev/reference/api/Page.html)注册页面实例

在注册时, 常见的操作如下:

1. 初始化一些数据，以方便被wxml引用展示
2. 在生命周期函数中发送网络请求，从服务器获取数据
3. 监听wxml中的事件，绑定对应的事件函数
4. 其他一些监听(比如页面滚动[onPageScroll]、下拉刷新[onPullDownRefresh]、上拉加载更多[onReachBottom]等)



### 发送网络请求

```html
<!--
  swiper 和 swiper-item是微信内置的轮播图组件
  image是 微信内置的图片展示组件

  在组件上直接设置属性 等价于设置属性值为true
  即 autoplay === autoplay="{{ true }}"

  swiper属性
  autoplay - 自动轮播
  circular - 无限轮播
  indicator-dots - 是否显示指示器
-->
<swiper autoplay circular>
  <!--
    wx:for 后边循环的值 如果是变量的时候 需要使用mustache语法进行包裹
    wx:for="{{ banners }}" - 迭代变量banners对应的值
    wx:for="banners" - 将字符串 banners 转换为字符数组后 再进行迭代
  -->
  <block wx:for="{{ banners }}" wx:key="acm">
    <swiper-item>
      <!--
        image和swiper是小程序的内置组件
        类似于html中对应的可替换元素，image和swiper是拥有默认宽高的，可以通过css来进行修改

        image的属性
          mode - 控制图片展示形式 - 类似于object-fit或background-size
            - widthFix 表示的是 根据宽度 自适应 图片的高度
      -->
      <image mode="widthFix" src="{{ item.image }}"></image>
    </swiper-item>
  </block>
</swiper>
```

```css
/*
  image 是选择小程序中所有图片的 标签选择器
*/
image {
  width: 100%;
}
```

```js
Page({
  data: {
    banners: []
  },

  // 页面加载完成后触发的回调
  onLoad() {
    // 一般在这个回调中，发生网络请求来获取对应的数据
    // 可以通过wx.request发送网络请求获取对应数据
    wx.request({
      // 请求地址 - 只有在后台配置过白名单的域名才可以被正常请求
      // 在测试的时候，可以在详情 - 本地设置中 关闭域名的合法性校验 以做测试用
      url: 'http://www.example.com/home/multidata',
      // 成功时候的回调 === 推荐使用箭头函数 以获取正确的this
      success: res => {
        this.setData({
          banners: res.data.data.banner.list
        })
      },
      // 失败时候的回调
      fail: err => console.error(err)
    })
  }
})
```



## Page生命周期

![image.png](https://s2.loli.net/2022/08/09/kxsNbf1oRpHt7OB.png)
