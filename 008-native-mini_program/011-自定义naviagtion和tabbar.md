## navigationBar

使用者

```json
{
  "usingComponents": {
    "naviagtion-bar": "/components/navigation-bar/navigation-bar"
  },
  // 设置navigationStyle为custom 表示不使用默认的navigationBar
  "navigationStyle": "custom"
}
```



`自定义navigationBar组件`

```html
<!-- 
	当自定义navigationBar的时候，布局将从顶部开始
	也就是顶部的状态栏将不占位
	所以需要自己设置margin-top进行占位
-->
<view 
  class="nav" 
  style="margin-top: {{statusBarHeight}}px;"
>
  <view class="left">
    <view class="slot">
      <slot name="left"></slot>
    </view>
    <view class="default">
      <van-icon name="arrow-left" size="22px" />
    </view>
  </view>
  
  <view class="center">
    <view class="slot">
      <slot name="center"></slot>
    </view>  
    <view class="default">
      {{ title }}
    </view>
  </view>
  
    <!-- 
      虽然小程序的nav-bar的右侧并不需要设置任何的内容
      因为会存在宽度大约为90px的胶囊按钮对其进行占位操作
      而且因为是自定义导航栏，为了方便布局
      所以一般情况下，依旧会设置右侧部分进行占位操作
    -->
  <view class="right"></view>
</view>
```

```css
.nav {
  height: 44px;
  display: flex;
  align-items: center;
}

.left {
  width: 120rpx;
  text-align: center;
}

.center {
  flex: 1;
  text-align: center;
}

.right {
  width: 180rpx;
}

.default {
  display: none;
}

.slot:empty + .default {
  display: block;
}
```

```js
Component({
  options: {
    // 开启多插槽模式
    multipleSlots: true
  },

  properties: {
    title: {
      type: String,
      value: '默认标题'
    }
  },

  data: {
    statusBarHeight: 0
  },

  // 当自定义导航栏的时候
  // 1. 原本导航栏右侧的胶囊按钮无法被修改
  // 2. 状态栏 无法被移除 且不占位
  
  // 无论在那种机型下，小程序的导航栏的高度都是44px
  lifetimes: {
    attached() {
      wx.getSystemInfo({
        success: res => {
          this.setData({
            // 通过getSystemInfo动态获取statusBar的高度
            statusBarHeight: res.statusBarHeight
          })
        } 
      })
    }
  }
})
```



## tabbar

